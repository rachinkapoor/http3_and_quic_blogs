# QUIC and HTTP/3 Features - Head of Line Blocking Removal

## QUIC Multiplexing of Individual Byte Streams and HOL Blocking Removal

Transmitting multiple files in a session **efficiently** has always been a challenge in computer networks. QUIC  promises to optimize this function with its new **multiplexing of individual byte streams **feature.

Let us take a deeper look into this QUIC/HTTP3 quirk.

### Multiple Connections and Multiplexing in HTTP/2 and Prior Versions

Back in the days of HTTP/1.1, we could only send one file over one connection. HTTP/1.1 also let us open 6 to 30 connections at the same time.

While this served the purpose earlier, this mechanism was not enough as web apps grew big and complex. One needed to send hundreds of resources with a page load, and the upper cap of 30 was not good enough.

So, we started **multiplexing** resources into a **single byte stream** with HTTP/2.

We would bundle several files into one **big file**. We would then transmit this **byte stream** over a single connection.

While this let us send multiple resources at speed in a single connection, it led to a new challenge - the HOL blocking problem.

### Head of Line (HOL) Blocking Problem

Imagine we multiplex or bundle 3 resources, A, B, and C and send them over HTTP/2.

We could bundle the packets making up these resources in different patterns.

For example, we could choose to **multiplex** them like so:

AABBCCAABBCC...

Now, computer networks are lossy things. So, it might happen we may suffer a **packet loss** as we transmit this multiplexed group of resources.

Suppose, we lose a packet of the resource B on the way.

The resource B arrives in a corrupt state on the end-point and obviously needs to be resent to be useful.

But, the problem with HTTP/2 multiplexing is that HTTP/2 does not recognize the **individual resources that it multiplexes**. So, it has no way of knowing if the lost packet belongs to A, B, or C because it does not recognize A,B, or C as **distinct** entities inside its multiplexed bundle.

So, when we suffer a packet loss, HTTP/2 resends the **whole bundle** with all 3 resources - A,B, and C. This is clearly a costly network operation.

![HOL Blocking problem - You need to retransmit GET 2 to complete the original GET 3 transmission. Credits Wireshark Developer Conference](./img/HOL_Blocking.png)

### QUIC Fix to the Head of Line Blocking Problem

QUIC fixes the above by **identifying and being aware of the individual byte streams inside the multiplexed bundle**. 

So, if we send our multiplexed bundle example above over QUIC, like so:

AABBCCAABBCC...

and we lose a packet of resource B on the way, QUIC recognizes that **only resource B has lost a packet ** and is thus corrupt when it arrives at the end-point. QUIC knows that the other two files, A and C, have arrived successfully.

So, it **retransmits only file B** again saving crucial network resources to optimize performance.

![QUIC Solves HOL Blocking by tracking individual byte streams. Credits: TheDataDaddi](./img/QUIC_multiplexing_indibytestreams.png)

### The Nuanced Quirks to the QUIC Fix to HOL Blocking

As with all other things QUIC, its HOL Blocking Removal feature also has nuances to it.

First, let us take a closer look at resource multiplexing.

### Multiplexing Scheme 

We could multiplex our resources in several patterns. Continuing our example of multiplexing three files, A, B, and C, we could choose different **multiplexing schemes** , like so:

- AAAABBBBCCCC
- AABBCCAABBCC
- ABCABCABCABC

The first case would be a **sequential multiplexing scheme**. The last one would be a **round-robin** multiplexing scheme. And the second one would be a sort of a hybrid scheme - somewhere in the middle of the first and the last ones.

### Stream Prioritization

Now, the resources we want to multiplex to send over the Internet have different priorities.

- **Render Block Resources**

  The browser cannot paint the page afresh or do run new JavaScript while it loads **render block** reources. Even more, we need to **download render-block resources in full ** before we can use them. So, obviously such render-block resources carry the **highest priority**.

  CSS and JS files are examples of render-block resources.

  Let us look at what happens when we multiplex render block resources in different multiplexing schemes.

  - **Render Block Resources in Round Robin Multiplexing**

    Suppose we pack in three render block resources in a round-robin scheme, like so:

    ABCABCABCABC...

    With this scheme, need all three resources to completely download **before we can use any of them**.

  - **Render Block Resources in Sequential Multiplexing**

    If on the other hand, we packed A,B, and C in a **sequential mutliplexing scheme**, we would something like this:

    AAAABBBBCCCC...

    Now, resource A arrives in full earlier than B and C, and we can **use it right away** without waiting for B and C to arrive.

    Comparing with the round-robin case above, we see that clearly a **sequential multiplexing scheme** is better for **render-block** resources.

    ​

- **Non Render Block Resources**

  On the other hand, **non render block resources** like HTML and image files benefit from a round robin scheme. As these resources do not block front-end rendering and can be gradually loaded, a round-robin scheme provides them with HOL blocking removal capabilities.

  It turns out most resources for web pages are **render-block**. So, a **sequential multiplexing scheme** is better than a round-robin scheme for **web page performance**.

### Packet Loss Recovery with QUIC HOL Blocking Removal

- **Sequential Multiplexing Scheme**

  Suppose we lose a packet in a sequential multiplexed bundle of 12 packets, like so:

  AAAAAABBBBBB...

  Here, we could not put in a packet of C as all 12 slots fill up with A and B packets.

  Now, if we lose a packet of A (or B), we need to **retransmit** that resource but we also do not  successfully deliver C!

  Clearly, the byte stream here is HOL blocked.

- **Round Robin Multiplexing Scheme**

  Now, suppose we pack the files in a round-robin scheme, like so:

  ABCABCABCABC...

  If we now lose a packet of A (or B), that stream needs to be resent. But, we still successfully transmit C!

  So, a **round robin multiplexing scheme** works best for **packet loss recovery** and **HOL Blocking Removal**.

### QUIC Quirk - A Case of Two Opposing Forces

So, we see that two opposing network forces nullify the benefits of QUIC's **individual byte stream recognition** feature.

While web page optimization favors a sequential multiplexing scheme, a round robin scheme is better for packet loss recovery and HOL blocking removal.

### QUIC HOL Blocking Removal Take-away 

Most users won't benefit much from QUIC's HOL Blocking removal because of other network factors.

But, HOL blocking removal will provide performance benefits in cases where we have few **render block resources**, or when we can **load resources incrementally**, or where **less data transmits**. A few such use cases could be :

- **Heavily Cached Web Pages**

- **Single Page API Calls**

  ​

