# Caching
Caching is used to increase the speed and and lower the latency of the system. When we discussed latency we discussed few examples of different operations or data transfers which took a different amount of time. We can imagine that caching is a method that can used to improve the latency when using certain type of methods such as the network, which took considerable amount of time. Now with caching, this latency can be significantly reduced by storing the data in cache which has considerably lower latency.

- We can use cache is number of places such as at the 
	- client level - client caches some data value such it does not need to go to the server to retrieve it, 
	- similarly cache can be at the server level such that server does not need to lookup database to retrieve data. It only need go the database just ones.
	- You can have caching at the hardware level 
		- CPU Caches ( make it faster to retrieve data from the memory )
		- L1 or L2 cache
- Some instance where caches are useful:
	- When there are lot of network requests and we want to avoid doing all these n/w requsts. In our example:	
	```cpp
		client --> server --> database
		client <-- server <-- database 	
	```
	In this case, we can keep the cache at the client level or the server level, such most of the n/w can be avoided.

	- Some computationaly long request so in that case we can have cache at the server level, so that server need no do this computational request everytime.
	
	- Imagine we have multiple servers and several client communicating with the server, and all these requests are hitting the database. When there are millions of requests per second this might overload the database for instance. So here we can use caching to not having to read from the database everytime. So we can use caching to not be able to read from database everytime. In this siutation we are trying to speed up the process but rather prevent the database from overloading. Here the caching may be placed between the server and the database, or each server may have the cache.

	- Now where do we store the cache ? The cache could be in the server, or as in an independent component, for exmaple **redis**, a popular in memory database, which is a key-value store.
		- An example of that would in algoexpert when user runs the code ( provided by the algoexpert.io ), in this case, the server need not run this computational task everytime the same code is run. Since the code was provided by the algoexpert, they can use a cache to immediately provide an answer. They way they do this is by first creating a hash of the code, which the key stored in the cache, and answer is the value of this key. So whenever a user use the code from algoexpert.io, the hash of the code is compared with the hash stored in the cache and corresponding value associated with that key and returned to the user.

	- So far we have seen examples where we read from the cache, we shall now look at examples where write to cache. If there is an application where users can read and write posts. So there is a client->server->database. So client make requests to the server to write a post, and these posts are stored in the database. So if want to cache the posts, we can do that on the server. Now we have two sources of truth, one on the cache and second on the database. How do we handle this? There are two concepts here : 

		- **Write-Through Cache** - A type of caching systems where when you write a piece of data, your system will write that piece of data both in the cache and in the database, in the same operation. So in our example, where user edits the post, the data in the first overwritten on the cache and another request is made to overwrite the data in the database. So downside is that you need to still go to the database, so every single time you make a change, you need to changes twice and go to the database.

		- **Write-Back Cache** - A type of caching system where only the cache is updated with the data, and not the server. Then, the database is updated asynchronously at a later period in time. One of the downsides, if something ever happens in the cache, you lose the data before being updated to the database asynchronously.

	- Caching can get really complicated when there are several client and several servers, each of these servers have dedicated caches. If this situation arises, here we can take example of youtube commenting system wherein one client make a comment which is cached in the respective caches on the server. If at a later time, the same client makes an edit to its comment and now because this comments was not synchronized with the database, its possible that another client still responnds to the older version of the comments from the user instead of the updated comment from the first user. This situation arises due to staleness of the data as the data was not updated properly.
		- A solution would be move cache out of the servers and maitain a single cache, something like cache and thereby serving as a single point of truth.

	- Its also important to understand that there may be parts of the system wherein we may not care that much about the staleness/non-stateness of the data. For example, the view count on a video is not necessarily the most important thing that need to be highly accurate. This is something we need to ask the interviewer to understand the what is importance to be cached in the system.

	- Some important points are: 
		- Caches works beautifully with immutable data ( or static information such as questions on a website ).
		- But if are dealing with data that is mutable then there are two different location where the data exists, make sure the data is in sync as data can become stale. 
		- Consider using caches when there is only a single thing reading or writing that data. The second we introduce more client reading/writing data, things can get tricky.
		- If staleness or data consistency is not an issue then its you can use caching.
		- Suppose you are able to design a system such that you are able to invalidate the data in the caching especially in a distributed environment. This brings us to **eviction** policy.

	- **Eviction Policy** - Discuss with the interviewer and determine the best approach possible when designing a system.
		- You remove the least recently used data in a cache and you have some way of tracking which peice of data was **least recently used**. 
		- There is also **least frequently** used data
		- You can also remove data based **last-in-first-out**
		- **Randomly**

[back](../SystemDesign.md)