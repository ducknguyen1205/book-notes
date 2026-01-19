
# Summary
The journey of building a complex system begins with simple steps, similar to how "a journey of a thousand miles begins with a single step". The initial setup discussed in the chapter involves everything running on a **single server**. This includes the web application, database, cache, and other components. This basic setup is illustrated in **Figure 1-1**, which depicts a single box labeled "Server" connected to a "User", with components like Web App, Database, and Cache residing within the server box. While simple, this configuration has a major limitation: it does not support failover or redundancy.

To address the lack of failover and redundancy, the chapter introduces **Database Replication**. This is a common technique, often using a master/slave relationship. The **master database** typically supports only write operations (like insert, delete, or update commands), while **slave databases** receive copies of the data from the master and support only read operations. Since most applications have a much higher ratio of reads to writes, there are usually more slave databases than master databases. **Figure 1-5** illustrates this with a "Master DB" connected to multiple "Slave DB" instances.

The design discussed in Figure 1-5 can handle database failures. If a slave database goes offline, read operations can be temporarily directed to the master database, or to other healthy slave databases if multiple are available. A new slave database will be brought online to replace the failed one. If the master database goes offline, a slave database can be promoted to become the new master. All database operations would then temporarily execute on this new master. A new slave database replaces the old one for data replication immediately. The source notes that promoting a new master in production systems is more complex due to potential data inconsistencies in the slave, which might require data recovery scripts. More complex replication methods like multi-masters and circular replication are mentioned but are beyond the scope of this book.

The chapter then shows an updated system design after adding a **Load Balancer** and database replication in **Figure 1-6**. In this setup, a user obtains the IP address of the load balancer via DNS. The user connects to the load balancer, which routes the HTTP request to one of the web servers (e.g., Server 1 or Server 2). A web server reads user data from a slave database for read operations and routes data-modifying operations (write, update, delete) to the master database.

To improve load and response times, adding a **Cache layer** and shifting static content to a **Content Delivery Network (CDN)** are discussed.

A **Cache** is described as a temporary storage area that keeps the results of expensive operations or frequently accessed data in memory to speed up subsequent requests. This mitigates performance issues caused by repeatedly calling the database. A separate **Cache tier** provides benefits such as better system performance, reduced database workloads, and the ability to scale the cache independently. **Figure 1-7** shows a web server interacting with a cache server before querying the database.

A caching strategy mentioned is **read-through cache**. Here, a web server first checks the cache for the requested data. If the data is in the cache, it is returned directly to the client. If not, the web server queries the database, stores the response in the cache, and then sends it back to the client. Cache servers typically offer APIs for common programming languages.

Several **considerations for using cache** are highlighted:

- **Decide when to use cache**: Use cache when data is frequently read but infrequently modified. Cache data is stored in volatile memory, so it's not suitable for persisting important data, which should be saved in persistent data stores.
- **Expiration policy**: Implementing an expiration policy is good practice to remove stale data from the cache. Setting the expiration too short can lead to excessive reloads from the database, while setting it too long can result in stale data.
- **Consistency**: Keeping the data store and cache synchronized is challenging, especially across multiple regions, as data-modifying operations on both are not in a single transaction. A Facebook paper on scaling Memcache is referenced for further details.
- **Mitigating failures**: A single cache server can be a **Single Point of Failure (SPOF)**. Using multiple cache servers across different data centers is recommended to avoid SPOF. Overprovisioning memory is also suggested to provide a buffer.

**Content Delivery Network (CDN)** is discussed for caching static content like JavaScript, CSS, images, and video files. Dynamic content caching is noted as a newer concept beyond the book's scope. CDN improves load time by serving static content from a server geographically closest to the user. The further a user is from a CDN server, the slower the website loads. **Figure 1-9** is mentioned as a great example showing how CDN improves load time. (The text provides an example: users in LA accessing content faster than users in Europe if CDN servers are in San Francisco).

The chapter contrasts **Stateful** and **Stateless architecture** for the web tier.

- In a **stateful architecture**, user session data and profile information might be stored on specific servers (e.g., User A's data on Server 1). This requires subsequent HTTP requests from the same client to be routed to that specific server, often using sticky sessions in load balancers, which adds overhead and makes adding or removing servers, or handling server failures, more difficult. **Figure 1-12** illustrates this by showing users tied to specific servers.
- In a **stateless architecture**, state data is stored in a shared data store external to the web servers. HTTP requests can be handled by _any_ web server, which fetches the necessary state data from the shared store. This approach is simpler, more robust, and scalable. **Figure 1-13** shows users connecting to multiple web servers that share a data store. **Figure 1-14** shows an updated design where session data is moved out of the web tier into a persistent data store (a NoSQL store is chosen for scalability). With a stateless web tier, autoscaling (adding or removing web servers based on traffic) is easily achieved.

To further scale and decouple system components so they can be scaled independently, a **Message Queue** is introduced. A message queue is a durable component stored in memory that supports asynchronous communication, acting as a buffer and distributing asynchronous requests. The basic model (**Figure 1-17**) involves producers/publishers creating messages and publishing them to the queue, while consumers/subscribers connect to the queue and process the messages. Adding message queues makes the system more loosely coupled and failure resilient. **Figure 1-19** shows an updated design including a message queue, along with logging, monitoring, metrics, and automation tools (though only one data center is shown).

As data grows, databases become overloaded, necessitating **Database scaling**. Two broad approaches exist: **vertical scaling** (increasing a server's capacity, e.g., adding more CPU/RAM) and **horizontal scaling** (adding more servers). Horizontal scaling is discussed further.

**Horizontal scaling** of the database is often achieved through **sharding**. Sharding involves splitting data into smaller partitions (shards) and storing them on multiple database servers. A hash function is used to map data keys to specific shards. An example is sharding based on user IDs using `user_id % N`, where N is the number of shards (**Figure 1-21**). The **sharding key** (or partition key) is crucial, determining how data is distributed and enabling efficient routing of queries to the correct database. An important criterion for choosing a sharding key is ensuring even data distribution. **Figure 1-22** illustrates a sharded user table using `user_id` as the sharding key.

Sharding is effective but introduces complexity and challenges. **Resharding data** is needed when a shard grows too large or data distribution is uneven, requiring updating the sharding function and moving data. Consistent hashing (discussed in Chapter 5) is a technique to help solve this.

Finally, to support rapidly increasing data traffic, the system employs sharding for databases and moves some non-relational functionalities to a **NoSQL data store** to reduce database load (**Figure 1-23**). An article on NoSQL use cases is referenced.

The chapter concludes by summarizing key techniques used to scale the system to support millions of users:

- Keep web tier stateless.
- Build redundancy at every tier.
- Cache data as much as possible.
- Support multiple data centers.
- Host static assets in CDN.
- Scale data tier by sharding.
- Split tiers into individual services.
- Monitor your system and use automation tools.

Scaling is an iterative process, and these techniques provide a foundation for tackling larger challenges. The chapter ends with a congratulatory note.