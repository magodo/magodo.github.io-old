---
layout: "post"
title: "Concurrent Porgramming"
---

> Introducing concurrent programming concept and highlight some key points
<!--excerpt-->

# Mordern OS provide 3 basic approach for building concurrent programs

1. Process
  * each logical control flow is a process
  * processes are scheduled and maintained by kernel
  * use explicit _interprocess communication(IPC)_ mechanism for communication

2. I/O multiplexing
  * application explicitly schedule their own logical flow in context of a process
  * logical flows are modeled as state machines that the main program explicitly transitions from state to state as a result of data arriving on file descriptors
  * since the program is a single process, all flows share the same address space

3. Threads
  * think of threads as a hybrid of the other two approcheas: scheduled by kernel, sharing the same virtual address space

## Concurrent programming with __Process__

__NOTE__

* Once a child process is forked, for example, in an server-client model, remember to release unused resource in both parent process and child process, for example, connection fd in parent process and listening fd in child process since they don't need it

__Pros & Cons__

* Pros:
    - Have separate address space, which eliminates accidentally overwrite the VM of another process
    - Processes are scheduled automatically by the kernel

* Cons:
    - Have separate address space, which is more difficult to share state information. To share information, they must use explicit IPC mechanisms
    - Tends to be slower because the overhead for process control and IPC is high

## Concurrent programming with __I/O Multiplexing__

The basic idea is to use the `select` function to ask the kernel to suspend the process, returing control to the application only after one or more I/O events have occured, as in the following examples:

* Return when any descriptor in the set {0, 4} is ready for reading.
* Return when any descriptor in the set {1, 2, 7} is ready for writing.
* Timeout if 152.13 seconds have elapsed waiting for an I/O event to occur.

`select` prototype:

    int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

In simple case, `select` takes two inputs:

* a descriptor set called the _readfds_
* cardinality _n_ of the readfds

`select` blocks until at least one descriptor in the readfds is ready for reading. A descriptor _k_ is ready for reading if and only if a request to read 1 byte from that descriptor would not block.

As a side effect, `select` modifies the fd\_set pointed to by argment _readfds_ to indicate a subset of the _readfds_ called the _ready set_, consiting of the descriptors in the readfds that are ready for reading.

The value returned indicates the cardinality of the _ready set_

__Pros & Cons__

* Pros: 
    - Give programmer more control over the behavior for their programs. E.g we can imagine writing an event-driven concurrent server that gives preferred service to some clients
    - Since I/O multiplexing runs in the context of a single process, thus every logical flow has access to the entire address space of the process. This makes it easy to share data between flows
    - Is able to be debugged as any sequential program, using a familiar debugging tool as GDB
    - More efficient

* Cons:
    - Coding complexity

## Concurrent programming with __Threads__

# Introducing thread

__General Introduction__

* The threads are scheduled by kernel
* Each thread has its own _thread context_, including a unique integer _thread ID_(TID), stack, stack pointer, program counter, general-purpose registers and condition codes
* All threads share the entire VM space of one process, including its code, data, heap, shared libraries and open files
* _main thread_ is the first thread to run in the process,  _peer threads_  are the following created threads. Together they form a _pool_ of peers, independent of which threads were created by which other threads
* The code and local data for a thread is encapsulated in a _thread routine_, it takes a generic pointer as input and returns a generic pointer

__Thread creation__

New thread is created via calling `pthread_create`:

    #include <pthread.h>

    typedef void *(func)(void *);
    
    int pthread_create(pthread_t *tid, pthread_attr_t *attr,
                       func *f, void *arg);

When `pthread_create` returns, argument _tid_ contains the ID of the newly created thread.

The new thread can determine its own thread ID by calling `pthread_self`:

    #include <pthread.h>

    pthread_t pthread_self(void);

Take note to _arg_ argument in `pthread_create`. When _arg_ points to a local variable in creating thread, it might raise __race__ condition against following thread creation.

For example, imagine there is a continuous creation of threads:

{% highlight C linenos %}
void main(){
    int connfd;

    ...

    while(1){
        connfd = accpet(listenfd, (SA *)&clientaddr, &clientlen);
        pthread_create(&tid, NULL, thread, &connfd);
    }
}

void *thread(void *vargp){
    int connfd = *((int*)vargp);
    ...
}
{% endhighlight %}

In this case, there is a __race__ condition between line 13 and line 7. When line 13 is executed after line 7, then it will cause two thread using identical connection file descriptor.

In order to avoid potentially dadly __race__, we should assign each _arg_ to its own dynamically allocated memory block, which is tended to be used exclusively by the created thread. Therefore, the changed code should look like this:

{% highlight C linenos %}
void main(){
    int *connfdp;            // define a pointer

    ...

    while(1){
        connfdp = (int*)malloc(sizeof(int));  // dynamically allocate thread-specific argument
        *connfdp = accpet(listenfd, (SA *)&clientaddr, &clientlen);
        pthread_create(&tid, NULL, thread, connfd);
    }
}

void *thread(void *vargp){
    int connfd = *((int*)vargp);
    free(vargp)               // thread is responsible to free the allocated memory
    ...
}

{% endhighlight %}

__Thread termiation__

A thread terminates in one of the following ways:

* The thread terminates _implicitly_ when its top-level thread routine returns
* The thread terminates _explicitly_ by calling the `pthread_exit` function. If the _main thread_ calls `pthread_exit`, it waits for all other _peer threads_ to terminate, and then terminates the main thread and the entire process with a return value of `thread_return`

<b></b>

    #include <pthread.h>

    void pthread_exit(void *thread_return);

* _main thread_ or _peer thread_ calls `exit` function, which terminates the process and all threads assciated
* Some other _peer thread_ terminates the current thread by calling the `pthread_cancel` function with the TID of the current thread

<b></b>

    #include <pthread.h>

    int pthread_cancel(pthread_t tid);

__Thread repeatedly termination__

Threads wait for other threads to terminate by calling the `pthread_join` function:

    #include <pthread.h>

    int pthread_join(pthread_t tid, void **thread_return);

The `thread_join` function blocks until thread _tid_ terminates, assigns the generic (void \*) pointer returned by the thread routine to the location pointed to by *thread_return* , and then _reaps_ any memory resources held by the terminated thread.

Note: unlike Unix `wait` function, the `pthread_join` function can only wait for a _specific_ thread to terminiate. There is no way to instruct `pthread_wait` to wait for an _arbitrary_ thread to terminate.

__Thread detachment__

At any point in time, a thread is _joinable_ or _detached_.

* A _joinable_ thread can be reaped and killed by other threads. Its memory resrouces are not freed until it is reaped by another thread;
* A _detached_ thread cannot be reaped or killed by other threads. Its memory resources are freed automatically by the system when it terminates.

By default, threads are created _joinable_. In order to avoid memory leaks, each joinable thread should:
* be explicitly reaped by another thread;
or
* be detached by a call to `pthread_detach` function:

<b></b>

    #include <pthread.h>

    int pthread_detach(pthread_t tid);

__Thread initialization__

The `pthread_once` function allows you to initialize the state associated with a thread routine.

    #include <pthread.h>

    pthread_once_t once_control = PTHREAD_ONCE_INIT;

    int pthread_once(pthread_once_t *once_control,
                     void (*init_routine)(void));

The *once_control* is a global or static variable that is always initialized to `PTHREAD_ONCE_INIT`. 

The first time you call `pthread_once` with an argument of *once_control*, it invokes `init_routine`, which is a function with _no_ input arguments that returning nothing(it is always used to define some global variables).

Subsequent calls to `pthread_once` with the same *once_control* variable do nothing. 

# Share variables in thread programs

The variable is _shared_ if and only if multiple threads reference some instance of the variable.

Each thread has its own separate _thread context_, which includes:

  * thread ID (kernel space)
  * stack (mem)
  * stack pointer (reg)
  * program counter (reg)
  * condition codes (reg)
  * general-purpose register (reg)

Each thread shares the rest of the process context with the other threads, which includes:

  * entire user virtual address space, includes:
      * read-only text (code)
      * r/w data
      * heap
      * shared library code and data
  * same set of open files

In operational sense, it is impossible for one thread to read or write the register values of another thread; On the other hand, any thread can access any location in the shared VM (including other threads' stack). Thus, registers are never shared, whereas VM is always shared(stack is not clean).

__mapping variables to memory__

* _Global variables_: At run time, the r/w area of VM contains exactly one instance of each global variables that can be referenced by any thread.
* _Local automatic variables_: At run time, each thread's stack contain its own instance of any local automatic variables.
* _Local static variables_: Same as _Global variables_.


