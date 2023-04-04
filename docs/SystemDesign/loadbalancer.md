# Load Balancer

* When a single server can handle minimum amount of requests, the system can only minimum throughput for any given amount of time. In that, when multiple requests from one client or several client, the server sooner or later becomes overwhelmed with the amount of requests. To solve this problem, we can either scale our system. We can either do this horizontal scaling, ie. add more servers, now we can handle more incoming traffic and process more request, but how do we assign the client request to the server, especially when there are several servers. This is where load balancer comes to picture. A load balancer sits between the client and several servers, and basically has the job to balance workloads across resources. In this case, rerouting client requests to the servers in a balanced way.
1. With load balancers, the server can get better latencies
2. Can make better use of the resources on the servers.
3. Can see load balancers as a rw proxy.
4. There are two type of load balancers
	- Hardware loadbalancers
		- Physical machines dedicated loadbalancing, less freedom and less customisation as hardware is limited.
	- Software loadbalancers 
		-  We can do more and have freedom with customization.
5. But how does the load balancers know about the existence of new servers or remove old servers? So when a new servers are added, it registers with the load balancers or deregisters when the servers are removed. The personals can configure load balencers and servers to know about each other.
6. So how does load balaners select which server to redirect traffic to ? 
	- Once can configure loadbalancer to select servers **randomly**, but this can lead to one server being overloaded. To avoid such issues, a round robin approach is used.  So roundrobin, 1st request -> 1st server, 2nd request -> 2nd server, 3rd request -> 3rd server and so on and eventually back to the 1st server, and this way the traffic is evenly distributed. 
	- Another more complex roundrobin approach is called **weighted rountrobin**, you would still assign 1st req -> 1st server and so on, but you will also add some weights to the servers, say 2nd server, which meanns some more requests(a little bit more traffic - maybe 2nd server more powerful machine ) would be sent to the 2nd server as compared to the first server.
	- Another way is based on **performance or based on load**. The loadbalancer perform health checks on the server to identify how much traffic is handling at a given time, how long server is taking to respond to traffic, how many resource the server is using. Based on that the loadbalancer distributes requests.
	- **IP based load balancing** - Based on the ip address of the client the request is distributed. The ip address of the client is hashes and depending on the value, the corresponding server( server number 1 ). Ip based load balancing is very useful if there is caching on the server. In your system, the responses are being caches, so results from a specific client always be redirected to the server, where response of that particular client's request has been cache, that can be accomplishes through IP based load balancing. IP based loadbalancer can maximise the cache hits.
	- **Path based server** selection strategy where the requests are distributed based on the path. All requests for running code "/code" will be redirected to a server, and similarly /payment goes to another server.
7. So you select loadbalancer based on the usecase of your system.
8. You can have multiple load balancers as well. For example you can 1st loadbalancer is IP based RR which redirects to another loadbalancer based on RR. So now what happens when loadbalancer gets overwhelmed? Reemeber to avoid single point of failure, so now instead of one loadbalancer, we can have multple load balancer and each communicating with each other.

[back](../SystemDesign.md)