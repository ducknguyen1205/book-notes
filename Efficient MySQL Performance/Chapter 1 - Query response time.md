#efficient-mysql-performance #book 


- **Core Definition of Performance**:
    
    - **Performance is query response time**. This is defined as the duration MySQL takes to execute a query, starting when the query is received and ending when the result set is sent to the client.
    - Synonymous terms for query response time include response time, query time, execution time, and (inaccurately) query latency.
    - ==Improving performance means **reducing query response time**.==
---
- **The "North Star" of MySQL Performance**:
    
    - ==**Query response time is the North Star of MySQL performance**.==
    - It's considered **meaningful** because it's the only metric users directly experience.
    - It's **actionable** because engineers can directly optimize queries or indirectly affect performance.
    - The approach to improving performance should begin by **using query metrics to understand MySQL's activity**, followed by **analyzing and optimizing slow queries**.
    - The author advises **not to begin by "throwing hardware at the problem"**.
---
- **Query Reporting**: The process of transforming raw query metrics into meaningful reports. This is a means to achieve query analysis.
    
    - **Sources of Query Metrics**:
        - **Slow query log**: Historically, it logged queries longer than `N` seconds (minimum `N` was 1). Today, it can log every query if `long_query_time` is set to zero, with microsecond resolution. On busy servers, logging every query can increase disk I/O and consume significant disk space. Percona Server's slow query log offers more metrics and query sampling.
        - **Performance Schema**: Considered the **best source of query metrics** because it's available in all current MySQL versions, works in various environments, provides all essential query metrics, and offers a wealth of data for deeper analysis.
    - **Aggregation**: Query metrics are **grouped and aggregated by query**.
        - Queries are uniquely identified by a **SHA-256 hash of the normalized SQL statement**, known as a **digest hash**.
        - **Normalization** involves replacing values with `?` and collapsing multiple whitespaces. The normalized form is called **digest text**.
        - The term **"query" becomes synonymous with "digest text"** in the context of grouping metrics.
        - Terminology for these concepts can vary among query metric tools (e.g., class, family, ==fingerprint (see image below)==, query ID, signature).


![[Pasted image 20250702103201.png]]
        - A **query abstract** is a highly abstracted SQL statement (e.g., `SELECT tbl`), which is succinct but not unique.
        - Dynamically generating the same logical query with different syntax can lead to them being reported as separate queries due to different digests. However, `IN` clauses with different values can normalize to the same digest.
    - **Reporting Structure**: Typically presented in a **two-level hierarchy**.
        - **Query Profile**: A high-level overview showing slow queries, usually sorted by **total query time** (the sum of execution time per query). This helps identify which query consumes the most MySQL time.
        - **Query Load**: An aggregate value (total query time per query divided by clock time) that can indicate query concurrency. A load greater than 1.0 suggests concurrency.
        - **Query Report**: A detailed view for a single query, presenting all its metrics74]. Ideally small, but problematic if over 50% of query time. **Performance Schema lock time specifically does NOT include row lock waits**, unlike the slow query log. Locks are primarily for writes and released on transaction commit/rollback.
        - **Rows examined**: The number of rows MySQL accessed to find matches. High values can indicate poor selectivity of queries or indexes.
        - **Rows sent**: The number of rows returned to the client. Comparison with "rows examined" provides insight into query/index selectivity and potential table scans.
        - **Rows affected**: The number of rows inserted, updated, or deleted. Useful for understanding the impact of bulk operations.
        - **Select scan**: The number of **full table scans on the first table accessed**. This is generally bad and suggests the query isn't using an index. Optimization is strongly advised.
        - **Select full join**: The number of **full table scans on joined tables**. Worse than `select scan` and practically requires optimization if non-zero.
        - **Created tmp disk tables**: The number of **temporary tables created on disk**. This indicates that in-memory temporary tables grew too large, affecting performance due to slower disk access.
        - **Query count**: The total number of times a query was executed.
    - **Metadata and the Application**:
        - **EXPLAIN plan (query execution plan)**: An indispensable tool for understanding MySQL's planned query execution, including join order, access method, and index usage.
        - **Table structures**: Obtained using `SHOW CREATE TABLE`.
        - **Application context**: Understanding _why_ the application executes a query is crucial for effective analysis and identifying simplification opportunities.
    - **Relative Values**: Metric values are **relative to the specific query and application**. Context is essential for interpretation (e.g., 1000 rows sent could be good or bad depending on intent).
    - **Average, Percentile, and Maximum Statistics**:
        - **Average**: Can be **overly optimistic** and misleading without knowing the value distribution.
        - **Percentile**: Standard practice (P95, P99, P999). P999 (99.9%) is preferred, as it discards a small, acceptable number of outliers.
        - **Maximum**: Considered the **best representation** of query time because it captures the worst user experience and provides a specific instance for debugging.
        - The **distribution of values** (between min and max) helps determine query stability.
- **Improving Query Response Time (Query Optimization)**: A "journey" with two main parts.
    
    - **Direct Query Optimization**: Involves **changes to queries and indexes**. This is often the first and most effective step, sometimes resolving the performance issue entirely. It leverages common tools like `EXPLAIN` and indexes, and specialized query-specific optimizations.
    - **Indirect Query Optimization**: Involves **changes to data and access patterns**. This requires greater effort and is pursued if direct optimization isn't sufficient.
---
- **When to Optimize Queries**: Optimization is not always necessary; only when response time is unacceptable.
    
    - **When performance affects customers**.
    - **Before and after code changes** to proactively identify regressions.
    - **Once a month**, to account for changes in data and access patterns over time.
---
- **MySQL: Go Faster**: There's **no magic solution** to make MySQL significantly faster without changing queries or the application.
    
    - **Time is a hard limit** for query throughput (QPS).
    - To make MySQL do more work in the same time, you must either **decrease response time** (by optimizing queries/application or improving hardware) or **increase load** (by executing more queries concurrently, though this has limits).
    - Pushing MySQL too hard can lead to **performance destabilization**.
---
- **Practice for Chapter 1**:
    - ==[See this tutorial for better understanding](https://www.youtube.com/watch?v=noFn2sgQiNw)==
    - **Identify Slow Queries**: Using the `pt-query-digest` command-line tool to analyze slow query logs and generate query profiles. This practice helps pinpoint which queries to optimize for the greatest performance gains.
