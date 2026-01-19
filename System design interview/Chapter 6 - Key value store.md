
# DESIGN A KEY-VALUE STORE

A **key-value store**, also known as a **key-value database**, is a non-relational database where each unique identifier (key) is stored with its associated value. This pairing is called a "key-value" pair. Keys must be unique and can be plain text or hashed values, with shorter keys generally performing better. Examples of keys include "last_logged_in_at" (plain text) or "253DDEC4" (hashed). The value can be strings, lists, objects, or other data types and is typically treated as an opaque object by the key-value store. Popular key-value stores include Amazon Dynamo, Memcached, and Redis.

This chapter focuses on designing a key-value store that supports two primary operations:

- `put(key, value)`: Inserts a "value" associated with a "key".
- `get(key)`: Retrieves the "value" associated with a "key".

**Key Takeaways:**

- A **key-value store** is a non-relational database where unique keys map to values.
- Values are often treated as opaque objects, meaning the store doesn't interpret their content.
- The fundamental operations are `put` (write) and `get` (read).

## Understand the problem and establish design scope

Designing a key-value store involves making tradeoffs between read/write performance, memory usage, consistency, and availability. For this design, the key-value store is characterized by the following requirements:

- **Small key-value pair size**: Less than 10 KB.
- **Ability to store big data**.
- **High availability**: The system must respond quickly even during failures.
- **High scalability**: The system should scale to support large datasets.
- **Automatic scaling**: Servers should be added or deleted automatically based on traffic.
- **Tunable consistency**: The level of data consistency should be configurable.
- **Low latency**.

**Key Takeaways:**

- System design involves **tradeoffs**, especially between consistency and availability.
- The scope includes supporting **small key-value pairs**, **massive data volumes**, and ensuring **high availability, scalability, and low latency**.
- The system should also support **automatic scaling** and **tunable consistency**.

## Single server key-value store

For a single server, an intuitive approach is to store key-value pairs in an **in-memory hash table**. While memory access is fast, this approach faces limitations due to finite memory capacity.

To fit more data on a single server, two optimizations can be applied:

- **Data compression**: Reduce the size of the stored data.
- **Tiered storage**: Store only frequently used data in memory, with the rest residing on disk.

Despite these optimizations, a single server will quickly reach its capacity when dealing with big data, necessitating a distributed key-value store.

**Key Takeaways:**

- A single-server key-value store can use an **in-memory hash table** for speed.
- **Data compression** and **memory/disk tiering** can extend a single server's capacity.
- Ultimately, a single server is **insufficient for big data** and large user bases, requiring a distributed solution.

## Distributed key-value store

A distributed key-value store, also called a **distributed hash table**, distributes key-value pairs across many servers. When designing such systems, understanding the **CAP theorem** is crucial.

### ==CAP theorem==

The **CAP theorem** states that it is **impossible for a distributed system to simultaneously provide more than two of these three guarantees**: consistency, availability, and partition tolerance.

==Here are the definitions of the three properties:==

- **Consistency**: All clients see the **same data at the same time** regardless of which node they connect to.
- **Availability**: Any client requesting data receives a **response**, even if some nodes are down.
- **Partition Tolerance**: The system continues to operate despite **network partitions**, which are communication breaks between nodes.
![[Pasted image 20250804104204.png]]
**Figure 6-1: CAP theorem** **Description:** This figure is a Venn diagram showing three overlapping circles representing Consistency (C), Availability (A), and Partition Tolerance (P). The diagram illustrates that a distributed system can only choose two out of these three properties at any given time. The intersections represent the achievable combinations:

- **CP (Consistency and Partition Tolerance):** Availability is sacrificed.
- **AP (Availability and Partition Tolerance):** Consistency is sacrificed.
- **CA (Consistency and Availability):** Partition Tolerance is sacrificed. **Purpose:** This figure visually represents the fundamental trade-offs imposed by the CAP theorem in distributed system design, showing which combinations are possible.

Key-value stores are classified based on the two CAP characteristics they support:

- **CP (Consistency and Partition Tolerance) systems**: These systems support consistency and partition tolerance while sacrificing availability. For example, a banking system might block operations to ensure data is absolutely consistent if a network partition occurs.
- **AP (Availability and Partition Tolerance) systems**: These systems support availability and partition tolerance while sacrificing consistency. They continue to accept reads (possibly stale data) and writes (which will sync later) even during a partition.
- **CA (Consistency and Availability) systems**: These systems support consistency and availability while sacrificing partition tolerance. The chapter notes that a CA system **cannot exist in real-world applications** because network failures (partitions) are unavoidable in distributed systems.

To further illustrate, consider data replicated across three nodes: n1, n2, and n3.
![[Pasted image 20250804104409.png]]
**Figure 6-2: Ideal situation** **Description:** This figure shows three server nodes (n1, n2, n3) arranged in a triangle, indicating that they are all connected and communicating with each other. Arrows depict data replication occurring seamlessly between all nodes. Clients are shown accessing any of these nodes. **Purpose:** This diagram depicts a theoretical scenario where network partitions never occur, allowing a system to achieve both consistency and availability simultaneously, as data is always synchronized across all replicas. It serves as a contrast to real-world distributed systems.
![[Pasted image 20250804104500.png]]
**Figure 6-3: Real-world distributed systems** **Description:** This figure shows three server nodes (n1, n2, n3). Node n3 is depicted as disconnected or "down," unable to communicate with n1 and n2, representing a network partition. The diagram then presents a decision point:

- ==**"Choose C" (Consistency):** Operations to n1 and n2 are blocked to prevent inconsistency, leading to unavailability.==
- ==**"Choose A" (Availability):** Operations to n1 and n2 continue, potentially leading to stale data on n1/n2 compared to n3 (if n3 had new data before failing) or n3 having stale data once it recovers (if n1/n2 accepted new writes).== **Purpose:** This figure illustrates what happens when a network partition _does_ occur. It highlights the core dilemma of the CAP theorem: when a partition happens, the system must choose between guaranteeing consistency (blocking operations) or availability (potentially serving/accepting inconsistent data).

**Key Takeaways:**

- The **CAP theorem** forces a choice between **Consistency (C)** or **Availability (A)** when **Partition Tolerance (P)** is a must in distributed systems.
- **CP systems** prioritize data accuracy, potentially at the cost of being temporarily inaccessible during partitions.
- **AP systems** prioritize continuous operation, potentially at the cost of serving stale data or accepting writes that lead to temporary inconsistency.
- **CA systems are theoretical** and not practical in real-world distributed environments because network failures are inevitable.

### System components

This section introduces the core components and techniques essential for building a distributed key-value store, largely drawing inspiration from prominent systems like **Dynamo**, **Cassandra**, and **BigTable**. These include:

- Data partition
- Data replication
- Consistency
- Inconsistency resolution
- Handling failures
- System architecture diagram
- Write path
- Read path

**Key Takeaways:**

- The design of a key-value store involves several **interconnected components** and techniques.
- Concepts from **Dynamo, Cassandra, and BigTable** serve as foundational inspiration.

### Data partition

For large applications, fitting all data on a single server is not feasible. Data is split into smaller partitions and stored across multiple servers. The two main challenges in this process are:

- **Evenly distributing data** across servers.
- **Minimizing data movement** when servers are added or removed.

**Consistent hashing**, a technique discussed in Chapter 5, is an excellent solution for these problems. In consistent hashing, servers and keys are mapped onto a hash ring. A key is stored on the first server encountered when moving clockwise from the key's position on the ring.
![[Pasted image 20250804104834.png]]
**Figure 6-4: Consistent hashing** **Description:** This figure shows a circular hash ring. Server nodes (s0, s1, s2, s3, s4, s5, s6, s7) are positioned at various points around the ring. A key (key0) is also mapped onto the ring. An arrow starting from key0 points clockwise to s1, indicating that key0 is stored on server s1.

Advantages of using consistent hashing for data partitioning include:

- **Automatic scaling**: Servers can be added and removed automatically based on the load.
- **Heterogeneity**: Servers with higher capacity can be assigned more virtual nodes (replicas on the ring), allowing for proportional distribution of data and load.

**Key Takeaways:**

- **Data partitioning** is necessary for large datasets that cannot fit on a single server.
- **Consistent hashing** is a key technique for evenly distributing data and minimizing data re-shuffling when servers are added or removed.
- It supports **automatic scaling** and handles **heterogeneous server capacities** through virtual nodes.

### Data replication

To ensure **high availability** and **reliability**, data in a key-value store must be replicated asynchronously across N servers, where N is a configurable parameter. These N servers are chosen by walking clockwise from a key's position on the hash ring and selecting the first N servers to store data copies. ==When using virtual nodes, care must be taken to choose unique physical servers to ensure the replicas are truly distinct.==
![[Pasted image 20250804105140.png]]
**Figure 6-5: Data replicated at s1, s2 and s3 (N=3)** **Description:** This figure shows a hash ring with multiple server nodes (s0 to s7). A key (key0) is mapped to a position on the ring. From key0's position, arrows extend clockwise to three distinct server nodes: s1, s2, and s3. These three servers are highlighted to indicate they store replicas of key0's data. 

For enhanced reliability, especially against outages like power failures or natural disasters, replicas are typically placed in **distinct data centers** connected by high-speed networks.

**Key Takeaways:**

- **Replication** across N servers is essential for **high availability and reliability**.
- Replicas are selected by traversing the consistent hash ring clockwise from the key's position.
- For optimal reliability, replicas should be placed in **geographically distinct data centers**

### Consistency

When data is replicated across multiple nodes, it needs to be synchronized to maintain **consistency**. **Quorum consensus** is a mechanism used to guarantee consistency for both read (R) and write (W) operations.

Here are the definitions related to quorum consensus:

- **N**: The **total number of replicas** where a data item is stored.
- **W**: The **write quorum size**. For a write operation to be considered successful, it must be acknowledged by at least W replicas.
- **R**: The **read quorum size**. For a read operation to be considered successful, it must receive responses from at least R replicas.
![[Pasted image 20250804105653.png]]
**Figure 6-6: N=3 example** **Description:** This figure depicts a "Coordinator" acting as a proxy between a "Client" and three replica nodes (s0, s1, s2) where data is replicated (N=3). For a write operation:

- The Client sends a write request to the Coordinator.
- The Coordinator forwards the write to all three replicas (s0, s1, s2).
- The example shows that if W=1, the Coordinator only needs to receive an acknowledgment from one replica (e.g., s1) for the write to be considered successful, without waiting for s0 and s2.

The configuration of W, R, and N involves a **tradeoff between latency and consistency**.

- If W = 1 or R = 1, operations are faster because the coordinator only waits for one replica's response.
- If W or R > 1, the system offers better consistency but queries are slower because the coordinator waits for responses from the slowest required replica.

**Strong consistency is guaranteed if W + R > N** because this condition ensures there is always at least one overlapping node that holds the latest data. For example, with N=3, W=2, R=2, `W + R = 4 > N = 3`, guaranteeing strong consistency.

**Table: How to configure N, W, and R to fit our use cases?** **Title:** Possible configurations for N, W, and R **Headers:**

- Configuration
- Description **Data Summary:**
- `R = 1` and `W = N`: Optimized for **fast read** operations.
- `W = 1` and `R = N`: Optimized for **fast write** operations.
- `W + R > N`: Guarantees **strong consistency** (e.g., N=3, W=R=2).
- `W + R <= N`: **Strong consistency is not guaranteed**.

The chapter also discusses different **consistency models**:

- **Strong consistency**: Any read operation returns the **most updated write data item**, meaning a client never sees outdated data. This is typically achieved by blocking new operations until all replicas are consistent, which can impact availability.
- **Weak consistency**: Subsequent read operations **may not see the most updated value**.
- ==**Eventual consistency**: A specific form of weak consistency. Given **enough time**, all updates propagate, and all replicas become consistent. Dynamo and Cassandra adopt this model. It allows inconsistent values to enter the system from concurrent writes, and clients are responsible for reconciling them.==

**Key Takeaways:**

- **Quorum consensus** (N, W, R) is used to control the consistency level of replicated data.
- The sum `W + R > N` **guarantees strong consistency**.
- There's a **tradeoff between latency and consistency** based on W and R values.
- **Eventual consistency** is often chosen for **highly available systems** like Dynamo and Cassandra, as it doesn't block operations.

### Inconsistency resolution: versioning

Replication, while providing high availability, can lead to inconsistencies among replicas. **Versioning** and **vector clocks** are employed to solve these problems by treating each data modification as a new, immutable version.
![[Pasted image 20250804110247.png]]
**Figure 6-7: Same values in replicas** **Description:** This figure shows two replica nodes, n1 and n2, both containing the "original value" for a data item D. A client can perform a `get("name")` operation on either server and receive the same value.
![[Pasted image 20250804110319.png]]
**Figure 6-8: Conflict versions** **Description:** Building on Figure 6-7, this figure shows that both n1 and n2 simultaneously receive requests to modify the "name" data item.
- Server 1 changes "name" to "johnSanFrancisco".
- Server 2 changes "name" to "johnNewYork". This results in two conflicting versions, v1 and v2, in the system.

A **vector clock** is a `[server, version]` pair associated with a data item. It helps determine if one version precedes, succeeds, or conflicts with others. When a data item D is written to a server Si:

- If `[Si, vi]` exists in the vector clock, `vi` is incremented.
- Otherwise, a new entry `[Si, 1]` is created.
![[Pasted image 20250804110420.png]]
**Figure 6-9: How vector clock works** **Description:** This figure illustrates the evolution of a data item D and its associated vector clock through a series of client writes to different servers (Sx, Sy, Sz).

1. **Client writes D1 to Sx:** Vector clock `D1[(Sx, 1)]`.
2. **Client reads D1, updates to D2, writes to Sx:** Vector clock `D2[(Sx, 2)]`. (D2 overwrites D1).
3. **Client reads D2, updates to D3, writes to Sy:** Vector clock `D3[(Sx, 2), (Sy, 1)]`.
4. **Client reads D2, updates to D4, writes to Sz:** Vector clock `D4[(Sx, 2), (Sz, 1)]`. (This happens concurrently with step 3, creating a conflict).
5. **Client reads D3 and D4, resolves conflict, writes D5 to Sx:** Vector clock `D5[(Sx, 3), (Sy, 1), (Sz, 1)]`. **Purpose:** This figure provides a concrete, step-by-step example of how vector clocks are updated with each data modification and how they capture the lineage and concurrent modifications of a data item, enabling conflict detection.

**Conflict detection** using vector clocks:

- Version X is an **ancestor** (no conflict) of Y if `Y.version_counter >= X.version_counter` for all common `[server, version]` pairs.
    - Example: `D([s0, 1], [s1, 1])` is an ancestor of `D([s0, 1], [s1, 2])`.
- Version X is a **sibling** (conflict exists) of Y if there is any server in Y's vector clock with a counter less than its corresponding counter in X.
    - Example: `D([s0, 1], [s1, 2])` and `D([s0, 2], [s1, 1])` indicate a conflict.

Downsides of vector clocks:

- **Client complexity**: Clients need to implement conflict resolution logic.
- **Vector clock size**: The `[server: version]` pairs can grow rapidly. A common fix is to set a threshold for length and remove the oldest pairs if exceeded, though this can lead to less accurate descendant relationship determination. However, Amazon's Dynamo has not encountered this as a production problem.

### Handling failures

Failures are inevitable and common in large-scale systems, so designing mechanisms to handle them is critical.

#### Failure detection

In a distributed system, it's not enough to trust a single source; typically, **at least two independent sources** are required to mark a server as down.

- **All-to-all multicasting**: A straightforward solution where every server checks every other server, but it's **inefficient** for many servers.
![[Pasted image 20250804110640.png]]
**Figure 6-10: All-to-all multicasting for failure detection** **Description:** This figure shows a network of interconnected server nodes (s0, s1, s2, s3, s4). Each node has arrows pointing to every other node, illustrating that each server is actively monitoring or communicating with all other servers to detect failures.

- **Gossip protocol**: A more efficient and decentralized method. It works as follows:
    1. Each node maintains a **node membership list** (member IDs, heartbeat counters).
    2. Each node **periodically increments its heartbeat counter**.
    3. Each node periodically **sends heartbeats to a set of random nodes**, which then propagate this information further.
    4. When nodes receive heartbeats, they **update their membership lists**.
    5. If a heartbeat hasn't increased for a predefined period, the member is considered **offline**.
![[Pasted image 20250804110822.png]]
**Figure 6-11: Gossip protocol for failure detection** **Description:** This figure shows server nodes (s0, s1, s2, s3) and a table representing node s0's "Node Membership List." In the list, s2 (member ID = 2) has an outdated heartbeat. Node s0 then sends heartbeats containing s2's status to other random nodes (e.g., s1, s3). These nodes confirm s2's inactivity, leading to s2 being marked as "down," and this information propagates. 

#### Handling temporary failures

The **strict quorum approach** can block read/write operations when servers are temporarily unavailable. To improve availability, a technique called "**sloppy quorum**" is used.

- **Sloppy quorum**: Instead of strict enforcement, the system chooses the **first W healthy servers for writes** and the **first R healthy servers for reads** on the hash ring, ignoring offline servers.
- **Hinted handoff**: If a server is unavailable, another server temporarily processes its requests. When the original server comes back online, the data changes are pushed back to it to ensure consistency.
![[Pasted image 20250804111100.png]]
**Figure 6-12: Sloppy Quorum and Hinted Handoff** **Description:** This figure shows a hash ring with servers s0, s1, s2, s3. Server s2 is shown as "unavailable."

- **Sloppy Quorum:** A client attempts to write data that would normally go to s2. Due to s2's unavailability, s3 (the next healthy server clockwise on the ring) is chosen to temporarily handle the write.
- ==**Hinted Handoff:** Once s2 comes back online, s3 "hints" (sends) the data back to s2, allowing s2 to catch up and restore consistency.==

**Key Takeaways (Temporary Failures):**

- **Sloppy quorum** enhances availability by allowing operations to proceed with a subset of healthy replicas when a full quorum is not possible due to temporary failures.
- **Hinted handoff** ensures data consistency by transferring temporarily handled data back to the original node once it recovers.

#### Handling permanent failures

Hinted handoff is effective for temporary failures, but not for permanently unavailable replicas. For permanent failures, an **anti-entropy protocol** is implemented to keep replicas in sync.

- **Anti-entropy**: Involves comparing each piece of data on replicas and updating each replica to the newest version.
- **Merkle tree**: Used for inconsistency detection and minimizing the amount of data transferred during anti-entropy.
    - **Merkle tree definition**: A hash tree where every non-leaf node is labeled with the hash of its child nodes' labels or values. This allows efficient and secure verification of large data structures.

**Merkle Tree Construction Example (Key space 1 to 12):** 
**Figure 6-13: Divide key space into buckets** **Description:** This figure shows the initial step of constructing a Merkle tree. The total key space (1-12) is divided into a fixed number of buckets (4 in this example). Each bucket covers a specific range of keys, such as "Bucket 1: 1-3", "Bucket 2: 4-6", "Bucket 3: 7-9", and "Bucket 4: 10-12".

**Figure 6-14: Hash each key in a bucket** **Description:** This figure shows the second step. Within each bucket defined in Figure 6-13, individual keys are hashed. For example, in "Bucket 1: 1-3," keys 1, 2, and 3 are shown with their respective hash values (hash(1), hash(2), hash(3)).

**Figure 6-15: Create a single hash node per bucket** **Description:** This figure shows the third step. For each bucket, the individual key hashes are combined and hashed again to create a single hash value for the entire bucket. For instance, "Bucket 1" now has a single hash "hash(1-3)" representing all keys within it.

**Figure 6-16: Build the tree upwards till root by calculating hashes of children** **Description:** This figure shows the final step in constructing a Merkle tree. Starting from the bucket hashes (e.g., hash(1-3), hash(4-6)), pairs of these hashes are combined and hashed upwards to form parent nodes. This process continues level by level until a single root hash is generated at the top of the tree. Inconsistent parts are highlighted, showing that the hashes themselves reveal discrepancies.

To compare two Merkle trees (from two replicas), one starts by comparing their **root hashes**.

- If root hashes match, data on both servers is identical.
- If root hashes differ, the comparison proceeds to the children's hashes (e.g., left child then right child). This recursive traversal identifies exactly which buckets are inconsistent, allowing only those specific buckets to be synchronized, thereby minimizing data transfer.

**Key Takeaways (Permanent Failures):**

- **Anti-entropy protocols** and **Merkle trees** are used to synchronize replicas after permanent failures.
- **Merkle trees** enable efficient detection of inconsistencies by comparing hashes at different levels, minimizing the data that needs to be transferred for synchronization.

#### Handling data center outage

To build a system capable of handling a complete data center outage (due to power, network, or natural disasters), it is crucial to **replicate data across multiple data centers**. If one data center goes offline, users can still access data through the other available data centers.

**Key Takeaways (Data Center Outage):**

- **Cross-data center replication** is a critical strategy for mitigating the impact of large-scale outages and ensuring continuous data availability.

### System architecture diagram

Having discussed various technical considerations, the chapter presents the overall architecture for a distributed key-value store.
![[Pasted image 20250804111505.png]]
**Figure 6-17: System architecture for a distributed key-value store** **Description:** This figure shows a high-level overview of the key-value store architecture.

- Clients interact with a **Coordinator** node.
- The Coordinator communicates with multiple **Nodes** that are distributed on a **hash ring**.
- These Nodes are also interconnected for replication and failure detection (implied by previous sections).

Main features of this architecture are:

- Clients communicate via **simple APIs**: `get(key)` and `put(key, value)`.
- A **coordinator** node acts as a proxy between the client and the key-value store.
- Nodes are distributed on a ring using **consistent hashing**.
- The system is **completely decentralized**, allowing automatic addition and removal of nodes.
- Data is **replicated at multiple nodes**.
- There is **no single point of failure (SPOF)**, as every node shares the same responsibilities.
![[Pasted image 20250804111652.png]]
**Figure 6-18: Node components** **Description:** This figure shows a single "Node" box, illustrating that each node in the distributed key-value store is responsible for multiple functions. Inside the node are components such as:

- Failure Detection
- Request Coordination
- Data Partition
- Concurrency Control
- Storage (Commit Log, Mem Table, SSTable)
- Replication
- Consistency Protocol (Quorum, Vector Clock)
- Anti-Entropy

**Key Takeaways:**

- The system architecture is **decentralized**, with each node capable of handling various responsibilities, thus eliminating single points of failure.
- A **coordinator** acts as an intermediary between clients and the distributed nodes.
- Key principles like **consistent hashing** for distribution and **replication** for fault tolerance are integrated into the core design.

### Write path

The proposed write path, largely based on Cassandra's architecture, details what happens when a write request is directed to a specific node.
![[Pasted image 20250804111721.png]]
**Figure 6-19: Write path** **Description:** This figure illustrates the sequence of operations for a write request.

The steps are:
1. The write request is first **persisted on a commit log file**. This ensures durability even if the node crashes.
2. Data is then **saved in the memory cache** (also known as a Mem Table). This provides fast writes and reads for recently written data.
3. When the memory cache is full or reaches a predefined threshold, the data is **flushed to SSTable** (Sorted-String Table) on disk. An **SSTable** is defined as a **sorted list of `<key, value>` pairs**.

**Key Takeaways:**

- The **write path** emphasizes **data durability** and **performance**.
- Writes are first logged to a **commit log** for immediate persistence, then to an **in-memory cache (Mem Table)** for fast access.
- Data is eventually flushed from memory to **SSTables (Sorted-String Tables)** on disk for long-term storage.

### Read path

The read path prioritizes retrieving data quickly, starting with in-memory checks before resorting to disk access.
![[Pasted image 20250804111829.png]]
**Figure 6-20: Read path for data in memory** **Description:** This figure shows a simplified read path for when data is present in memory.

1. A "Client" sends a read request to a "Node."
2. The Node checks its **"In-Memory Cache."**
3. If the data is found in the In-Memory Cache, it is directly returned to the Client. 

If the data is not in memory, it must be retrieved from disk. To efficiently locate the correct SSTable on disk, a **Bloom filter** is commonly used. A **Bloom filter** is a **space-efficient probabilistic data structure used to test if an element is a member of a set**.

![[Pasted image 20250804112026.png]]
**Figure 6-21: Read path for data not in memory** **Description:** This figure details the read path when the requested data is not found in the in-memory cache.

1. The "Node" first checks the "In-Memory Cache." If not found, it proceeds.
2. The Node then consults a **"Bloom Filter."**
3. The Bloom Filter indicates which **"SSTables"** (Sorted-String Tables) on disk _might_ contain the key (it can have false positives, but no false negatives).
4. The Node queries the relevant SSTable(s) on disk.
5. The SSTable(s) return the data.
6. The Node returns the data to the "Client."

The steps for reading data not in memory are:

1. The system first checks if the data is in the **memory cache**. If not, it proceeds to the next step.
2. The system then checks the **Bloom filter**.
3. The Bloom filter helps determine which **SSTables** might contain the key.
4. The relevant SSTables return the dataset.
5. The result is returned to the client.

**Key Takeaways:**

- The **read path** prioritizes fast in-memory lookups.
- For data not in memory, a **Bloom filter** is used to efficiently identify potential **SSTables** containing the key, minimizing slow disk access.

## Summary

This chapter covers numerous concepts and techniques for designing a distributed key-value store. The table below summarizes the key features and the corresponding techniques used:
![[Pasted image 20250804112245.png]]
**Table: Features and corresponding techniques used for a distributed key-value store** **Title:** Features and corresponding techniques used for a distributed key-value store **Headers:**

- Feature
- Technique **Data Summary:**
- **Big Data / Scalability:** Data Partitioning (Consistent Hashing, Virtual Nodes)
- **High Availability / Reliability:** Replication, Quorum (N, W, R), Sloppy Quorum, Hinted Handoff, Merkle Tree
- **Consistency:** Tunable Consistency (W + R > N), Eventual Consistency, Vector Clock
- **Low Latency:** Read/Write Path Optimization, Mem Table, SSTable, Bloom Filter
- **Automatic Scaling:** Consistent Hashing
- **Failure Detection:** Gossip Protocol **Purpose:** This table serves as a concise recap of the entire chapter, mapping the desired features of a distributed key-value store to the specific system design techniques employed to achieve them. It acts as a quick reference for the key takeaways.

---

**Overall Summary of Chapter 6: Design a Key-Value Store**

Chapter 6 aims to provide a comprehensive guide to designing a highly available, scalable, and low-latency distributed key-value store capable of handling massive amounts of data. The core objective is to build a system that supports basic `put(key, value)` and `get(key)` operations efficiently in a distributed environment.

The chapter begins by establishing the fundamental requirements for such a system, emphasizing small key-value pairs, vast data storage, high availability, scalability, automatic scaling, tunable consistency, and low latency. It quickly moves beyond a single-server setup, acknowledging its limitations for large-scale applications.

The central theme of the distributed design is navigating the **CAP theorem**, which forces a critical choice between consistency and availability in the face of inevitable network partitions. The chapter explains how different configurations of **quorum consensus (N, W, R)** allow for tunable consistency, with eventual consistency often being favored for highly available systems like Dynamo and Cassandra.

Data distribution is achieved through **consistent hashing**, ensuring even load balancing and minimal data redistribution when nodes are added or removed, further enhanced by the use of virtual nodes. For resilience and availability, **data replication** across multiple, distinct servers (and even data centers) is implemented.

Inconsistency, a byproduct of replication, is managed through **versioning and vector clocks**, which allow the system to detect and resolve conflicting data modifications, often requiring client-side logic for reconciliation.

Robust **failure handling** is another critical aspect, with the chapter detailing:

- **Failure detection** using decentralized methods like the **gossip protocol**.
- Handling **temporary failures** with **sloppy quorums** and **hinted handoffs**, ensuring continued availability.
- Addressing **permanent failures** through **anti-entropy protocols** and **Merkle trees** for efficient data synchronization.

Finally, the chapter lays out the **overall system architecture** and dives into the optimized **write and read paths**. The write path ensures data durability via a commit log and fast in-memory writes to a Mem Table before flushing to SSTables on disk. The read path prioritizes cache lookups and employs **Bloom filters** to efficiently locate data within SSTables on disk, minimizing latency.