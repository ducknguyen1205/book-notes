### **Step 1 - Understand the problem and establish design scope**

The initial step in any system design interview is to clarify the requirements with the interviewer. This involves understanding the specific features to support.

**Candidate-Interviewer Interaction Example:**

- **Candidate:** Is this a mobile app? Or a web app? Or both?
    - **Interviewer:** Both.
- **Candidate:** What are the important features?
    - **Interviewer:** A user can publish a post and see her friends’ posts on the news feed page.
- **Candidate:** Is the news feed sorted by reverse chronological order or any particular order such as topic scores?
    - **Interviewer:** To keep things simple, let us assume the feed is sorted by reverse chronological order.
- **Candidate:** How many friends can a user have?
    - **Interviewer:** 5000.
- **Candidate:** What is the traffic volume?
    - **Interviewer:** 10 million DAU.
- **Candidate:** Can feed contain images, videos, or just text?
    - **Interviewer:** It can contain media files, including both images and videos.

**Requirements Summary:**

- **Functionality:** Users can publish posts and view their friends' posts on the news feed page.
- **Sorting:** News feed is sorted in **reverse chronological order**.
- **User Scale:** Supports 10 million Daily Active Users (DAU).
- **Friends Limit:** A user can have up to 5000 friends.
- **Content Types:** Posts can include text, images, and videos.
- **Clients:** Supports both mobile and web applications.

**Key Takeaways:**

- Always **clarify requirements** with the interviewer to align on expectations and scope.
- The system needs to handle both **publishing and retrieving news feed content**, including media.
- A significant scale of **10 million DAU** is a key metric.
- The news feed sorting is simplified to **reverse chronological order** for this design.

### **Step 2 - Propose high-level design and get buy-in**

The design of the news feed system is logically separated into two main flows: **feed publishing** and **news feed building**.

- **Feed publishing**: Involves writing post data to cache and database, and populating it into friends' news feeds when a user publishes a post.
- **Newsfeed building**: Aggregates friends' posts, in reverse chronological order, to construct the news feed.

### **Newsfeed APIs**

APIs are the primary communication method between clients and servers, designed in a **REST-style** for this system. The two most important APIs are for feed publishing and news feed retrieval.

**Feed publishing API:**

- **Purpose:** To create a new post.
- **Method:** HTTP POST request.
- **Endpoint:** `/v1/me/feed`.
- **Parameters (Params):**
    - `content`: The text of the post.
    - `auth_token`: Used for authenticating API requests.

**Newsfeed retrieval API:**

- **Purpose:** To retrieve a user's news feed.
- **Method:** HTTP GET request.
- **Endpoint:** `/v1/me/feed`.
- **Parameters (Params):**
    - `auth_token`: Used for authenticating API requests.

### **Feed publishing**
![[Pasted image 20250805101031.png]]
**Figure 11-2: Feed publishing high-level design**

- **Flow:**
    1. **User** (left) initiates a post, e.g., "Hello" via an API call.
    2. The request goes through a **Load Balancer**.
    3. The Load Balancer directs the request to **Web Servers**.
    4. **Web Servers** redirect traffic to internal services.
    5. A **Post Service** persists the post in the **Database (DB)** and **Cache**.
    6. A **Fanout Service** pushes the new content to friends' news feeds, storing news feed data in the cache for fast retrieval.
    7. A **Notification Service** informs friends of new content and sends push notifications.

**Key Takeaways:**
- The design emphasizes **real-time propagation** of new posts to friends' news feeds via a ==fanout mechanism.==

### **Newsfeed building**

This section details how the news feed is constructed.
![[Pasted image 20250805101616.png]]
**Figure 11-3: Newsfeed building high-level design**

- **Flow:**
    1. **User** (left) sends a request to retrieve their news feed, e.g., `/v1/me/feed`.
    2. The request goes through a **Load Balancer**.
    3. The Load Balancer directs traffic to **Web Servers**.
    4. **Web Servers** route requests to the **Newsfeed Service**.
    5. The **Newsfeed Service** fetches the news feed from the **Newsfeed Cache**.
    6. The Newsfeed Cache stores news feed IDs required for rendering the news feed.

**Key Takeaways:**

- The process leverages a **Newsfeed Service** to fetch pre-computed news feed data from a **cache** for speed.
- This separation of concerns allows for **independent scaling** of publishing and building processes.

### **Step 3 - Design deep dive**

This section delves deeper into the feed publishing and news feed building flows, focusing on important components and optimizations.

### **Feed publishing deep dive**
![[Pasted image 20250805101727.png]]
**Figure 11-4: Feed publishing detailed design**

- **Components and their functions:**
    - **User:** Interacts with the system to create posts.
    - **Load Balancer:** Distributes incoming traffic to web servers.
    - **Web Servers:** Handle client communication, enforce authentication, and implement rate-limiting. They interact with other services to process posts.
    - **Post Service:** Responsible for persisting post data.
    - **Post Database (DB):** Stores the actual post content persistently.
    - **Post Cache:** Stores recently posted content for fast retrieval.
    - **Fanout Service:** Core component for delivering posts to friends' news feeds.
    - **Graph Database:** Manages and stores friend relationships, suitable for graph-like data queries.
    - **User Cache:** Stores user information for quick access by the fanout service.
    - **Message Queue:** Decouples the fanout service from fanout workers, acting as a buffer for events.
    - **Fanout Workers:** Consume events from the message queue and store news feed data in the news feed cache.
    - **Newsfeed Cache:** Stores `<post_id, user_id>` mappings to represent a user's news feed.
    - **Notification Service:** Sends push notifications and informs users about new content.

**Web servers**

- Beyond client communication, **web servers** are responsible for **authentication** (ensuring only signed-in users with valid `auth_token` can post) and **rate-limiting** (limiting posts per user within a period to prevent spam and abuse).

**==Fanout service==**

- **Fanout** is the process of delivering a post to all relevant friends/followers.
- **Two primary fanout models exist:**
    - **Fanout on write (Push model):**
        - **Mechanism:** News feed is pre-computed at write time. A new post is immediately delivered to friends' caches upon publication.
        - **Pros:**
            - News feed is generated in real-time and pushed immediately.
            - Fetching news feed is fast due to pre-computation.
        - **Cons:**
            - **Hotkey problem:** Can be slow and resource-intensive for users with many friends (e.g., celebrities) due to the need to fetch large friend lists and generate news feeds for all.
            - **Resource waste:** Pre-computing news feeds for inactive users or those who rarely log in wastes computing resources.
    - **Fanout on read (Pull model):**
        - **Mechanism:** The news feed is generated dynamically at read time, meaning recent posts are pulled when a user loads their home page.
        - **Pros:**
            - More efficient for **inactive users**, as resources are not wasted on pre-computation.
            - Avoids the hotkey problem, as data is not pushed to friends.
        - **Cons:**
            - Fetching the news feed can be **slower** as it's computed on demand.
- **Hybrid Approach:** The recommended design uses a **hybrid approach** to combine the benefits of both.
    - A **push model** is used for the majority of users, prioritizing fast news feed retrieval.
    - For **celebrities or users with many friends/followers**, a **pull model** is used to avoid system overload (hotkey problem).
    - **Consistent hashing** can help distribute requests/data more evenly to mitigate the hotkey problem.
![[Pasted image 20250805102001.png]]
**Figure 11-5: Fanout service detailed design**

- **Fanout Service Workflow:**
    1. **Fetch friend IDs** from the **Graph Database**. Graph databases are well-suited for managing friend relationships and recommendations.
    2. **Get friends' information** from the **User Cache**. The system filters friends based on user settings (e.g., muting) or selective sharing preferences.
    3. **Send friends list and new post ID to the Message Queue**.
    4. **Fanout Workers** fetch data from the message queue and **store news feed data in the Newsfeed Cache**.
        - The Newsfeed Cache functions as a `<post_id, user_id>` mapping table.
        - When a new post is made, it is appended to this news feed "table".
        - To keep memory usage low, only **IDs** are stored, not full user or post objects. A configurable limit is set for the cache size, assuming users mostly look at recent content.
    5. **Store `<post_id, user_id>`** in the news feed cache.
![[Pasted image 20250805102121.png]]
**Figure 11-6: News feed cache example**
### **Newsfeed retrieval deep dive**

![[Pasted image 20250805102213.png]]
**Figure 11-7: Newsfeed retrieval detailed design**

- **Description:** This diagram shows a user's request for their news feed going through a load balancer to web servers. Web servers interact with the Newsfeed Service, which fetches Post IDs from the Newsfeed Cache. The Newsfeed Service then retrieves full user and post objects from the User Cache and Post Cache, respectively. Media content (images/videos) is explicitly shown to be served from a CDN. The fully hydrated news feed is returned to the client.
- **Flow of URL Redirecting:**
    1. A **User** sends a request (e.g., `/v1/me/feed`) to retrieve their news feed.
    2. The **Load Balancer** distributes the request to **Web Servers**.
    3. **Web Servers** call the **Newsfeed Service** to fetch news feeds.
    4. **Newsfeed Service** gets a list of **post IDs** from the **Newsfeed Cache**.
    5. To create a complete news feed (including username, profile picture, post content, images), the **Newsfeed Service** fetches full user and post objects from **User Cache** and **Post Cache**.
    6. **Media content (images, videos)** is served directly from the **CDN** for faster retrieval.
    7. The fully **hydrated news feed** is returned in **JSON format** back to the client for rendering.

### **Cache architecture**

Caching is paramount for a news feed system's performance. The cache tier is divided into five distinct layers.

**Figure 11-8: Cache architecture**

- **Description:** This diagram shows a layered cache architecture for a news feed system, categorized into five main types: News Feed, Content, Social Graph, Action, and Counters.
- **Five Cache Layers:**
    - **News Feed:** Stores **IDs of news feeds** (e.g., which posts belong to a user's feed).
    - **Content:** Stores **actual post data**. Popular content is kept in a "hot cache".
    - **Social Graph:** Stores **user relationship data** (e.g., who is friends with whom).
    - **Action:** Stores information about **user actions** on posts (e.g., likes, replies, shares).
    - **Counters:** Stores **numerical counters** (e.g., counts for likes, replies, followers, following).

**Key Takeaways:**

- **Fanout on write (push model)** is efficient for reads but can lead to **hotkey problems** for users with many friends.
- **Fanout on read (pull model)** is resource-efficient for inactive users but **slower for reads**.
- A **hybrid fanout approach** (push for most, pull for celebrities) offers a balanced solution.
- **Message queues** are used to **decouple services** and enable asynchronous processing, especially for fanout workers.
- The Newsfeed Cache stores **only IDs** to conserve memory.
- During news feed retrieval, data is "hydrated" by fetching full objects from **various cache layers** (User, Post, News Feed) and media from **CDN**.
- A **multi-layered cache architecture** (News Feed, Content, Social Graph, Action, Counters) is critical for performance.

### **Step 4 - Wrap up**

This chapter presented the design of a news feed system, encompassing two primary flows: **feed publishing** and **news feed retrieval**.

**Additional Talking Points (if time allows):**

- **Scaling the database:**
    - **Vertical scaling vs. Horizontal scaling**: Discuss adding more power vs. adding more servers.
    - **SQL vs. NoSQL**: Consider different database types based on data structure and access patterns.
    - **Master-slave replication**: A common technique for improving read performance and reliability.
    - **Read replicas**: Dedicated database instances for handling read operations.
    - **Consistency models**: Discuss tradeoffs between strong, weak, and eventual consistency.
    - **Database sharding**: Distributing data across multiple database servers to handle large datasets.
- **Other general system design considerations:**
    - **Keep web tier stateless**: Essential for horizontal scalability and robustness.
    - **Cache data as much as you can**: Maximize caching to reduce database load and improve response times.
    - **Support multiple data centers**: Improves availability and reduces latency for globally distributed users.
    - **Loosely coupled components with message queues**: Enhances resilience, scalability, and maintainability by decoupling services.
    - **Monitor key metrics**: Essential for system health and performance tuning, such as Queries Per Second (QPS) during peak hours and news feed refresh latency.

### **Reference materials**

The chapter refers to the following materials:

- How News Feed Works: https://www.facebook.com/help/327131014036297/
- Friend of Friend recommendations Neo4j and SQL Sever: http://geekswithblogs.net/brendonpage/archive/2015/10/26/friend-of-friend-recommendations-with-neo4j.aspx

### **Overall Summary**

Chapter 11, "DESIGN A NEWS FEED SYSTEM," provides a comprehensive guide to designing a scalable news feed system, addressing challenges inherent in handling millions of daily active users and rich media content. The core objective is to enable users to **publish posts** and **view their friends' posts** efficiently and reliably.

The design is structured around two main, interdependent flows:

1. **Feed Publishing:** When a user creates a post, the system must process it, store it, and disseminate it to the news feeds of relevant friends. This involves **Web Servers** for authentication and rate-limiting, a **Post Service** for persistence in databases and caches, a **Fanout Service** to propagate the post to friends' news feeds, and a **Notification Service** for push notifications. The fanout mechanism employs a **hybrid approach**—primarily pushing content to most users for fast retrieval, but pulling for users with a large number of followers (e.g., celebrities) to mitigate system overload.
2. **Newsfeed Building (Retrieval):** When a user requests their news feed, the system must quickly aggregate relevant posts from their friends and present them. This flow leverages **Web Servers** and a **Newsfeed Service** to retrieve post IDs from a **Newsfeed Cache**, and then "hydrates" these IDs into full post objects (including user info and media) by fetching data from **User Cache**, **Post Cache**, and **Content Delivery Networks (CDNs)**.

The system architecture heavily relies on **caching** across multiple layers (News Feed, Content, Social Graph, Action, Counters) to ensure low latency and reduce database load. **Message queues** are crucial for decoupling system components, allowing for asynchronous processing and increased robustness, especially in the fanout process. While databases are essential for persistent storage of posts, user data, and social graphs, their scalability is addressed through techniques like **replication and sharding**. The overall design prioritizes **scalability, reliability, and high availability**, while also considering important aspects like API design, authentication, and rate-limiting.