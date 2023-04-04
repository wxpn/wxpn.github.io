# Availability
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

[back](../SystemDesign.md)