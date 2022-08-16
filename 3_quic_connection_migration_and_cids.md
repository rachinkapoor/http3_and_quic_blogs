# QUIC and HTTP/3 Features - Connection Migration and CIDs

## Connection Migration

QUIC is designed for the new-age technology reality with a decidedly mobile-first outlook. So, it packs in a connection-migration function that lets you continue with the same connection when you switch networks. 
This is great for on-the-go mobile workers and users as they hop on and off from Wi-fi's to cellular networks on the same connection without losing time on setting up fresh ones.

Keeping the same connection intact holds the promise of seamless file downloads and videos even when hopping networks. Also, the reduced cost of not setting a fresh connection should provide some speed gains.

![QUIC Connection Migration Demo. Credits : Yu Yu](./img/QUIC_connection_migration_demo.gif)

Let us take a deeper look into how QUIC achieves this. First, let us look at how the old TCP sets up and remembers connections.

### Connection Setups with the Old TCP - The 4-Tuple Connection Info

TCP was never built with frequent network changes for mobile devices in mind. So, when a client and server setup a TCP connection, TCP used a 4-tuple info package to remember the connection:

-  **The Client IP Address**
-  **The Server IP Address** 
-  **The Client Port Number**
-  **The Server Port Number**

We need the last two values, the client and server port numbers to identify the different apps on these machines that run on different ports. 

Now, if a user on a mobile network switches to a fixed line network, TCP cannot know it's the same client sending data to the same server over a new network. No such information is in-built into the 4-tuple format TCP uses to identify clients and servers!

TCP can only set up a fresh connection between the same client and server. As we know, you need a new handshake to setup a new connection that involves 2 or 3 RTTs with TCP/TLS. So, you lose speed when you hop networks. For videos, this means lags and hiccups and for file requests, it means new transmissions. 

Worse, congestion control in networks ( this is a mechanism to prevent sending more data than a connection can handle) starts slowly for new connections before peaking to the optimum speed. So, a fresh connection on a network change means you start again from the slow speed phase - not very performant!

### Remembering Clients and Servers - QUIC Connection IDs

QUIC solves this problem by adding another value to make it a 5-tuple to remember a connection's end-points. We call these values **Connection IDs ( or CIDs)**. 

Now, when you hop networks, but use the same IP addresses and port numbers, QUIC can match the CIDs also to know its the same old connection. So, it does not waste resources in setting new handshakes. 

To dive a little deeper, a single CID for a connection isn't actually great for security. It gives hackers a very easy way to track their victims across networks once they get their hands on a CID. 

![Connection Migration in HTTP/3 and QUIC . Credits: Robin Marx](./img/QUIC_connection_migration.png)

### Multiple QUIC CIDs - Optimizing for Security

To solve this security glitch with single CIDs, QUIC actually provides clients and servers with a set of CIDs for *different networks* but that signify the *same connection*. So, we may have CID X for network A, CID Y for network B, and CID Z for network C over the *same connection*. Now, these CIDs are encrypted (didn't we say QUIC encrypts everything), so attackers cannot know that these all link to the same connection.

As the user hops from network A to network, the CIDs between him and the server shifts from X to Y, but QUIC knows these both mean the *same connection*. So, it does not waste resources to setup a new one.

### Getting Into the Bones of CIDs - Separate Client and Server CIDs

It turns out even setting separate CIDs for different networks isn't enough for the real-world network realities.

The culprits again are the mysterious **middle boxes** or the devices in between the end-points. So, if you have proxies or load balances sitting in between the server and the client, these middle boxes need to know the CID to route the traffic correctly over QUIC. However, storing all server CIDs will be a memory guzzler for load balancers. So, servers need to choose their own **CIDs in** **pre-decided formats** that load-balancers identify easily to optimize performance. 

But, the Internet has no mechanism to route server-generated CIDs to clients upstream! So, we need a set of separate CIDs at both end-points to factor in load-balancers and proxies!

### QUIC Connection Migration Analysis

Once again, QUIC's Connection Migration feature is an example of its nuanced reality. While some edge cases deliver on its promise of enabling continued file downloads or seemless video on network changes, many users won't benefit much from it.

#### Connection Migration is Rare

  Connection Migration only happens when you hop between **different kinds of networks**. For example, you won't have a connection migration if you change towers while **being on the same cellular network**. Such changes happen lower down on the protocol stack, and you do not get a new IP address for these changes. 

  So, the QUIC connection migration sets in for only a few rare cases - like when you **switch from a Wi-fi to a cellular network**.

#### Connection Migration Use Cases: Large File Downloads

  Connection migration lets you continue large file downloads from the same point if you switch networks without needing to start again.

  QUIC CID feature certainly delivers on this promise. But, again there is a caveat in that you can use an alternate old features to do the same. For example, you can use HTTP Range Requests to resume file downloads from the break-point, taking away some shine off QUIC's connection migration benefits!

#### Connection Migration Use Cases - Video Streaming

   QUIC connection migration also lets you watch videos seamlessly without much lag when swapping networks.

  Again, there are a few quirks. First, when we switch networks on the same connection with QUIC CIDs, we might not get the **same available bandwidth on the new network** as the old one. The congestion algorithms need to **reset from the slow start-up phase** on the new network again to avoid inundating the network! So, we will see some lag in the videos as we change networks.

  Also, even on the older HTTP/2, videos can open up separate connections on the two networks when switching between them. There is some **time overlap** when the old network fades away and the new one sets in. So, these separate connections help videos sync to provide seamless viewing with only minor lag.

  ### QUIC Connection Migration - Mobile Apps Support

  As we see, the QUIC connection migration feature is relevant when swapping  between cellular networks and broadbands. You would want to thus embed QUIC in mobile apps outside the browser if you look to tap into its connection migration feature.

  #### Apple iOS Support for QUIC

  Apple provides support for QUIC in its Safari browser from iOS 14 onwards. The good news is that beginning iOS 15, you can also use QUIC for apps outside the browser.

  #### Android Support for QUIC

  Android does not natively provide native support for QUIC at the time of this writing. But, you can use the **cromet** network stack in your Android apps to support QUIC. Cromet picks up the Chromium networking stack that the Chrome browser uses and packages it as a library for mobile app usage, and it supports QUIC.

  ### QUIC Connection Migration - The Take-Away

  As we see, continuing on the same connection with QUIC CIDs provides benefits in some use cases.

  So, QUIC connection migration won't make much difference to all your users. But, it would help:

  #### For Mobile-First Apps or Sites that Target On-the-Move Users

  As we saw, connection migration helps when we hop from cellular networks to Wi-fi's and vice-versa, so if your app or site targets a on-the-go mobile user-base, then QUIC connection migration will provide a better user experience. For example, geolocation apps like Google Maps or cab-hiring apps would benefit from QUIC.

  #### For Real-Time Constant Interaction Apps 

  Because the average web page load takes seconds, it is unlikely to coincide with a network change.

  But real-time apps that need continuous interaction like video and audio apps, or gaming apps, would provide a better user experience on QUIC as connection migration reduces hiccups and lags when changing networks.

  ​

  #### 



#### 