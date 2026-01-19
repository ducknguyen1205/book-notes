This chapter focuses on providing a **reliable strategy to approach system design questions** in interviews. System design interview questions are often considered the most difficult technical interview questions because they are typically **broad, vague, and open-ended**, without a single correct answer. The objective of this chapter is to equip readers with a **step-by-step framework** to tackle these questions.

The chapter presents a **4-step process for effective system design interviews**. While every interview is different and there's no one-size-fits-all solution, these steps provide common ground to cover.

Here is a breakdown of the four steps:

**Step 1 - Understand the problem and establish design scope** The first and critical step is to **understand the requirements and clarify ambiguities** by asking questions. System design questions are often intentionally left open-ended. This step involves acting like a curious child, like "Jimmy" in the provided analogy, who is eager to answer questions, but in this context, it's about asking clarifying questions.

The example provided uses "Design a news feed system" to illustrate this step. A candidate asks the interviewer clarifying questions such as:

- Is the system a mobile app, web app, or both?
- What are the most important features (e.g., make a post, see friends' news feed)?
- How is the news feed sorted (e.g., reverse chronological order or weighted by importance)? For simplicity, the example assumes sorting by reverse chronological order.
- What is the maximum number of friends a user can have (e.g., 5000)?
- What is the traffic volume (e.g., 10 million daily active users (DAU))?
- Can the feed contain images, videos, or just text? In the example, it can contain media files.

Asking these questions helps in understanding the scope and refining the problem.

**Step 2 - Propose high-level design and get buy-in** In this step, the goal is to **develop a high-level design and reach an agreement** with the interviewer. It is recommended to **collaborate with the interviewer** during this process.

Going through a few **concrete use cases** can help frame the high-level design and discover potential edge cases. Whether to include API endpoints and database schema depends on the problem's scale; for large problems like designing Google Search, it might be too low-level, but for smaller ones, it could be appropriate. Communication with the interviewer is key here.

Using the "Design a news feed system" example, the high-level design is divided into two flows:

- **Feed publishing**: When a user publishes a post, data is written to cache/database, and the post is populated into friends' news feeds.
- **News feed building**: The news feed is constructed by aggregating friends' posts, in this simplified case, in reverse chronological order.
![[Pasted image 20250519104721.png]]
**Figure 3-1** is mentioned as presenting the high-level design for the **feed publishing flow**. **Figure 3-2** is mentioned as presenting the high-level design for the **news feed building flow**. (Note: The details of these figures are not provided in the text excerpts).
![[Pasted image 20250519104753.png]]
==The recommended time allocation for this step is **10 - 15 minutes**.==

**Step 3 - Design deep dive** By this step, the candidate and interviewer should have agreed on the goals, feature scope, sketched a high-level design, received feedback, and have initial ideas on areas for deep dive.

**Time management is essential** in this step to avoid getting lost in minute details that don't effectively demonstrate abilities. The focus should be on demonstrating the ability to design a scalable system, not on explaining highly specific or complex algorithms like Facebook's EdgeRank in detail.

Continuing the "Design a news feed system" example, the deep dive investigates two important use cases:

1. Feed publishing
2. News feed retrieval

**Figure 3-3** shows the detailed design for the **feed publishing** use case, and **Figure 3-4** shows the detailed design for the **news feed retrieval** use case. (Note: The details of these figures are not provided in the text excerpts, but the text mentions they will be explained in detail in Chapter 11).
![[Pasted image 20250519104841.png]]

==The recommended time allocation for this step is **10 - 25 minutes**.==

**Step 4 - Wrap up** This is the final step, where the interviewer might ask follow-up questions or allow the candidate to discuss additional points.

Several directions can be followed in this step:

- **Identify system bottlenecks and discuss potential improvements**. It's crucial to acknowledge that no design is perfect and there's always room for improvement. This demonstrates critical thinking.
- **Recap the design**, especially if multiple solutions were proposed, to refresh the interviewer's memory.
- Discuss **error cases** such as server failure or network loss.
- Mention **operation issues**, such as how metrics and error logs are monitored or how the system is rolled out.
- Discuss **scaling** the system for the "next scale curve" (e.g., scaling from 1 million to 10 million users).
- Propose **other refinements** that could be made with more time.

The recommended time allocation for this step is **3 - 5 minutes**.

Chapter 3 provides a table summarizing the recommended time allocation for each step: **Table 3-1** (implicitly referenced by the time allocations listed in):

- Step 1 Understand the problem and establish design scope: **10 - 15 minutes**
- Step 2 Propose high-level design and get buy-in: **10 - 15 minutes**
- Step 3 Design deep dive: **10 - 25 minutes**
- Step 4 Wrap: **3 - 5 minutes**

Following this framework is vital for success in the interview. Consistent practice with detailed steps and examples is recommended.