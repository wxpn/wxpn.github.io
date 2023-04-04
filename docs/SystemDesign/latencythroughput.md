## Latency And Throughput
These are extremly important to understand in the context of system design. These are two important measures to describe the performance of a system.

### Latency
How long it takes for data to travel from one point in a system to another point? For example, latency in network - how long will it take for data to go from the client to the server and back from server to the client. 

Now if you are on a server, and you are reading some data from disk or memory, so time take to read the data is also called as latency. So depending on how the system is built the latency  might differ, some may take longer and some may be shorter. Its important to remember the following magnitudes:
- Reading 1MB data sequencially from memory -> 250 micro secs
- Reading 1MB data sequencially from SSD -> 1,000 micro secs
- Sending 1MB data over 1GBPS network -> 10,000 micro secs ( e.g making a API call)
- Reading 1MB data over HDD -> 20,000 micro sec
- Sending one packet from CA -> NL -> CA -> 150,000 micro secs.

When we design a system, we must optimise the system by lowering the overal latency. This also depends on the application or system as well, some application such as online games which need millisec response and extremely low latency while playing the game, then there are other application which do not really care that much about latency but are more interested in availability and uptime.

### Throughput
How much work a machine can perform in a given period of time? We are referring how much data can be transfered from one point in a system to another point in a system in a given amount of time. Measured in GBPS ( Gigabits per second ).

When there are multiple clients trying to send data to a server, how much can the server handle in a given amount of time (e.g per second). So how to increase throughput or how do we optimize a system for throughput? The simple answer is that you pay to increase throughput. For example if take an example a wxpn.github.com, the thing that determines the how many bits that can go in and out of wxpn.github.com server is determined by the Github. 

But increasing number of throughput does not solve all the problems. So in this example, to avoid bottleneck on a single server, best method would be to distribute the incoming traffic across multiple servers.

Another important point to understand is that even though both latency and throughput are very much related and important when it comes to measuring a systems performance, they are not necessarily corelated. For example, if you have a part of a system that requires very low latency, in other words, they support very fast data transfers. But you then might have a part of the system which has a very low throughput. This means they advantage which we had in previous part of the system with the low latency is now cancelled out with the current part of the system. In other words, you cannot make assumptions about latency and throughput based on the other, these are co-related things.
