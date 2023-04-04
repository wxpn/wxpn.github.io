# Client Server Model
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

[back](../SystemDesign.md)