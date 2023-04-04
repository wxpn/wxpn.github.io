## Relational Databases
* In the storage section previously we mentioned they are hundreds are storage methods, its generally hard to catogorize any given databases. There is however a major category based on which databases can be categorized - **structured** and **unstructured**. If we specifically look at the structure imposed on the data, then databases can be classified as two types - relational databases and non-relatonal databases.

* What is a relational database ? Its a type of database that imposes on the data stored in it a tabular like structure. Data stored in the relational databases, would be stored as tables.
* When dealing tables in a relational database, then data stored in the table has to conform to table schema.
* Most relationsal databases support SQL ( Structures Query Language ). - Very powerful language used to query relational databases. Although most relational databases support SQL, but these cannot be used interchanageably, they are not the same. There are few relational databases that does not support SQL. Because relational databases often support sql, these often come with powerful query capabilties. This can be one reason people use relational databases over non-relational databases.
* A SQL database must use ACID transaction - An operation in a database that has four properties A ( Adomicity ), C ( Consistency) 

- A ( **Adomicity** ), if the trasaction consists of multiple sub-operations. For example, trasferring funds from one account to another, which involves increasing an account funds and decreasing funds. Adomicity dictates that these sub-operations are considered as a unit, even if one of them fail, then the trasaction fails. All need to succeed for the transaction to succeed.
- C ( **Consistency** ), menas that any transaction is going to conform and abide by all the rules of database. There will not stale state in the transaction where one trasaction was executed and another transaction failed without knowing why.
- I ( **Isolation** ), multple trasaction can occur at the same time, and it would seem like they have executed simultaneously, but in reality these would be executed sequentially.
- D ( **Durability** ), when you make a transaction, the affects of these transaction is permanent, which means that transactions are stored in the disk.

## Database Index
Say we want to look at the customer who has the largest amount. Such a operation is a linear time operation, and if they have distributed system and large databases, this operation can take some time. This is where database index become useful. The idea is that we can create an auxiliary datastructure in your database that can be optimized for fast searching on an attribute of the table. There are lot of types of databases - bitmap index, loose index, dense index, etc

For eg. create a database index with amount in ascending order. We could bring a linear operation to logarithmic time.

The trade off is that a axiliary database may take more space and whenever you update/insert data into the database, the database index also needs to be updated. Therefore the write operation is going to slower but read is going to be much faster.

## Key-Value Store
We saw in the previous section that relational database along with SQL have very powerful quering capabilities. However, at some time, the very same structure can prove combuersome than useful, and in those cases we might want to use non-relational databases, aka non-sql database. One of the popular type of non-sql database is a key-value store. A key-value is what you see below:

```
	key     value
	___			_____
	foo     gool
	bar     systemexpert
	baz     1,2,3
```
They are incredembly flexible and simple, **flexible** as they dont have that imposed structure, and its **simple** from a conceptual standpoint. 
- Couple of example that come to mind is caching, in caching, we typically store the response from a request as value and key would be hash value or ip address or username, etc.

**Dynamic Configuration** - Sometimes we want to have special parameters or constants in your systems that different parts of the system can rely on. We can have a key-value pair, for example, `"is_alive":true` which can be constant widely throughout the system.

Using **Key-Value stores**, we are accessing value directly through keys, therefore this results in lowered latencies and improves troughput. 
- There are several key-value stores - redis, zookeeper, dynamodb. Some key-value stores stores data in the disk, and there are some such as redis (in-memory) which store only on memory. We need to pick the one more useful to you.

## Specialized Storage Paradigms
- **Blob Store** - Binary Large Object - some arbitrary peice of unstructured data e.g a video file, image file, binary file. large text file. - unstructured data - specializes in storing large numbers of unstructured data. They behave like a key-value store, because usually we request blob via some key, and therefore it would seems like a key-value store, but these are entirely different functionalities or use-cases. A key-value might not be able to store same amount of data, and more optimized for lower latency.
Some vendors are below:
	- GCS
	- S3
	- Azure Blob

- **Time Series DB** - A database specialized in storing time series data. When we have large amount of data, all relevent to time, events that happen at a given time ( a second or millisecond ). When need to perform very time series like computation, such calculating roling averages, finding maxima or minima, in these situation we might want to use time series database. Some appli,cation which uses this type of database, include monitoring, storing telemetry, storing crypto prices.
	- Influx DB
	- Prometheus	

- **Graph DB** - A graph database is a database built over a graph model. Where there too many relationship between tables of a database
	- neo4j

- **Spatial DB** - This database is optmised for storing spatial data, such as location on the map, restaurants in a country, somtthing like google map, etc.
	- Quadtree  

[back](../SystemDesign.md)