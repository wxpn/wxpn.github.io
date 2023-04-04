# Map Reduce

The concept of map-reduce is actually very simple, you got a dataset that is spread across multiple machines. There is a **map** function that you as an engineer or the system administrator is going to specify, that function is going to transform the data into the intermediary values ( key-value pairs) and these key-value pairs after being re-organised in some way is **reduced** into some final output. 

There are some important points to note about map-reduce function.

* When we are dealing with a map reduce model, we assume we have a distributed file system, means that we have some large datasets that is split up into chunks. These chunks are likely replicated and spread across muliple machines in order of hundreds/thousands. Our distributed control system has some sort of central control plane that is aware of everything going in the mapreduce job or process. That means that the central control plane has access to all the chunks are, how to communicate with all the machines which store all this data, knows how to communicate with the machine that perform the map function ( aka worker machine ), and also for the reduce operation - know how to communicate with reduce workers. Finally know where the output is going to live.

* Often times when we are dealing these large datasets, we do not want the move the large data, instead we send the map programs to this data operate on them. This is avoid moving this large data.

* The key-value pairs structure in the intermediatery step is very important. Its important because when reducing data values coming from multiple chunks of the same dataset, all these chunks are chunks from the same large dataset, we are likely looking at some form commonality in these various pieces of data. This is why key-value pairs structure is very important. Natually we are going to see some keys being similar so we can agregate them into one single meaningful value.

* On of the main things the model trying to accomplish is failures. If there is machine failure, its going to perform the map or reduce operation again where the failure occured. Its important the operation is idempotent, that means that if we repeat the map function and reduce function again and again, we should see the same outcome.

* As the engineer or the system administrator dealing with the map-reduce job, the main thing they need to are about is what map or reduce function they are going to specify, and what the input going to be, what the output intermediate key-value pair is going to look like, the reduce function and the output would be.

[back](../SystemDesign.md)