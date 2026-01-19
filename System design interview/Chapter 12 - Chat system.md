### Step 1 - Understand the problem and establish design scope

The initial step in a system design interview is to ask clarifying questions to define the requirements.

**Candidate-Interviewer Interaction:**

- **Candidate:** "What kind of chat app shall we design? 1 on 1 or group based?"
    - **Interviewer:** "It should support both 1 on 1 and group chat."
- **Candidate:** "Is this a mobile app? Or a web app? Or both?"
    - **Interviewer:** "Both."
- **Candidate:** "What is the scale of this app? A startup app or massive scale?"
    - **Interviewer:** "It should support **50 million daily active users (DAU)**."
- **Candidate:** "For group chat, what is the group member limit?"
    - **Interviewer:** "A maximum of **100 people**"
- **Candidate:** "What features are important for the chat app? Can it support attachment?"
    - **Interviewer:** "1 on 1 chat, group chat, online indicator. The system only supports **text messages**."
- **Candidate:** "Is there a message size limit?"
    - **Interviewer:** "Yes, text length should be less than **100,000 characters long**."
- **Candidate:** "Is end-to-end encryption required?"
    - **Interviewer:** "Not required for now but we will discuss that if time allows."
- **Candidate:** "How long shall we store the chat history?"
    - **Interviewer:** "**Forever**."

**System Requirements:** For this chapter, the focus is on designing a chat application similar to Facebook Messenger, with emphasis on the following features:

- A **one-on-one chat** with low delivery latency.
- **Small group chat** (maximum of 100 people).
- **Online presence** indicator.
- **Multiple device support**, allowing the same account to be logged in on several devices simultaneously.
- **Push notifications**. The system is designed to support **50 million daily active users (DAU)**.

### Step 2 - Propose high-level design and get buy-in

To develop a high-quality design for a chat system, it's necessary to understand client-server communication. Clients, whether mobile or web applications, do not communicate directly with each other; instead, they connect to a central chat service.

The chat service must support these fundamental functions:

- Receiving messages from other clients.
- Identifying the correct recipients for each message and relaying the message.
- Holding messages for offline recipients until they come online.
![[Pasted image 20250812105806.png]]
**Figure 12-2: Client and chat service communication**

For the sender side, the **HTTP protocol** is a viable option for sending messages, leveraging HTTP keep-alive to maintain persistent connections and reduce TCP handshakes. Many popular chat applications, like Facebook, initially used HTTP for message sending.

The receiver side is more complex, as HTTP is client-initiated. Techniques to simulate server-initiated connections include polling, long polling, and WebSocket.

#### Polling
![[Pasted image 20250812105844.png]]
**Figure 12-3: Polling** This figure shows a client periodically sending requests to the server to check for new messages, even if no messages are available.

In **polling**, the client regularly queries the server for new messages. This method can be **costly** as it consumes server resources for frequent "no new messages" responses.

#### ==Long polling==
![[Pasted image 20250812105916.png]]
**Figure 12-4: Long polling**

**Long polling** keeps the connection open until new messages are available or a timeout is reached. Once messages are received, the client sends another request. **Drawbacks of long polling include**:

- **Sender and receiver might not connect to the same chat server** in stateless HTTP setups, making message routing difficult.
- **A server cannot easily detect client disconnection**.
- **Inefficiency** for inactive users, as periodic connections are still made after timeouts.

#### WebSocket
![[Pasted image 20250812105939.png]]
**Figure 12-5: WebSocket connection setup**

**WebSocket** is the most common solution for server-to-client asynchronous updates. It is a **bidirectional and persistent connection** initiated by the client, which starts as an HTTP connection and is "upgraded" via a handshake. WebSocket connections typically work through firewalls as they use standard ports (80 or 443). Due to persistent connections, **efficient connection management is crucial on the server-side**.

#### High-level design

While WebSocket is chosen for real-time communication, other chat application features (e.g., sign up, login, user profile) can still use traditional HTTP request/response methods.
![[Pasted image 20250812110129.png]]
**Figure 12-7: High-level system components**

The chat system is broadly divided into:

- **Stateless Services**: These are traditional public-facing request/response services for features like login, signup, and user profiles. They sit behind a load balancer and can be monolithic or microservices. A key stateless service is **service discovery**, which provides clients with chat server DNS host names.
- **Stateful Service**: The **chat service** is stateful because clients maintain persistent network connections to it. Clients usually stick to one chat server as long as it's available, and service discovery works with the chat service to prevent overload.
- **Third-party Integration**: **Push notification** is a critical third-party integration for informing users of new messages even when the app is closed. For more details, refer to "Chapter 10 Design a notification system".


![[Pasted image 20250812110316.png]]
**Figure 12-8: High-level design with various services**

The adjusted high-level design includes:

- Clients maintain persistent **WebSocket connections** to chat servers for real-time messaging.
- **Chat servers** facilitate message sending/receiving.
- **Presence servers** manage online/offline status.
- **API servers** handle user login, signup, profile changes, and other general functionalities.
- **Notification servers** send push notifications.
- A **key-value store** stores chat history, ensuring offline users can access past messages when they come online.

#### Storage

Choosing the right database (relational vs. NoSQL) is critical for the data layer.

- **Generic data** (user profiles, settings, friend lists) are typically stored in robust **relational databases**, using replication and sharding for availability and scalability.
- **Chat history data** has specific patterns:
    - **Enormous volume**: Facebook Messenger and WhatsApp process 60 billion messages daily.
    - **Access patterns**: Only recent chats are frequently accessed, but random access (search, mentions, jump to messages) must be supported.
    - **Read-to-write ratio**: Approximately 1:1 for 1-on-1 chats.

**Key-value stores are recommended for chat history due to**:

- **Easy horizontal scaling**.
- **Very low data access latency**.
- Better handling of **long-tail data** compared to relational databases, where large indexes can make random access expensive.
- Adoption by reliable chat applications like **Facebook Messenger (HBase)** and **Discord (Cassandra)**.

#### Data models
![[Pasted image 20250812110442.png]]
**Figure 12-9: Message table for 1 on 1 chat** 

![[Pasted image 20250812110504.png]]
**Figure 12-10: Message table for group chat** 

**Message ID**: Message IDs are critical for ensuring message order and must be:
- **Unique**.
- **Sortable by time** (newer messages have higher IDs).

**Approaches for Message ID generation**:

- **`auto_increment` in MySQL**: Not suitable for NoSQL databases.
- **Global 64-bit sequence number generator (e.g., Snowflake)**: Discussed in "Chapter 7: Design a unique ID generator in a distributed system".
- **Local sequence number generator**: IDs are unique only within a group. This is sufficient for maintaining message sequence within a 1-on-1 or group channel and is easier to implement.

### Step 3 - Design deep dive

This section delves into service discovery, messaging flows, and online/offline indicators.

#### Service discovery

The main role of **service discovery** is to recommend the best chat server to a client based on factors like geographical location and server capacity. **Apache Zookeeper** is a popular open-source solution for this, registering available chat servers and selecting the optimal one.
![[Pasted image 20250812110654.png]]
**Figure 12-11: Service discovery flow (Zookeeper)** 

**Service discovery workflow (using Zookeeper)**:

1. **User A attempts to log in** to the application.
2. The **load balancer sends the login request to API servers**.
3. After backend authentication, **service discovery (Zookeeper) identifies the most suitable chat server** for User A (e.g., server 2) and returns its information to User A.
4. User A then **establishes a WebSocket connection with the recommended chat server**.

#### Message flows

This section explores 1-on-1 chat flow, message synchronization across multiple devices, and group chat flow.
![[Pasted image 20250812110731.png]]
**1 on 1 chat flow** **Figure 12-12: 1 on 1 chat message flow**

**1 on 1 chat message flow**:

1. **User A sends a chat message to Chat server 1**.
2. **Chat server 1 obtains a `message_id`** from the ID generator.
3. **Chat server 1 sends the message to the message sync queue**.
4. The message is **stored in a key-value store**.
5. **If User B is online**, the message is forwarded to Chat server 2 (where User B is connected).
6. **If User B is offline**, a push notification is sent from push notification (PN) servers.
7. **Chat server 2 forwards the message to User B** via the persistent WebSocket connection.

![[Pasted image 20250812110818.png]]
**Message synchronization across multiple devices** **Figure 12-13: Message synchronization across multiple devices**

Many users have multiple devices, requiring message synchronization.

- Each device maintains a `cur_max_message_id` variable, tracking the latest message ID on that device.
- New messages for a device are those where the recipient ID matches the logged-in user ID, and the message ID in the key-value store is greater than the device's `cur_max_message_id`.
- This approach makes message synchronization straightforward as each device can independently retrieve new messages from the KV store.
![[Pasted image 20250812111005.png]]
**Small group chat flow** **Figure 12-14: Group chat message fanout (sender side)**

When User A sends a message in a group chat, the message is **copied to each group member's message sync queue** (acting as an inbox for each recipient). This design is suitable for small group chats because:

- It **simplifies message synchronization** as each client only needs to check its own inbox.
- For small group sizes, storing a copy in each recipient's inbox is **not too expensive** (e.g., WeChat limits groups to 500 members). However, for very large groups, copying messages for each member becomes unacceptable.
![[Pasted image 20250812111210.png]]
**Figure 12-15: Group chat message flow (recipient side)** This figure shows how a recipient's inbox (message sync queue) aggregates messages from different senders.

On the recipient side, each user has an "inbox" (message sync queue) that receives messages from various senders.

#### Online presence

An online presence indicator (e.g., a green dot) is a key feature of many chat applications. Presence servers are responsible for managing online status and communicating with clients via WebSocket.
![[Pasted image 20250812111229.png]]
**User login** **Figure 12-16: User login flow (simplified)** This figure (though not explicitly drawn in the source, it's mentioned as explained in "Service Discovery") describes the process of a user logging in and establishing a WebSocket connection, leading to their online status being saved in the KV store. After a user logs in and a WebSocket connection is established, their **online status and `last_active_at` timestamp are saved in the key-value store**, indicating they are online.
![[Pasted image 20250812111318.png]]
**User logout** **Figure 12-17: User logout flow (simplified)** This figure (not explicitly drawn, but conceptually referred to) describes the process where a user logs out, their online status is updated to offline in the KV store, and the presence indicator reflects this. When a user logs out, their **online status is updated to offline in the key-value store**.

**User disconnection** Frequent disconnections/reconnections (e.g., in a tunnel) can cause rapid, undesirable changes in the online status indicator. To address this, a **heartbeat mechanism** is used:

- An online client periodically sends a **heartbeat event** to presence servers.
- If presence servers receive a heartbeat within a certain time (e.g., `x` seconds), the user is considered online; otherwise, they are marked offline.
![[Pasted image 20250812111404.png]]
**Figure 12-18: Heartbeat mechanism**
![[Pasted image 20250812111424.png]]
**Online status fanout** **Figure 12-19: Online status fanout** This figure shows how presence servers use a publish-subscribe model, where a user's status change is published to channels subscribed by their friends to update their online status.

To inform friends of status changes, presence servers use a **publish-subscribe model**:

- Each friend pair maintains a **channel**.
- When User A's online status changes, it's **published to relevant channels** (e.g., A-B, A-C, A-D), which are subscribed to by friends (User B, C, D).
- Communication between clients and servers for status updates is through **real-time WebSocket**.

This design is effective for **small user groups** (like WeChat's 500-member limit). For **larger groups** (e.g., 100,000 members), informing all members of every status change is expensive. A solution for large groups is to **fetch online status only when a user enters a group or manually refreshes their friend list**.

**Key Takeaways:**

- **Service discovery (e.g., Zookeeper)** is crucial for selecting the optimal chat server for clients, enhancing load balancing and geographical proximity.
- **Message flows** involve obtaining a unique message ID, queuing the message, storing it in a key-value store, and then delivering it to online recipients via WebSocket or sending push notifications to offline users.
- **Multi-device synchronization** is managed by each device tracking its latest message ID (`cur_max_message_id`) and fetching newer messages from the KV store.
- For **small group chats**, messages are fanned out to individual recipient inboxes to simplify sync, a strategy used by WeChat.
- **Online presence** is managed by presence servers using a heartbeat mechanism to determine online status and a publish-subscribe model to notify friends of status changes. This approach scales well for small groups, but for large groups, status updates might be fetched on demand.

### Step 4 - Wrap up

**Additional talking points for an interview (if time allows)**:

- **Media files**: Extending the chat app to support photos and videos would involve considerations like compression, cloud storage, and thumbnails due to their larger size.
- **End-to-end encryption**: Discussing how to implement E2E encryption, as seen in WhatsApp, where only sender and recipient can read messages.
- **Client-side caching**: Caching messages on the client side can reduce data transfer between client and server.
- **Improve load time**: Strategies like building geographically distributed networks or edge caches (e.g., Slack's Flannel) to cache user data and channels can improve load times.
- **Error handling**:
    - **Chat server error**: If a chat server fails (with potentially hundreds of thousands of persistent connections), service discovery (Zookeeper) provides a new server for clients to reconnect to. Reconnecting all lost clients can be a slow process.
    - **Message resent mechanism**: Techniques like retry and queuing are common for resending messages.

---
### Overall Summary

Chapter 12, "Design a Chat System," provides a comprehensive guide to designing a scalable chat application, focusing on 1-on-1 and small group chat functionalities for **50 million daily active users**. The core objective is to ensure **low delivery latency, reliable message persistence, online presence indication, and multi-device support**.

The chapter begins by establishing design scope through clarifying questions, defining key features and non-functional requirements. It then explores various network protocols for client-server communication, highlighting the benefits of **WebSocket** for its **bidirectional and persistent nature**, making it ideal for real-time messaging, unlike polling or long polling.

The high-level architecture proposes a distributed system consisting of **stateless services** (for login, user profiles, and service discovery), a **stateful chat service** (managing persistent WebSocket connections), and **third-party integrations** (like push notification services). For data storage, **key-value stores** are recommended for chat history due to their superior horizontal scalability and low latency for massive, frequently accessed data, while traditional relational databases handle generic user data. The chapter details message data models for both 1-on-1 and group chats, emphasizing the importance of **unique and time-sortable message IDs**.

The deep dive section refines key components:

- **Service discovery**, using tools like **Apache Zookeeper**, efficiently routes clients to optimal chat servers based on factors like geographic location and server capacity.
- **Message flows** are meticulously detailed, explaining how messages are processed, queued, persisted in the key-value store, and delivered to online or offline recipients, including a strategy for **multi-device synchronization** using a `cur_max_message_id` on each device. For small group chats, a fan-out on write model is adopted where messages are copied to each recipient's "inbox".
- **Online presence** is managed by dedicated presence servers utilizing a **heartbeat mechanism** to detect user activity and a **publish-subscribe model** to fan out status updates to friends via WebSocket connections, with considerations for scaling to larger groups.

Finally, the chapter concludes by summarizing the design and suggesting additional areas for discussion, such as handling media files, implementing end-to-end encryption, client-side caching, and robust error handling mechanisms for various system components. The overall design emphasizes **scalability, reliability, and low latency** crucial for a successful chat system.