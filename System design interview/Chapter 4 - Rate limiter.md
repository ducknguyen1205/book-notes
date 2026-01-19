
# Summary of Chapter 4: Design a Rate Limiter

This chapter focuses on designing a **rate limiter** for a network system. A rate limiter is used to **control the rate of traffic** sent by a client or service. In the context of HTTP, it specifically limits the number of client requests over a defined period. If requests exceed the set threshold, they are blocked.

Examples of rate limiting rules include limiting posts per second, account creations per day from an IP address, or reward claims per week from a device.

## Benefits of Using an API Rate Limiter

Using an API rate limiter provides several key benefits:

- **Prevent resource starvation** caused by Denial of Service (DoS) attacks. Many large tech companies implement rate limiting to prevent such attacks.
- **Reduce cost** by limiting excess requests, allowing for fewer servers and prioritizing resources for high-priority APIs. This is crucial when using paid third-party APIs charged on a per-call basis.
- **Prevent servers from being overloaded** by filtering out excess requests from bots or user misbehavior.

## Step 1 - Understand the Problem and Establish Design Scope

System design interviews often start with open-ended questions, and clarifying the requirements is critical. For designing a rate limiter, clarification questions include:

- Whether it's a client-side or server-side API rate limiter (focus is on server-side).
- If throttling is based on IP, user ID, or other properties (should be flexible).
- The scale of the system (must handle a large number of requests).
- If it works in a distributed environment (yes).
- Whether it's a separate service or implemented in application code (design decision).
- If users should be informed when throttled (yes).

Based on these questions, the requirements for the system are summarized as:

- Accurately limit excessive requests.
- **Low latency**; it should not slow down HTTP response time.
- Use as little memory as possible.
- **Distributed rate limiting**, shareable across multiple servers or processes.
- **Exception handling** to show clear exceptions to throttled users.
- **High fault tolerance** so that rate limiter issues don't affect the entire system.

## Step 2 - Propose High-Level Design and Get Buy-In

Starting with a simple client and server model, the question arises: **Where to put the rate limiter?**.

- **Client-side implementation:** Generally unreliable as requests can be forged, and client implementation may not be controlled.
- **Server-side implementation:** Place the rate limiter on the server-side. This is shown in **Figure 4-1**.
![[Pasted image 20250516152535.png]]
- **Middleware implementation:** An alternative is a rate limiter middleware placed before API servers. This middleware throttles requests to APIs. **Figure 4-2** illustrates this, and **Figure 4-3** provides an example where the middleware throttles a third request when the limit is 2 requests per second, returning a HTTP status code 429. This HTTP 429 response code signifies "too many requests". API gateways are common middleware components that include rate limiting functionality, especially in microservices architecture.
![[Pasted image 20250516152546.png]]

Choosing where to implement depends on factors like technology stack, required algorithm control, use of an existing API gateway, and available engineering resources.

## Algorithms for Rate Limiting

Various algorithms exist, each with pros and cons. Popular algorithms include:

- Token bucket
- Leaking bucket
- Fixed window counter
- Sliding window log
- Sliding window counter

### Token Bucket Algorithm

This is a widely used and well-understood algorithm. It works by maintaining a bucket holding tokens. Requests consume tokens; if enough tokens are available, a token is taken, and the request proceeds. If not, the request is dropped. **Figure 4-5** explains this, and **Figure 4-6** shows token consumption, refill, and rate limiting with a bucket size of 4 and refill rate of 4 per minute. It takes two parameters: **bucket size** (max tokens) and **refill rate** (tokens added per second). The number of buckets needed depends on the rate-limiting rules, such as different buckets for API endpoints, IP addresses, or a global bucket.
![[Pasted image 20250516152646.png]]
![[Pasted image 20250516152657.png]]
![[Pasted image 20250516152805.png]]
### Leaking Bucket Algorithm

This algorithm is implemented as a FIFO queue. Requests are added to the queue if not full and dropped otherwise. Requests are processed from the queue at a fixed rate. **Figure 4-7** explains its operation. It takes two parameters: **bucket size** (queue size) and **outflow rate** (requests processed per fixed rate). Shopify uses this algorithm.
![[Pasted image 20250516152900.png]]
- Pros: Memory efficient (limited queue size), suitable for stable outflow rate needs.
- Cons: A traffic burst can fill the queue with old requests, causing recent ones to be rate-limited; tuning parameters might be difficult.

### Fixed Window Counter Algorithm

This algorithm divides time into fixed windows and assigns a counter to each. Requests increment the counter. Once the counter reaches the threshold, new requests are dropped until the next window starts. **Figure 4-8** illustrates this with a limit of 3 requests per second.
![[Pasted image 20250516152930.png]]
- Pros: Memory efficient, easy to understand, fits use cases needing quota reset at the end of a window.
- Cons: Allows more requests than the quota at the edges of a window due to traffic spikes.

### Sliding Window Log Algorithm

This algorithm addresses the fixed window counter's edge case issue. It tracks request timestamps, often in cache using sorted sets (like Redis). **When a request arrives, outdated timestamps (older than the current window start) are removed.** The new request's timestamp is added. If the log size is within the allowed count, the request is accepted; otherwise, it's rejected. **Figure 4-10** shows an example with a limit of 2 requests per minute.
![[Pasted image 20250516153114.png]]
- Pros: **Very accurate** rate limiting, ensures requests don't exceed the limit in any rolling window.
- Cons: Consumes a **lot of memory** as timestamps for rejected requests may still be stored.

### Sliding Window Counter Algorithm

This is a hybrid approach combining fixed window counter and sliding window log. One implementation calculates the number of requests in a rolling window based on requests in the current window plus a fraction of requests from the previous window, weighted by the overlap percentage. ==ASK AI TO EXPLAIN==

- Pros: Smooths out traffic spikes, memory efficient.
- Cons: An approximation, only works for non-strict look-back windows as it assumes even distribution of requests in the previous window. However, this approximation is often acceptable in practice (e.g., Cloudflare experiments showed a very low error rate).

## High-Level Architecture (with Redis)

The basic idea involves using a counter. If the counter exceeds the limit, the request is disallowed. Storing counters in a database is slow due to disk access. **In-memory cache**, like **Redis**, is preferred due to its speed and expiration strategy. Redis provides `INCR` (increment counter) and `EXPIRE` (set timeout for deletion) commands.

The high-level architecture is shown in **Figure 4-12**. The workflow is:

1. Client sends a request to the rate limiting middleware.
2. Middleware fetches the counter from Redis and checks the limit.
3. If the limit is reached, the request is rejected.
4. If not, the request is sent to API servers. Simultaneously, the system increments the counter in Redis.

## Step 3 - Design Deep Dive

Key questions in the deep dive include how rules are created/stored and how rate-limited requests are handled.

### Rate Limiting Rules

Rules define the throttle behavior. Examples (inspired by Lyft) show rules specifying domain, descriptors (like message type or auth type), unit (day, minute), and requests per unit. Rules are typically stored in **configuration files** on disk.

### Exceeding the Rate Limit

When a request is rate limited, APIs return an **HTTP 429 (Too Many Requests) error**. Depending on the use case, rate-limited requests might be enqueued for later processing (e.g., for orders). Clients can be informed about throttling and remaining quota using **HTTP response headers**:

- `X-Ratelimit-Remaining`: Number of allowed requests left in the window.
- `X-Ratelimit-Limit`: Total calls allowed per time window.
- `X-Ratelimit-Retry-After`: Seconds to wait before retrying without throttling. When a 429 error occurs, the `X-Ratelimit-Retry-After` header is typically returned.

### Detailed Design
![[Pasted image 20250516154324.png]]
**Figure 4-13** shows a detailed design.

- Rules are stored on disk and pulled into cache by workers.
- Client requests go to the rate limiter middleware.
- Middleware loads rules from cache and fetches counters/timestamps from Redis.
- Based on the check, the request is either forwarded to API servers or rejected with a 429 error. Rejected requests might be dropped or sent to a queue.

### Rate Limiter in a Distributed Environment

Scaling to multiple servers and concurrent threads introduces challenges:

- **Race condition:** Concurrent reads and writes to the counter in Redis can lead to incorrect counts. **Figure 4-14** illustrates this where two requests reading a counter value of 3 might both write back 4 instead of the correct 5.
![[Pasted image 20250516154620.png]]
- **Synchronization issue:** Clients might send requests to different rate limiters if the web tier is stateless. Without synchronization, rate limiters lack complete data for a client. **Figure 4-15** shows this scenario. Sticky sessions could solve this but are not scalable. A **centralized data store like Redis** is a better approach. **Figure 4-16** shows a design with a centralized Redis.
![[Pasted image 20250516154651.png]]

![[Pasted image 20250516154705.png]]
### Performance Optimization

Optimizations include:

- **Multi-data center setup:** Crucial due to high latency over distance. Edge servers reduce latency by serving traffic from locations closer to users.
- **Eventual consistency:** Synchronizing data across data centers can use an eventual consistency model.

### Monitoring

Monitoring is essential to check the rate limiter's effectiveness. Key aspects to monitor are:

- Effectiveness of the rate limiting algorithm.
- Effectiveness of the rate limiting rules. If rules are too strict, valid requests are dropped; if ineffective during traffic spikes, algorithms might need replacement (e.g., using Token bucket for bursts).

## Step 4 - Wrap up

The chapter summarized the discussed rate-limiting algorithms:

- Token bucket
- Leaking bucket
- Fixed window
- Sliding window log
- Sliding window counter

It also covered system architecture, distributed concerns, performance, and monitoring. Additional discussion points if time permits include:

- **Hard vs soft rate limiting:** Hard means requests cannot exceed the threshold; soft allows exceeding for a short period.
- **Rate limiting at different levels:** Beyond the application layer (Layer 7 HTTP), it can be applied at lower layers like IP (Layer 3) using tools like Iptables. The OSI model layers are mentioned for context.
- **Avoiding being rate limited (client-side best practices):** Use client cache, understand limits, implement exception handling for graceful recovery, and add sufficient back-off time to retry logic.

# Reference materials
[1] Rate-limiting strategies and techniques: https://cloud.google.com/solutions/rate-limitingstrategies-techniques 
[2] Twitter rate limits: https://developer.twitter.com/en/docs/basics/rate-limits 
[3] Google docs usage limits: https://developers.google.com/docs/api/limits 
[4] IBM microservices: https://www.ibm.com/cloud/learn/microservices 
[5] Throttle API requests for better throughput: https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-requestthrottling.html 
[6] Stripe rate limiters: https://stripe.com/blog/rate-limiters 
[7] Shopify REST Admin API rate limits: https://help.shopify.com/en/api/reference/restadmin-api-rate-limits [8] Better Rate Limiting With Redis Sorted Sets: https://engineering.classdojo.com/blog/2015/02/06/rolling-rate-limiter/ 
[9] System Design — Rate limiter and Data modelling: https://medium.com/@saisandeepmopuri/system-design-rate-limiter-and-data-modelling9304b0d18250 
[10] How we built rate limiting capable of scaling to millions of domains: https://blog.cloudflare.com/counting-things-a-lot-of-different-things/ 
[11] Redis website: https://redis.io/ 
[12] Lyft rate limiting: https://github.com/lyft/ratelimit 
[13] Scaling your API with rate limiters: https://gist.github.com/ptarjan/e38f45f2dfe601419ca3af937fff574d#request-rate-limiter 
[14] What is edge computing: https://www.cloudflare.com/learning/serverless/glossary/whatis-edge-computing/ 
[15] Rate Limit Requests with Iptables: https://blog.programster.org/rate-limit-requests-withiptables
[16] OSI model: https://en.wikipedia.org/wiki/OSI_model#Layer_arc