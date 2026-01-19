
# Summary of Chapter 5: Design Consistent Hashing

==Consistent Hashing is presented as a commonly used technique to achieve **horizontal scaling** by **distributing requests or data efficiently and evenly across servers.**==

### The Rehashing Problem

Before introducing consistent hashing, the chapter highlights a problem with a common approach to load balancing data across `N` cache servers. This approach uses a hash function `hash(key)` and a modular operation: `serverIndex = hash(key) % N`.

An example is provided with 4 servers (N=4) and 8 string keys with their hash values.
![[Pasted image 20250516155803.png]]

Figure 5-1 illustrates this distribution, showing key0, key1, key4 mapped to server 1; key2 mapped to server 3; key3, key5 mapped to server 2; and key6, key7 mapped to server 0. ==This approach works well when the size of the server pool is fixed and data distribution is even.==
![[Pasted image 20250516155822.png]]

However, issues arise when servers are added or removed. If server 1 goes offline, the server pool size becomes 3 (N=3). Using the same hash function `hash(key)` results in the same hash values, ==but the modular operation `hash % 3` gives different server indexes.==
![[Pasted image 20250516155935.png]]
**Table 5-2:** Key redistribution after server 1 goes offline (N=3)

Figure 5-2 shows the new distribution. Most keys are redistributed, not just those originally on server 1. This causes a "storm of cache misses" as clients contact the wrong servers. Consistent hashing is presented as an effective technique to mitigate this problem.
![[Pasted image 20250516155957.png]]
### Consistent Hashing

Quoted from Wikipedia, consistent hashing is a special kind of hashing where, when a hash table is re-sized, **only k/n keys need to be remapped on average**, where `k` is the number of keys and `n` is the number of slots. This is in contrast to traditional hash tables where changing the number of slots causes nearly all keys to be remapped.

### Hash Space and Hash Ring

Consistent hashing uses a hash function `f` (like SHA-1) whose output range forms a hash space, for example, from 0 to 2^160 - 1 for SHA-1. Figure 5-3 illustrates this hash space as a line segment. By connecting the ends of this line segment, a **hash ring** is formed, as shown in Figure 5-4.
![[Pasted image 20250516160101.png]]
### Hash Servers

Using the same hash function `f`, servers (identified by IP or name) are mapped onto this ring. Figure 5-5 shows 4 servers mapped onto the hash ring. Note that the hash function here is different from the one in "the rehashing problem," and there is no modular operation.
![[Pasted image 20250516160221.png]]
### Hash Keys

Cache keys are also hashed onto the same ring using the same hash function. Figure 5-6 shows 4 cache keys (key0, key1, key2, key3) hashed onto the hash ring.
![[Pasted image 20250516160255.png]]
### Server Lookup

To determine which server a key is stored on, you start at the key's position on the ring and **move clockwise until the first server is found**. Figure 5-7 illustrates this process:
![[Pasted image 20250516160448.png]]
- key0 is stored on server 0.
- key1 is stored on server 1.
- key2 is stored on server 2.
- key3 is stored on server 3.

### Add a Server

With consistent hashing, adding a new server requires redistribution of only a fraction of keys. In Figure 5-8, when server 4 is added, only key0 is redistributed. Before server 4 was added, key0 was stored on server 0 (moving clockwise from key0). Now, key0 is stored on server 4 because server 4 is the first server encountered moving clockwise from key0's position. The other keys (k1, k2, k3) remain on their original servers.
![[Pasted image 20250516160521.png]]
### Remove a Server

Similarly, when a server is removed, only a small fraction of keys require redistribution. In Figure 5-9, when server 1 is removed, only key1 must be remapped. Moving clockwise from key1's position, the first server encountered is now server 2. The rest of the keys are unaffected.

### Two Issues in the Basic Approach

The basic consistent hashing algorithm, introduced by Karger et al., involves mapping servers and keys to a ring using a uniform hash function and finding the server for a key by moving clockwise. However, this basic approach has two identified problems:

1. **Uneven partition size**: It's difficult to maintain similarly sized partitions on the ring for all servers, especially when servers are added or removed. A partition is the hash space between adjacent servers. Figure 5-10 shows an example where if s1 is removed, s2's partition becomes twice as large as s0 and s3's partitions.
![[Pasted image 20250516160701.png]]
2. **Non-uniform key distribution**: Keys might not be evenly distributed across servers. For example, if servers are mapped to positions shown in Figure 5-11, most keys could end up on server 2, while server 1 and server 3 hold no data.
![[Pasted image 20250516160741.png]]
### Virtual Nodes (Replica)

To address the issues of uneven distribution, the concept of **virtual nodes** (also called replicas) is introduced. Instead of mapping each server to a single point on the ring, each server is mapped to multiple points (virtual nodes). A virtual node refers to a real server. If each server is mapped to `k` virtual nodes, there will be `N * k` virtual nodes on the ring.
![[Pasted image 20250516160830.png]]
Figure 5-13 shows an example with 3 servers (s0, s1, s2) and each server mapped to 3 virtual nodes (s0_0, s0_1, s0_2, etc.).
![[Pasted image 20250516160901.png]]
To find which server a key is stored on, you go clockwise from the key's location and find the first virtual node encountered. That virtual node points to the real server. For example, in Figure 5-13, to find where k0 is stored, you move clockwise from k0's position and find virtual node s1_1, which refers to server 1.

As the number of virtual nodes increases, the distribution of keys becomes more balanced because the standard deviation gets smaller. Research suggests that with 100 to 200 virtual nodes, the standard deviation is between 5% (200 virtual nodes) and 10% (100 virtual nodes) of the mean, leading to balanced data distribution. Using more virtual nodes improves balance but requires more space to store information about them, presenting a tradeoff that can be tuned to system requirements.

### Wrap up

The chapter concludes with a summary of the benefits and applications of consistent hashing.

The benefits include:

- Minimized keys are redistributed when servers are added or removed.
- Ease of horizontal scaling due to more even data distribution.
- Mitigation of the **hotspot key problem**, where excessive access to a single shard (like for celebrities) can overload a server, by distributing data more evenly.

Consistent hashing is widely used in real-world systems, including:
- Partitioning component of Amazon’s Dynamo database.
- Data partitioning across the cluster in Apache Cassandra.
- Discord chat application.
- Akamai content delivery network.
- Maglev network load balancer.

#### Real-world systems detail:

1. **Amazon Dynamo Database**:
    
    - **Usage**: Dynamo uses consistent hashing to partition data across multiple nodes in a distributed database system.
    - **Problem**: It ensures that each node in the cluster is responsible for a specific range of data partitions.
    - **Benefit**: This approach provides scalability and fault tolerance by allowing nodes to be added or removed without significant reorganization of data.
        
2. **Apache Cassandra**:
    
    - **Usage**: Cassandra employs consistent hashing for partitioning data across nodes in a cluster.
    - **Problem**: It ensures even distribution of data and efficient querying by directing requests to the appropriate nodes based on hashed keys.
    - **Benefit**: Enables high availability and performance scalability while minimizing data movement when nodes are added or removed.
        
3. **Discord Chat Application**:
    
    - **Usage**: Discord uses consistent hashing for routing messages and managing user sessions across its distributed infrastructure.
    - **Problem**: Ensures that messages are reliably delivered to the correct server handling a particular chat channel or user session.
    - **Benefit**: Provides low-latency message delivery and efficient session management, crucial for real-time chat applications.
        
4. **Akamai Content Delivery Network (CDN)**:
    
    - **Usage**: Akamai employs consistent hashing to efficiently distribute content across its global network of servers.
    - **Problem**: Ensures that requests for content are directed to the nearest and most available server to minimize latency and maximize throughput.
    - **Benefit**: Improves content delivery speed and reliability by leveraging a distributed network architecture with intelligent routing.
        
5. **Maglev Network Load Balancer**:
    
    - **Usage**: Maglev uses consistent hashing to distribute incoming network traffic across a set of backend servers.
    - **Problem**: Ensures that each incoming request is directed to the appropriate server based on hashed criteria (such as source IP or session ID).
    - **Benefit**: Optimizes server utilization, improves fault tolerance, and enhances overall network performance by evenly distributing load among backend servers.
        

Consistent hashing, in these contexts, provides a robust mechanism for distributing workload, data, or traffic across distributed systems, ensuring efficient operations and scalability without sacrificing reliability.

# Reference materials

Consistent hashing: https://en.wikipedia.org/wiki/Consistent_hashing
Consistent Hashing: https://tom-e-white.com/2007/11/consistent-hashing.html 
Dynamo: Amazon’s Highly Available Key-value Store: https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf 
Cassandra - A Decentralized Structured Storage System: http://www.cs.cornell.edu/Projects/ladis2009/papers/Lakshman-ladis2009.PDF 
How Discord Scaled Elixir to 5,000,000 Concurrent Users: https://blog.discord.com/scaling-elixir-f9b8e1e7c29b 
CS168: The Modern Algorithmic Toolbox Lecture #1: Introduction and Consistent Hashing: http://theory.stanford.edu/~tim/s16/l/l1.pdf 
Maglev: A Fast and Reliable Software Network Load Balancer: https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44824.pdf