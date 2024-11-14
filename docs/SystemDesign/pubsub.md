# Publish/Subscribe Pattern
Going back to example of the chat application we discussed wherein we used streaming. Now imagine that we expand that system to a large scale distributed system which is likely want we want when we build a real product. We would encounter some problems real quickly

	- handling network partition ?
	- what can we do when we lose connection with the server ?
	- what happens when server die and then what happens to the data/messages ?

The chat application is not the best application as this does not seem super important but what happens when we are designing a stock-broker application, then protecting data becomes really important. So persistent storage is very important when designing distributed systems. Then we are thinking that this problem could be solved with a database? But its not that simple. But a typical database is not we want for any and all applications. Say the client triggers a asynchronous operation ( a non-blocking operation ) and that operation goes to the server which take some time to complete, and eventually when it completes the response has to go back to client. This is not something we can store in the database. Even if we decide to build a custom storage solution in the server, this is where ideally the business logic of the system resides and because we want to have a good seperation of duties, this solution is not ideal. This is where **publish/subscribe pattern** comes handy.

What is a publish/subscribe pattern ? Is a paradijm that consist of 4 entities:
- publishers - are essential going to be servers.
- subscribers - these are the client listening for data. The subscribers are subscribed to a topic.
- Topics - Conceptualy speaking, these are channels.
- Message

```
  				-----------T1----------
[P1]  ->           M2, M1           <----- [S1]
					-----------------------			|
																			|
  				-----------T2----------			|
[P2]  ->           M2, M1           	^-- [S2]
					-----------------------

  				-----------T3----------
[P3]  ->           M2, M1           <---- [S3]
					-----------------------
```

Here subscribers ( or clients ) do not communicate with the publishers directly, they subscribe to topics and publishers dont communciate with the client ( subscribers), they just publish data to topics. They pubs and subs do not know each other, they only know topics, where one sends and other listens for data/information. Multiple subscribers can subscribe to a topic. When we say message, we are not referring to a chat application, we are here reffering to a data block, commands, etc.

Every message that sends/published to a topic, the topic will keep a record of all the subscribers subscribed to it and every message also has a message id. When a message is pushed out to the subscribers, the subscribers are going to send an acknowledge back to the subscribers.

In the topic of persistence storage, when message published into the topic these are guarenteed to be comsumed atleast ones. You might encounter a situation (M1) may get pushed to S1. Just as its been pushed out to S1 the connection between S1 and T1 was disconnected and S1 was unable to send an ACK to T1. So when S1 connects back, the T1 is going to assume that M1 was not consumed as no ack was recieved, therefore T1 would resend the message M1. But, in reality S1 has recived M1 twice. This is called idempotent operation, ( lets say M1 was an operation ) which means that no matter now many times the operation was performed the result would always be the same. Conversly, no every operation can be idempotent for example "youtube like" counter, an operation to increase counter cannot be idempotent as this they lead to duplicate likes, so this depends on the operationl goal.

Note that topic is generally a queue, and works like a FIFO. For example of trading application, sending execution order need to be performed sequentially, and chat application the messages should be send the in the order we send them.

Another interesting property is that we can replay messages, there are multiple pub-sub solutions which allows to rewinding the messages from a snapshot in time. The topic is kind of a database although it may not confirm the typical database which we know, but we can store different types of data, i.e T1 may have stock prices and T2 may have crypto prices, and alerts to T3, etc. We can add more business logic on the subscriber level, for example S1 can have a busines logic which collects stock prices are from tech companies alone. S2 may have some other business logic such as for non-tech companies, etc.

Some of the popular solution is Kafka, google clound pub-sub.

[back](../SystemDesign.md)