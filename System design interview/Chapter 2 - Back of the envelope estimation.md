#  Summary

Chapter 2 focuses on **Back-of-the-Envelope Estimation**, a crucial skill for system design interviews. According to Jeff Dean, a Google Senior Fellow, these calculations are **estimates created using a combination of thought experiments and common performance numbers** to evaluate whether potential designs will meet requirements.

To effectively perform back-of-the-envelope estimations, you need a solid understanding of scalability basics. The chapter highlights three key concepts that should be well understood:

1. Power of two.
2. Latency numbers every programmer should know.
3. Availability numbers.

### Power of two

This section explains the **data volume units using the power of 2**. While data volume in distributed systems can be enormous, calculations rely on these basics. A byte consists of 8 bits, and an ASCII character uses one byte. The chapter includes **Table 2-1** which explains the data volume units.
![[Pasted image 20250519101737.png]]

### Latency numbers every programmer should know

Dr. Dean from Google provided typical computer operation lengths in 2010. While some numbers are now outdated due to faster computers, they still offer insight into the relative speed and slowness of different operations.

![[Pasted image 20250519101854.png]]

The chapter includes **Notes** defining nanosecond (ns), microsecond (µs), and millisecond (ms) and their relationships (1 ns = 10⁻⁹ seconds, 1 µs = 10⁻⁶ seconds = 1,000 ns, 1 ms = 10⁻³ seconds = 1,000 µs = 1,000,000 ns).

A Google software engineer built a tool to visualize Dr. Dean's numbers, taking the time factor into consideration. The chapter refers to **Figures 2-1** which show these visualized latency numbers as of 2020. By analyzing these numbers (presumably shown in Figure 2-1), the following conclusions are drawn:

- **Memory is fast, but the disk is slow.**
- **Avoid disk seeks if possible.**
- **Simple compression algorithms are fast.**
- **Compress data before sending it over the internet if possible.**
- **Data centers are usually in different regions, and sending data between them takes time.**

![[Pasted image 20250519103747.png]]

### Availability numbers

**High availability** is defined as a system's ability to be continuously operational for a desired long period. It is measured as a percentage, where 100% indicates zero downtime. Most services typically have availability between 99% and 100%.
![[Pasted image 20250519103939.png]]
A **Service Level Agreement (SLA)** is a common term for service providers, formally defining the level of uptime delivered to customers. Cloud providers like Amazon, Google, and Microsoft typically set their SLAs at 99.9% or higher. Uptime is traditionally measured in "nines". The more nines, the better the availability. The chapter includes **Table 2-3**, which correlates the number of nines to the expected system downtime.


![[Pasted image 20250519104053.png]]
# Notes
## L1, L2 Cache

L1 and L2 refer to cache levels in computer architecture, forming part of the memory hierarchy. They're named in order of proximity to the CPU, with lower numbers indicating closer and faster caches.

### Cache Levels Explained

- **L1 Cache**: The smallest, fastest cache located directly on the CPU core
- **L2 Cache**: Larger but slightly slower cache, either on the CPU core or shared between cores
- **L3 Cache (not mentioned but common)**: Even larger shared cache among all cores

### Key Differences

| Feature  | L1 Cache             | L2 Cache                   |
| -------- | -------------------- | -------------------------- |
| Size     | Typically 32-64KB    | 256KB-2MB                  |
| Speed    | Fastest (1-3 cycles) | Fast (10-20 cycles)        |
| Location | On CPU core          | On or near CPU             |
| Scope    | Private to each core | Often shared between cores |

### Real-World Examples

1. **Web Application Caching**:
   - L1: In-memory process cache (application variables)
   - L2: Redis/Memcached distributed cache

2. **Database Systems**:
   - L1: Buffer pool in memory
   - L2: SSD-based cache

3. **Content Delivery Networks**:
   - L1: Edge server cache (closest to users)
   - L2: Regional/origin caches
