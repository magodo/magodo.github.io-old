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

# Concurrent programming with __Process__

__NOTE__
* Once a child process is forked, for example, in an server-client model, remember to release unused resource in both parent process and child process, for example, connection fd in parent process and listening fd in child process since they don't need it

# Concurrent programming with __I/O Multiplexing__

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






