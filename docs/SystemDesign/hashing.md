# Hashing

A hashing function is a specific data type ( such as a string or an identifier) and outputs a number. Different input may have the same output, but a good hashing function minimizes those hashing collisions.

```
____																		____
|C1|																		| A|
|__|\												/	---->			|__|
	   \   								/
____	 \							/								____
|C2|		 \					/									| B|
|__|\			 \  		____									|__|
		\			----->	|LB| -------------->
____		\--->			|__|									____
|C3|									\									| C|
|__|----------------		\	--------->		|__|
									 |	   \ 
____							 |			 \						____
|C4| --------------|				\	------>		| D|
|__|																		|__|

```
Four clients are going to be speaking with servers with loadbalancer in the middle. When a load balancer has to reroute a request from the client to the server, it must have some kind of server selection strategy to pick what server to reroute that traffic to. While lot of these server strategies are fine to begin with but depending on the system, these may introduce some problems. Lets assume for seconds these request that client are making, are requests which are computationally expensive and take lot of time for server to process. One way to deal with this is caching. The server can store in memory caches where every request that goes through the server is cached, and if the response is cached, it skips the operation which it otherwise has to do and immediately returns the cached response.

The above aproach has an issue if the loadbalancer uses a roundrobin approach for server selection because if the client1 makes the same request again and this time, loadbalancer select another server ( not the server selected previously ), then the new server would need to do the computation again. So this aproach does not make good use of in-memory caching, and system is not getting nearly as many cache hits as its supposed to. This is where **hashing** comes to picture.

With hashing, we can hash the request that come into the server, for the sake of the example, lets use client1 and hash the name. The hashing function would return result in integers:

```
			hash(c1) = 11
			11 % 4 = 3 -> server 3

			hash(c2) = 12
			12 % 4 = 0 -> server 1

```

Now we have another problem, we are dealing with large scale distributed system, we could have more than four server and if begin mod every time, we are only sending traffic to 4 servers. So, this above solution would fail if one of these servers fail or when we want to add more servers or remove servers. So, its not possible we change logic everytime, we add or remove servers. But even if we create a new logic to change the mod, the mod function would now return a new number. For example, lets say we add a server, so mod 5.

```
			hash(c1) = 11
			11 % 5 = 3 -> server 1

			hash(c2) = 12
			12 % 5 = 0 -> server 2
			.
			.
			.
			hash(c5) = 12
			12 % 5 = 3 -> server 2
```
So this solution does not scale well. So how do we solve this problem? This can be solved with **Consistent Hashing** and **Rendezvous Hashing**.

## Consistent Hashing
Imagine that we have circle and we have placed the servers on the circle in an even distributed manner. We position the server on the cicle based on the value from the hashing function. Similarly we position the clients on circle also based on the hashing function. So now we have positioned the server and client on the circle. Now, we need to determine to which server the request need to be sent from the client.

We go to each client and either move in clockwise or anticlockwise direction, from each client and the first server your encounter, is the server where the request would be routed. So when we remove a server, the next server closest to the client would receive the requests going forward. And when we add a new server, the client closest to the new server ( in whichever direction is being following - clockwise/anticlosckwise ) would receive the new request.

So, with this when we remove/add a server, we still maintain most of the previous mapping and association remain, and only a few servers and clients were affected.

Suppose we want to even more evenly distribute the traffic, because with the above method, its possible that certain servers are may receive less traffic and certain server may receive more traffic as some client may be closer to them. So we can pass the server through multiple hash function, which means that there maybe multiple locations for any given server. 

Another situation when there some powerful servers among these servers, and you want such server to receive more traffic when compared to other normal servers. So we can pass this particular server more hashing functions, that way this server would be more number of locations, which means that server would be getting more traffic.

## Rendezvous Hashing
Say there are multple clients and multiple servers. Now there are two sets of servers - set1 and set2.

Set1 has multiple servers. So the client1 would rank the servers in the set1 based on some logic. The client1 would pick the highest ranking server from the list. The step is then repeated for every client on set1 servers. So every client would have a highest ranked server.

So what happens when we remove a server from set1? So, now only the one of client would get affected due to this, so client would then choose th second highest ranking server. This is how **Rendezvous** hashing works.

[back](../SystemDesign.md)