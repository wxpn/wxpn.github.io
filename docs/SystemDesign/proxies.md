## Proxies
There are two type of proxies - reverse proxy and reverse proxy

### Forward proxy
A server sits in between a client/set-of-client and server/set-of-server. Its like the client is saying to communicate with the server on behalf of me. The server now processes the request and returns the response to the forward proxy and fw proxy would return to the client.

FW proxy is a way to hide the ip address of the client. This is becase the source ip address from the initial request can be replaced with the proxy ip and sent to the server. That way server see only the ip of the proxy.

### Reverse Proxy
FW acts on the behalf of the client, so a RW proxy acts on the behalf on the server. When a client sends a request to the server, and if there is a reverse proxy confgured, then the request is actually being sent to RW proxy, acting on the behalf of the server. The RW proxy would then send this request to client. The client does not know that its communicating with the rw proxy or an actual server.

In the example of dns resolution of `github.io`, the dns would return the ip address of rw proxy instead of the actual server.

You can configure the rw proxy to handle certain tyes of bad requests, take care of logging, cache certain html pages or other static contents. The best use cases of rw proxy is to use as load balancer ( a server which effectively distribute a load between a bunch of server ). Nginx is a very popular tool which is used as a load balancer and as a reverse proxy.