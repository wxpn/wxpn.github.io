# Network Protocols
These are formats for communication. The most important ones are IP, TCP and HTTP.
## IP (Intenet Protocol) Address

The modern internet effectively runs on IP. When a machine(client) in the network interacts with another machine(server) in network, and it sends data to the server. This data is going to be sent as an IP packet. IP Packets are the building blocks of communication between machines over the internet. There are other units beyong ip packets, and ip packets are built up of bytes. Its important to know its structure and it has two machine sections -> **ip header** and **data**.

When we sending lot of information, its likely that it may not fit into a single ip packet, so we need to use multiple packets. When we have multiple ip packets and all that we are using is the ip protocol for sending these packets, then its very possible that some of the packets are going to get lost. So in that case, you are unable to send all the data you are trying to send. So also not guarenteeing the order in which you originally send the packet. So as you see, using only IP protocol, the internet can fall apart quickly. This is where **TCP** comes into the picture.

## TCP ( Transmission Control Protocol)
Its built on top of the IP protocol. So its built to solve the problem explained above. Its respects the order of the packet originally sent so you are guarenteeing the order the destination would recieve is same as its want sent, in a reliable way. And in an error free way, so some packets get corrupted during transmission, then the sender will know and these packets can be sent again.

But we cannot send meaningful data in tcp. With tcp we are usually are sending arbitrary data, which is not very useful. This is where HTTP comes into picture.

## HTTP ( Hyper Text transfer protocol )
HTTP is built on top of TCP. It introduces a higher level abstration above TCP and above IP. This abstraction is request-response paradigm, where one machine sends request to one machine, and this machine response to sending machine. HTTP also gives the opportunity to add further business logic.

[back](../)