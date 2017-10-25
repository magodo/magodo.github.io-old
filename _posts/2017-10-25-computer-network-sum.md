---
layout: "post"
title: "Computer Network Tip"
categories:
- "tool"
---

<!--more-->

***
Table of Content

* TOC
{:toc}
***

# 2. Physical Layer

# 3. Data Link Layer

Data link layer take the packet from network layer, and encapsulates them into **frames** for transmission. Each frame contains a frame header, a payload field for holding the packet, and a frame trailer.

                Packet              ---- Network Layer

                  |
                  v

    | Header | Payload | Trailer |  ---- Data Link Layer


## 3.1 Design Issues

Data link layer has following functions:

1. Provide a well-defined service interface to network layer.
2. Deal with transmittion errors.
3. Regulate the flow of data so that slow receivers are not swamped by fast senders.

### 3.1.1 Service Provided to Network Layer

1. Unacknowledged connectionless service.

    Used for low error rate channels, real-time services(e.g. voice).

2. Acknowledged connectionless service.

    Used for unreliable channel (e.g. wireless systems).

    The request-ack stuff done at data link layer is better than at network layer in this case, it is because links usually have a strict maximum frame length imposed by the hardware, and known propagation delays. The network layer might send a large packet that is broken up into, say, 10 frames, of which 2 are lost on average. It would then take a very long time for the packet to get through. Instead, if individual frames are acknowledged and retransmitted in data link layer, then errors can be corrected more directly and more quickly.

3. Acknowledged connection-oriented service.

    Used for long, unreliable links such as a satellite channel or a long-distance telephone circuit.

### 3.1.2 Framing

The data get from Physical Layer is some kind of bit stream, Data Link Layer needs to break up the bit stream into frames. A good design must take it easy for the receiver to:

1. Find the start of new frames.
2. Use little of the channel bandwidth for framing.

Four methods are introduced:

1. Byte count

    * Description

        Use a field in the header to specify the number of bytes in the frame.

    * Con

        Bit flip in length field cause un-synchronization. The receiver even can't know the start of next frame.

2. Flag bytes with byte stuffing

    * Description

        Use a **flag byte** as both the starting and ending delimiter, shown as below:

            | FLAG | HEADER |    PAYLOAD    | TRAILER | FLAG |
      
        There are cases that the payload contains the flag byte, in which case the sender need to stuff a same flag byte before it to escape, while the receiver should remoe the first flag byte if it encounter two contiguous flag bytes. This is called **byte stuffing**.

    * Con

        This method is tied to the use of 8-bit bytes.

3. Flag bits with bit stuffing.

    * Description

        The delimiter is now changed from a flag byte to a bit pattern (e.g. 01111110/0x7E). Whenever the sender's data link layer encounters five consective 1s, it automatically stuffs a 0 bit at the end. When the receiver sees five consecutive incoming 1 bits, followed by a 0 bit, it automatically destuffs the 0 bit.

4. Physical layer coding violations.

    * Description

        Add redundency to help the receiver. For example, in the 4B/5B line code 4 data bits are mapped to 5 bitss to ensure sufficient bit transitions. This means that 16 out of the 32 signal possibilities are not used. We can use some reserved signals to indicate the start and end of frames.

### 3.1.3 Error Control

Data link layer needs to ensure the frames are eventually delivered to the network layer only once.

1. ACK

    For the acknowledged services, normally the protocal asks the receiver to acknowledge to the sender if one frame is correctly received or not. Then the sender will judge if need a resend. 

2. Sender Timer

    To avoid the sender's from waiting forever for the ack (e.g. sender's frame is lost in link, which means there will be no ack), the sender should start a timer once sending a frame. When timeout, it should send the frame again.

3. Numbered Frames

    For the 2nd case, if the ack is lost in link, then the sender re-send the frame, which will cause the receiver to accept the same frame twice. To avoid this, it is necessary to assign sequence number to outgoing frames so the receiver could distinguish retransmissions from originals.

### 3.1.4 Flow Control

The data link layer needs to ensure a fast sender will not swamp a slow receiver. Two approaches are commonly used:

1. Feedback-based flow control (link layer and higher layers)

    The receiver sends back information to the sender giving it permission to send more data, or at least telling the sender how the receiver is doing.

2. Rate-based flow control (transport layer)

    The protocol has a built-in mechanism that limits the rate at which senders may transmit data, without using feedback from receiver.

## 3.2 Error Detection and Correction

There are two strategies for dealing with errors:

1. Error-correcting (Forword Error Correction, abbr. FEC)

    By using error-correcting codes, the receiver is able to deduce what the transmitted data must have been.

    (used on non-reliable channels)

2. Error-detecting

    By using error-detecting codes, the receiver is able to deduce that an error has occured(but not which error).

    (used on reliable channels)

### 3.2.1 Error-Correcting Codes

Four codes are mentioned:

1. Hamming codes

    * To reliably detect *d* errors, you need a distance *d*+1 code (i.e. distance between two closest leagle codes >= *d*+1 );

    * To reliably correct *d* errors, you need a distance 2\**d*+1 code.

        (m+r+1) <= 2**r

    Look [here](http://users.cis.fiu.edu/~downeyt/cop3402/hamming.html) for the detailed calculation process.

2. Binary convolutional codes

    Soft-decision decoding, could handle 'uncertainty' case.

3. Reed-Solomon codes

4. Low-Density Parity Check codes

### 3.2.2 Error-Detecting Codes

Three codes are mentioned:

1. Parity

    A single **parity bit** is appended to the sent data.

    If one block only has one parity bit, and there is long burst error garbling the block, then the probability of detecting the error is only 50%. We can regard each block to be a n\*k rectangle, then we can either append one parity bit for each row (i.e. up to k bit errors) or *interleave* the parity bit for each column. In this case, we can improve the detection rate in long burst case.

2. Checksums

3. Cyclic Redundancy Checks (CRCs)
