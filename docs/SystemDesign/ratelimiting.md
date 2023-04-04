# Rate Limit
Basically setting some form of threshold on certain operation past which the operation would return error. Limiting the operation that could be performed within an amount of time.
```
[client] ------| only 3 req per 10 sec |-----> [server ] ------| only 3 req per 10 sec |--------> [database]
```
So when there are more than 3 request in 10 seconds, this request is not processed by the server or the database.
The reason rate limit is so important for certain operation because the application could be brought down by malicious actors by denial of service attack. When a bad actor flood the system with bunch of traffic and basically brings the systems down becuase the system cannot handle traffic anymore. The system does not have enough throuput to handle the traffic and therefore rate limiting prevents the system from being brought down.

Rate limit can be applied for several users:
* Rate limit can be done based on the user - based on user behavior.
* Rate limit on a certain region.
* Rate limit based on ip address

Although the rate limiting may be useful from single machine denial of service, but in the case of difstributed denial of service, this may not useful at all. DDoS attack is caused by several machines and its not possible to rate limit based on ip address or even users or from certain region. 

If we are rate limiting based on users, then we need store the accesses of users in the memory and in the case of a large distributed system with lot of servers, this is not very scalable and rate limiting can fall apart. This is because the in-memory storage of the user is only on one server, and if the user sends the request to another user within the window of error, this server would allow the request to flow through as this accesses were not stored there. \[ This problem could be solved with a very sophisticated load balancing system but this would be too cumbersome. \]
- So in a distributed system its always better to store the accesses in a seperate system such as redis for example, which is a key-value store. This would be seperated-out database and when client would send request the server, the server would check *redis* to if there is a problem with the request, and if yes, the request is blocked and error is returned, or if under threshold the requst is allowed.

[back](../SystemDesign.md)