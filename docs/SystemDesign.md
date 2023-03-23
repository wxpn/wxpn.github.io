---
layout: default
---

# System Design
System design is the process of defining the architecture, components, modules, interfaces, and data for a system to satisfy specific requirements. It involves creating a detailed plan for a system's construction and operation, including the selection of hardware and software components and their integration into a functional whole.

System design can be applied to a wide range of fields, including software development, electrical engineering, and mechanical engineering. The goal of system design is to create a system that meets the needs of its users and stakeholders, is reliable, efficient, and scalable, and is capable of being maintained and updated as needed.

## Client Server Model
A client (browser) requests information from a server, the server(any website which has a/several servers) responds to the client with the requested information. The client does not actually know what the server is. All that it knows that it can communicate with it. It does not know what the server represents.

When you first enter the url into your browser, the browser does not really know what the url is. The browser performs a DNS query to determine the ip address of the website(the server). 

All computer connected to the internet, have ways to find public IP addresses, or discover routes to those ip addresess. That means they can send data to IP addresses, they can send packets of information to the IP address. We can see IP addresses as mail boxes which other machines in the network can send data towards. An example is check the DNS - `dig google.com`

Now, it send an HTTP request to the IP address of the server ( website ). That request so going to have the source address of the client. So when the server receives the request, it knows that it needs to send the response to the client ip address.

We know that servers are machine that are waiting for requests from clients, once they recieve these request, they can identify the source ip address and send response back to the clients. A server usually listens for requests on specific ports. When we need to communicate with a specific machine in the network, we also need to provide what port you need to connect to the server.

When we are talking about the client server model, it turns out that most client actually know which port they should use depending on the protocol they are using. For example:
- http - 80
- dns - 53
- https - 443
So in our example, when we type in the url in the browser, the server understand that client is trying to access http service on the server and service response with the html content on the server to the client.  

## Network Protocols
These are formats for communication. The most important ones are IP, TCP and HTTP.
### IP (Intenet Protocol) Address

The modern internet effectively runs on IP. When a machine(client) in the network interacts with another machine(server) in network, and it sends data to the server. This data is going to be sent as an IP packet. IP Packets are the building blocks of communication between machines over the internet. There are other units beyong ip packets, and ip packets are built up of bytes. Its important to know its structure and it has two machine sections -> **ip header** and **data**.

When we sending lot of information, its likely that it may not fit into a single ip packet, so we need to use multiple packets. When we have multiple ip packets and all that we are using is the ip protocol for sending these packets, then its very possible that some of the packets are going to get lost. So in that case, you are unable to send all the data you are trying to send. So also not guarenteeing the order in which you originally send the packet. So as you see, using only IP protocol, the internet can fall apart quickly. This is where **TCP** comes into the picture.

### TCP ( Transmission Control Protocol)
Its built on top of the IP protocol. So its built to solve the problem explained above. Its respects the order of the packet originally sent so you are guarenteeing the order the destination would recieve is same as its want sent, in a reliable way. And in an error free way, so some packets get corrupted during transmission, then the sender will know and these packets can be sent again.

But we cannot send meaningful data in tcp. With tcp we are usually are sending arbitrary data, which is not very useful. This is where HTTP comes into picture.

### HTTP ( Hyper Text transfer protocol )
HTTP is built on top of TCP. It introduces a higher level abstration above TCP and above IP. This abstraction is request-response paradigm, where one machine sends request to one machine, and this machine response to sending machine. HTTP also gives the opportunity to add further business logic.


## Storage
If we think of any system we are designing, that system will require some form of storage - information on user, process. This is where database comes into picture. A database is where one can store and retrieve data. There are two types of popular storage:
- Disk - Data is persistence even if the server is shutdown.
- Memory - Data is not persistent, and if the server is shutdown or goes down due to an outage, all data is lost. Note that in-memory access is much faster than disk access.

There are several type of database offerings and these are based on the type of the data stored in the database.

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

## Availability
How resistent is the system to failures ? What happens when server to the system fails? Often desribed as fault tolerence? Another way to think about availablility, as the percentage of time, in a given period of time like a month/year for instance are operation enough such that all its primary functions are satisfied. 

In this day and age all application have an implied guarantee of availability. For example, for github.io, when you login, you expect the application to be up and operational. If the  customer is down, the customer may be unhappy leading to losing customers and bad publicity.

Now they varying degrees of availability depending on the application. In the case of github.io, the application is down for couple of hours, there would be unhappy customer ofcourse, but this would not be the end of the world. But, imagine the application is used for ground control in an airport, and if this application goes down, then this would not be unaccepted.

So, how do we measure availability? We measure availability as the percentage of the systems upime in a given year. So if the system is available for half of the year, then availablity   
would be 50%. This is bad score for availability.

Since most of the systems are available most of the time, we often measure this based on **Nines**. These are effectively percentages, percentages with number 9. So if the system 99% available? Then we have 2 9's of availability. Similarly 99.9% ( 3 nines ) and similarly 99.999 ( 5 nines ). So more 5 nines ( 99.999 ) -> regarding as highly available systems.

Availability is so important these days that most systems instead of having an implied guarantee of availability, have explicit guarantee of availability. 
- Many service providers have **SLA (Service Level Agreement)**. An agreement between service providers and the customers on the systems availability - The service providers would guarantee this percentage of availability.
- An **SLO ( Service Level Objective )** - It is an component of SLA, the percentage of uptime guarantee is called as **SLO**.

Even though availability is important and is of value when designing a system, however one must understand that achieving high availability is very difficult and comes with certain trade-offs. So having very high availability ( say 99.999's) is also not very important always. This is because high availability may lead to higher latency and lower throughput so as a system designer have to think long and hard to decide which parts of the system needs be highly available and which do not. E.g When you take a system and disect its components, its becomes very clear that there are some core component which needs to be highly available such as a payment systems for instance. And there would be parts of systems, such as dashboard for monitoring sales or metric systems need not be highly available, its still bad but not as bad as payment systems going down. So when you design systems we must understand the parts of the system that needs to be highly available and the parts that need not be.

How do we actually improve the availability of a system? You must ensure that first and foremost the system does not have single point of failure. That is if these components fail, this causes the entire application to fail. You can eliminate single point of failure using **reduntancy**. Reduntancy is the act of duplicating or multiplicating part of the system which are considered a single point of failure.

- For example if there is an server which handles all the incoming request to the main application, then if this server goes down, then the application may be inaccessible. This becomes an single point of failure. A solution would be add number of redundant servers.

Now to handle the incoming traffic to these redundant servers, we now need a load balancer between the clients and the servers. But now the load balancer is a single point of server. So now we can have more reduntant load balancers as well. This way we can eliminate SPOF. So now any point in time, if servers are down or load balancers are overwhelmed, there are other load balancers which can handle the load and systems will continue performing.

- **Passive Redundancy** - When ther are multiple machine working together such that, such as load balancers for examples, among 3 lb, one of them fails, then nothing much will fail, the system would continue processing, other two server may suffer a little more load until the server comes back up.
- **Active redundancy** - When you have multple machines that work together such that only one or few machine would be typically handling traffic. If this machine fail, then the other machine would be informed of this failure and would take over. This is done via what called is as **leader-election**.

It is also important to consider that its not always software, but we also need to have processes in place when there is a failure to ensure availability. Some system failures require a humen intervention, server crashes and you need a human to bring that up. So we need to have proceses in place to bring the server up in the proper timeframe.

## Caching
Caching is used to increase the speed and and lower the latency of the system. When we discussed latency we discussed few examples of different operations or data transfers which took a different amount of time. We can imagine that caching is a method that can used to improve the latency when using certain type of methods such as the network, which took considerable amount of time. Now with caching, this latency can be significantly reduced by storing the data in cache which has considerably lower latency.

- We can use cache is number of places such as at the 
	- client level - client caches some data value such it does not need to go to the server to retrieve it, 
	- similarly cache can be at the server level such that server does not need to lookup database to retrieve data. It only need go the database just ones.
	- You can have caching at the hardware level 
		- CPU Caches ( make it faster to retrieve data from the memory )
		- L1 or L2 cache
- Some instance where caches are useful:
	- When there are lot of network requests and we want to avoid doing all these n/w requsts. In our example:	
	```cpp
		client --> server --> database
		client <-- server <-- database 	
	```
	In this case, we can keep the cache at the client level or the server level, such most of the n/w can be avoided.

	- Some computationaly long request so in that case we can have cache at the server level, so that server need no do this computational request everytime.
	
	- Imagine we have multiple servers and several client communicating with the server, and all these requests are hitting the database. When there are millions of requests per second this might overload the database for instance. So here we can use caching to not having to read from the database everytime. So we can use caching to not be able to read from database everytime. In this siutation we are trying to speed up the process but rather prevent the database from overloading. Here the caching may be placed between the server and the database, or each server may have the cache.

	- Now where do we store the cache ? The cache could be in the server, or as in an independent component, for exmaple **redis**, a popular in memory database, which is a key-value store.
		- An example of that would in algoexpert when user runs the code ( provided by the algoexpert.io ), in this case, the server need not run this computational task everytime the same code is run. Since the code was provided by the algoexpert, they can use a cache to immediately provide an answer. They way they do this is by first creating a hash of the code, which the key stored in the cache, and answer is the value of this key. So whenever a user use the code from algoexpert.io, the hash of the code is compared with the hash stored in the cache and corresponding value associated with that key and returned to the user.

	- So far we have seen examples where we read from the cache, we shall now look at examples where write to cache. If there is an application where users can read and write posts. So there is a client->server->database. So client make requests to the server to write a post, and these posts are stored in the database. So if want to cache the posts, we can do that on the server. Now we have two sources of truth, one on the cache and second on the database. How do we handle this? There are two concepts here : 

		- **Write-Through Cache** - A type of caching systems where when you write a piece of data, your system will write that piece of data both in the cache and in the database, in the same operation. So in our example, where user edits the post, the data in the first overwritten on the cache and another request is made to overwrite the data in the database. So downside is that you need to still go to the database, so every single time you make a change, you need to changes twice and go to the database.

		- **Write-Back Cache** - A type of caching system where only the cache is updated with the data, and not the server. Then, the database is updated asynchronously at a later period in time. One of the downsides, if something ever happens in the cache, you lose the data before being updated to the database asynchronously.

	- Caching can get really complicated when there are several client and several servers, each of these servers have dedicated caches. If this situation arises, here we can take example of youtube commenting system wherein one client make a comment which is cached in the respective caches on the server. If at a later time, the same client makes an edit to its comment and now because this comments was not synchronized with the database, its possible that another client still responnds to the older version of the comments from the user instead of the updated comment from the first user. This situation arises due to staleness of the data as the data was not updated properly.
		- A solution would be move cache out of the servers and maitain a single cache, something like cache and thereby serving as a single point of truth.

	- Its also important to understand that there may be parts of the system wherein we may not care that much about the staleness/non-stateness of the data. For example, the view count on a video is not necessarily the most important thing that need to be highly accurate. This is something we need to ask the interviewer to understand the what is importance to be cached in the system.

	- Some important points are: 
		- Caches works beautifully with immutable data ( or static information such as questions on a website ).
		- But if are dealing with data that is mutable then there are two different location where the data exists, make sure the data is in sync as data can become stale. 
		- Consider using caches when there is only a single thing reading or writing that data. The second we introduce more client reading/writing data, things can get tricky.
		- If staleness or data consistency is not an issue then its you can use caching.
		- Suppose you are able to design a system such that you are able to invalidate the data in the caching especially in a distributed environment. This brings us to **eviction** policy.

	- **Eviction Policy** - Discuss with the interviewer and determine the best approach possible when designing a system.
		- You remove the least recently used data in a cache and you have some way of tracking which peice of data was **least recently used**. 
		- There is also **least frequently** used data
		- You can also remove data based **last-in-first-out**
		- **Randomly**

## Proxies
There are two type of proxies - reverse proxy and reverse proxy

### Forward proxy
A server sits in between a client/set-of-client and server/set-of-server. Its like the client is saying to communicate with the server on behalf of me. The server now processes the request and returns the response to the forward proxy and fw proxy would return to the client.

FW proxy is a way to hide the ip address of the client. This is becase the source ip address from the initial request can be replaced with the proxy ip and sent to the server. That way server see only the ip of the proxy.

### Reverse Proxy
FW acts on the behalf of the client, so a RW proxy acts on the behalf on the server. When a client sends a request to the server, and if there is a reverse proxy confgured, then the request is actually being sent to RW proxy, acting on the behalf of the server. The RW proxy would then send this request to client. The client does not know that its communicating with the rw proxy or an actual server.

In the example of dns resolution of `github.io`, the dns would return the ip address of rw proxy instead of the actual server.

You can configure the rw proxy to handle certain tyes of bad requests, take care of logging, cache certain html pages or other static contents. The best use cases of rw proxy is to use as load balancer ( a server which effectively distribute a load between a bunch of server ). Nginx is a very popular tool which is used as a load balancer and as a reverse proxy.
