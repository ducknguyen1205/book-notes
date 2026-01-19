
## CHAPTER 10: DESIGN A NOTIFICATION SYSTEM

Notifications can come in various formats: mobile push notifications, SMS messages, and Emails.

### Figure 10-1: Example Notifications

- **Description:** This figure visually illustrates the three types of notification formats discussed in the chapter:
    - **Mobile Push Notification:** Shows a typical phone screen with a notification banner, possibly from an app, displaying a short message and an icon.
    - **SMS Message:** Depicts a text message conversation, showing a short, plain text message from a sender.
    - **Email:** Presents a screenshot of an email client showing an email with a subject line, sender, and a body of text, possibly including images or links.
- **Role in Chapter:** This figure introduces the diverse forms of notifications that the system needs to support, setting the stage for the design discussion.

---

### Step 1 - Understand the problem and establish design scope

Designing a scalable system capable of sending millions of notifications daily is complex and requires a deep understanding of the notification ecosystem. System design interview questions are intentionally open-ended, making it crucial to ask clarifying questions to establish requirements.

**Candidate-Interviewer Interaction Example:**

- **Candidate asks:** What types of notifications are supported?
- **Interviewer responds:** Push notification, SMS message, and email.
- **Candidate asks:** Is it a real-time system?
- **Interviewer responds:** A soft real-time system is desired, meaning notifications should be received as soon as possible, but slight delays are acceptable under high workload.
- **Candidate asks:** What are the supported devices?
- **Interviewer responds:** iOS devices, Android devices, and laptop/desktop.
- **Candidate asks:** What triggers notifications?
- **Interviewer responds:** Notifications can be triggered by client applications or scheduled on the server-side.
- **Candidate asks:** Will users be able to opt-out?
- **Interviewer responds:** Yes, opted-out users should no longer receive notifications.
- **Candidate asks:** How many notifications are sent each day?
- **Interviewer responds:** 10 million mobile push notifications, 1 million SMS messages, and 5 million emails.

**Requirements Summary:**

- Support for **push notifications, SMS messages, and emails**.
- **Soft real-time** delivery.
- Support for **iOS, Android, and web/desktop** clients.
- Notifications triggered by **client applications or server-side scheduling**.
- **User opt-out** functionality.
- Daily volume: **10M mobile push, 1M SMS, 5M emails**.

---

### Step 2 - Propose high-level design and get buy-in

This section outlines a high-level design supporting various notification types and focuses on contact information gathering, and notification sending/receiving flows.

#### Different types of notifications

The design starts by understanding how each notification type works at a high level.

- **iOS push notification:** Requires three main components:
    
    - **Provider:** Builds and sends notification requests.
    - **APNS (Apple Push Notification Service):** A remote service by Apple for propagating push notifications to iOS devices.
    - **iOS Device:** The end client receiving notifications.
    - **Payload:** A JSON dictionary containing notification data, including a `device token` (unique identifier) and the actual notification `payload` (e.g., `{"aps":{"alert":"Hello world!"}}`).
    - ![[Pasted image 20250804163824.png]]
- **Android push notification:** Similar flow, but uses **Firebase Cloud Messaging (FCM)** instead of APNS.
    
- **SMS message:** Typically uses **third-party SMS services** like Twilio or Nexmo. These are usually commercial services.
    
- **Email:** Many companies use **commercial email services** like Sendgrid or Mailchimp for better delivery rates and analytics, rather than setting up their own servers.
![[Pasted image 20250804163850.png]]
### Figure 10-6: Design after including third-party services

- **Description:** This figure is a high-level block diagram showing the interaction between the "Notification System" and various "Third-party Services," which then deliver notifications to "iOS Devices," "Android Devices," "SMS," and "Email." The "Notification System" is at the center, with arrows pointing towards the third-party services.

#### Contact info gathering flow

To send notifications, the system must gather mobile device tokens, phone numbers, or email addresses.
![[Pasted image 20250804164015.png]]
### Figure 10-7: Contact Info Gathering Flow
![[Pasted image 20250804164035.png]]
### Figure 10-8: Database tables for contact info

- **Description:** This figure displays two simplified database tables: `user` and `device`.
    - **Table 10-1: User Table**
        - **Title:** User Table
        - **Headers:** `user_id`, `name`, `email`, `phone_number`
        - **Role in Chapter:** Shows how basic user information, including email addresses and phone numbers, is stored in the `user` table.
    - **Table 10-2: Device Table**
        - **Title:** Device Table
        - **Headers:** `device_id`, `user_id`, `device_token`
        - **Role in Chapter:** Illustrates how device tokens, crucial for push notifications, are stored in the `device` table, allowing for multiple devices per user.

#### Notification sending/receiving flow

The chapter first presents an initial design, then proposes optimizations.

**High-level design (Initial):**
![[Pasted image 20250804164058.png]]
### Figure 10-9: Initial high-level design

**Components of the initial design:**

- **Service 1 to N:** Represents various microservices, cron jobs, or distributed systems that trigger notification sending events (e.g., a billing service, a shopping website).
- **Notification system:** The core, initially a single server, providing APIs to other services and building notification payloads for third-party services.
- **Third-party services:** External services responsible for delivering notifications (APNS, FCM, Twilio, Sendgrid). Extensibility is important here due to potential unavailability in new markets (e.g., FCM in China).
- **iOS, Android, SMS, Email:** The end-user devices or channels receiving the notifications.

**Problems identified in the initial design:**

- **Single point of failure (SPOF):** A single notification server means if it goes down, the entire system is affected.
- **Hard to scale:** Combining database, cache, and all notification processing on one server makes independent scaling challenging.
- **Performance bottleneck:** Resource-intensive tasks (e.g., HTML page construction, waiting for third-party responses) in a single system can lead to overload, especially during peak hours.

**High-level design (Improved):** To address the challenges, the improved design incorporates several optimizations:

- Database and cache are moved out of the notification server.
- More notification servers are added with automatic horizontal scaling.
- Message queues are introduced to decouple system components.
![[Pasted image 20250804164219.png]]
### Figure 10-10: Improved high-level design

**Components of the improved design:**

- **Service 1 to N:** Same as before, services triggering notifications.
- **Notification servers:** Now a pool of servers (behind a load balancer) that provide internal APIs, perform basic validations, query cache/database for data, and put notification data into message queues.
    - **Example API to send an email:**
        
        ```
        POST https://api.example.com/v/sms/send
        Request body
        {
          "to": "+1650XXXXXXX",
          "from": "+1222XXXXXXX",
          "message": "Hello world!",
          "user_id": "xxxx"
        }
        ```
        
        - **`POST https://api.example.com/v/sms/send`**: This is the HTTP method and endpoint for sending an SMS message. `POST` indicates data is being sent to create or update a resource.
        - **`"to": "+1650XXXXXXX"`**: The recipient's phone number.
        - **`"from": "+1222XXXXXXX"`**: The sender's phone number.
        - **`"message": "Hello world!"`**: The actual text content of the SMS.
        - **`"user_id": "xxxx"`**: An identifier for the user initiating the SMS.
        - **Role in Chapter:** This JSON snippet demonstrates a typical API request body for sending an email (though the example endpoint uses `/sms/send`), illustrating the kind of data passed to the notification servers.
- **Cache:** Stores frequently accessed data like user info, device info, and notification templates for fast retrieval.
- **DB:** Stores persistent data related to users, notifications, and settings.
- **Message queues:** Decouple components and act as buffers during high notification volumes. Each notification type (iOS PN, Android PN, SMS, Email) gets a distinct queue to prevent outages in one third-party service from affecting others.
- **Workers:** Servers that pull notification events from message queues and send them to the corresponding third-party services.
- **Third-party services, iOS, Android, SMS, Email:** Remain the same as in the initial design, responsible for final delivery.

**Workflow of notification sending:**

1. A service calls APIs from notification servers to send notifications.
2. Notification servers fetch metadata (user info, device token, settings) from cache or DB.
3. A notification event is sent to the corresponding message queue (e.g., iOS PN queue).
4. Workers pull notification events from the message queues.
5. Workers send notifications to third-party services.
6. Third-party services deliver notifications to user devices.

**Key Takeaways for Step 2:**

- **Leverage Third-Party Services:** For specific notification types (push, SMS, email), it's efficient to integrate with established third-party providers (APNS, FCM, Twilio, Sendgrid).
- **Contact Info Collection:** A fundamental step is to securely collect and store user contact details (device tokens, phone numbers, emails) in a database.
- **Decoupling with Message Queues:** Using message queues is crucial for handling high volumes, providing buffering, and isolating failures across different notification types.
- **Horizontal Scaling:** Move state (DB, Cache) out of application servers to enable easy horizontal scaling of notification servers and workers.
- **Initial Design Flaws:** A monolithic notification server is prone to SPOF, difficult to scale, and can become a performance bottleneck.

---

### Step 3 - Design deep dive

This section delves into reliability, additional components, and an updated system design.

#### Reliability

Two important reliability questions are addressed:

- **How to prevent data loss?** Notifications can be delayed or re-ordered, but not lost. To ensure this, notification data is persisted in a database, and a retry mechanism is implemented. A **notification log database** is used for data persistence.
![[Pasted image 20250804164901.png]]
### Figure 10-11: Data persistence with Notification Log DB

- **Description:** This diagram extends the previous design by specifically showing "Notification Log DB" as a persistent storage component that "Notification Servers" write to _before_ putting events into "Message Queues." This ensures that even if the queues or workers fail, the original notification data is preserved.
- ==**Will recipients receive a notification exactly once?**== The answer is generally no, due to the distributed nature which can result in duplicate notifications. ==A **dedupe mechanism** is suggested: when an event arrives, check its ID; if seen before, discard it, otherwise send.==

#### Additional components and considerations

Beyond basic sending/receiving, a comprehensive notification system includes:

- **Notification template:** Reusing preformatted notifications saves time, reduces errors, and maintains consistent formatting.
    
    - **Example template of push notifications:**
```
BODY: You dreamed of it. We dared it. [ITEM NAME] is back — only until [DATE].
CTA: Order Now. Or, Save My [ITEM NAME]
```

- **Notification setting:** Allows users fine-grained control over which notifications they receive (opt-in/opt-out). This information is stored in a `notification_setting` table.
    
    - **Simplified `notification_setting` table fields:**
```sql 
user_id   bigInt
channel   varchar    # push notification, email or SMS
opt_in    boolean    # opt-in to receive notification
```
- **`user_id bigInt`**: The unique identifier for the user.
- **`channel varchar`**: The type of notification channel (e.g., "push notification", "email", "SMS").
- **`opt_in boolean`**: A flag indicating whether the user has opted in (true) or out (false) for this specific channel.

- **Rate limiting:** Prevents overwhelming users with too many notifications, which could lead them to turn off notifications entirely.
    
- **Retry mechanism:** If a third-party service fails to send a notification, it is re-added to the message queue for retrying. Persistent failures trigger alerts to developers.
    
- **Security in push notifications:** Uses `appKey` and `appSecret` pairs (common in iOS/Android apps) to ensure only authenticated/verified clients can send notifications via APIs.
    
- **Monitor queued notifications:** A key metric to track is the total number of queued notifications. A large number indicates workers aren't processing fast enough, requiring more workers to avoid delivery delays.
![[Pasted image 20250804170004.png]]
### Figure 10-12: Queued messages to be processed
    
- **Events tracking:** Collecting metrics like open rate, click rate, and engagement helps understand customer behaviors and is implemented via integration with an analytics service.
![[Pasted image 20250804170021.png]]
### Figure 10-13: Sample events to be tracked

- **Description:** This figure shows a list of sample events that might be tracked for analytics purposes within a notification system:
    - **Notification Sent:** Indicates a notification was dispatched.
    - **Notification Opened:** Records when a user opened a notification.
    - **Click-Through Rate:** Measures the percentage of users who clicked on a notification.
    - **User Opt-Out:** Logs when a user opts out of notifications.
    - **Error Rate:** Tracks the frequency of notification delivery errors.

#### Updated design
![[Pasted image 20250804170103.png]]
### Figure 10-14: Updated notification system design

**New components and features in the updated design:**

- **Notification servers:** Now include **authentication** and **rate-limiting**.
- **Retry mechanism:** Notifications that fail to send are put back into the message queue for retries by workers.
- **Notification templates:** Provide a consistent and efficient creation process.
- **Monitoring and tracking systems:** Added for system health checks and future improvements.

**Key Takeaways for Step 3:**

- **Data Persistence:** A notification log database is crucial to prevent data loss, even if message queues or workers fail.
- **Deduplication:** While "exactly once" delivery is hard, a dedupe mechanism helps reduce duplicate notifications by checking event IDs.
- **Templating:** Notification templates enhance consistency, reduce errors, and save time in generating high volumes of notifications.
- **User Control:** Implement fine-grained user settings (opt-in/out) to respect user preferences and avoid overwhelming them.
- **System Health:** Monitoring queued notifications is vital to identify bottlenecks and scale workers as needed.
- **Security:** Use `appKey` and `appSecret` for API security.
- **Comprehensive Design:** A robust system incorporates authentication, rate limiting, retry logic, templates, user settings, monitoring, and tracking.

---

### Step 4 - Wrap up

Notifications are essential for keeping users informed. This chapter detailed the design of a scalable notification system supporting push notifications, SMS, and email, utilizing message queues for decoupling.

**Summary of algorithms (though specifically rate limiting algorithms are mentioned, the chapter's focus is on overall system design):**
- Token bucket
- Leaking bucket
- Fixed window
- Sliding window log
- Sliding window counter _(Note: These rate-limiting algorithms are detailed in Chapter 4, but mentioned here as potential components within the notification system's overall design, particularly for the rate limiter. The chapter itself focuses on the notification system's architecture rather than deep-diving into these algorithms within its own text.)_

**Additional talking points for discussion (if time allows in an interview):**

- **Hard vs soft rate limiting:**
    - **Hard:** Request count cannot exceed threshold.
    - **Soft:** Requests can temporarily exceed the threshold.
- **Rate limiting at different levels:** Beyond application level (HTTP Layer 7), it can be applied at other layers (e.g., IP addresses using Iptables at IP Layer 3).
- **Avoid being rate limited (client best practices):**
    - Use **client cache** to reduce frequent API calls.
    - Understand and respect limits, avoid sending too many requests quickly.
    - Include **exception handling** and **retry logic with sufficient back off time** for graceful recovery.

**Key Takeaways for Step 4:**

- **Iterative Design:** System design is not about a perfect solution, but understanding tradeoffs and constraints.
- **Scalability & Reliability:** Core principles like reliability, security, tracking, monitoring, user settings, and rate limiting are crucial.
- **Advanced Considerations:** Discussions on different types of rate limiting (hard vs soft, different layers) and client-side best practices can demonstrate deeper understanding.

---

## Overall Summary of Chapter 10's Objectives and Components:

The primary **objective** of Chapter 10 is to provide a comprehensive architectural design for a **scalable, reliable, and high-volume notification system** that can deliver messages across multiple channels, including **mobile push notifications, SMS, and Email**. The system aims for soft real-time delivery and supports user opt-out functionalities.

The chapter begins by clarifying requirements and understanding the diverse nature of notification types, highlighting the reliance on **third-party services** (like APNS, FCM, Twilio, Sendgrid) for actual message delivery. It emphasizes the critical need for a **contact info gathering flow** to collect necessary user data (device tokens, phone numbers, emails) and store it in a structured database.

The design evolves from a simplistic, single-server approach (prone to single points of failure, scalability issues, and performance bottlenecks) to a **distributed, highly available architecture**. Key to this improved design is the **decoupling of components using message queues** for each notification type, acting as buffers and ensuring resilience. **Notification Servers** handle API requests, validation, and data fetching from **Cache** and **Database**, then enqueue events. **Workers** consume from these queues and interface with the third-party services.

For **reliability**, the system incorporates a **Notification Log Database** to persist all notification data, preventing loss, and implements a **retry mechanism** for failed deliveries. While "exactly once" delivery is difficult, a **deduplication mechanism** helps mitigate duplicate notifications.

Further **enhancements** include using **notification templates** for efficient and consistent message creation, respecting **user notification settings** (opt-in/out), implementing **rate limiting** to prevent user overload, ensuring **security** for API access (e.g., appKey/appSecret), and robust **monitoring and event tracking** to measure system health and user engagement. The final design diagram brings all these components together, showcasing a sophisticated and production-ready notification system.