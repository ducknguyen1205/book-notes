
Software architecture begins with the code, and this chapter overviews the major programming paradigms that have emerged over time. A paradigm is defined as a way of programming, unrelated to language, that dictates which programming structures to use and when to use them. To date, there have been three such paradigms, and the sources suggest there are unlikely to be any others.

The three paradigms discussed are structured programming, object-oriented programming, and functional programming.

# STRUCTURED PROGRAMMING

The first paradigm to be widely adopted was **structured programming**, which was discovered by Edsger Wybe Dijkstra in 1968. Dijkstra showed that utilizing unrestrained jumps, specifically **goto statements**, is harmful to a program’s structure. He replaced these jumps with familiar constructs like **if/then/else** and **do/while/until**.

**Principle:**

> **Structured programming imposes discipline on direct transfer of control**.

# OBJECT-ORIENTED PROGRAMMING

The second paradigm adopted, **object-oriented programming (OOP)**, was discovered earlier, in 1966, by Ole Johan Dahl and Kristen Nygaard. They observed that the function call stack frame could be moved to the heap, which led to the disciplined use of function pointers and the discovery of **polymorphism**.

**Principle:**

> **Object-oriented programming imposes discipline on indirect transfer of control**.

# FUNCTIONAL PROGRAMMING

The third paradigm, **functional programming**, was actually the first to be invented, predating computer programming itself. It originated from the work of Alonzo Church and his **$\lambda$-calculus** in 1936. A foundational concept of $\lambda$-calculus is **immutability**, meaning the values of symbols do not change. Because of this, a functional language fundamentally has **no assignment statement**. While most functional languages include mechanisms to alter variable values, they do so only under **very strict discipline**.

**Principle:**

> **Functional programming imposes discipline upon assignment**.

# FOOD FOR THOUGHT

A striking observation about these three paradigms is that **each one removes capabilities from the programmer**; none of them introduce new capabilities. They each impose a discipline that is negative in its intent, telling programmers **what not to do** rather than what to do. Collectively, the three paradigms remove goto statements, function pointers (or indirect control), and assignment. Since it is likely that nothing fundamental remains to be removed, these three paradigms are probably the **only three** negative paradigms that will ever be seen. Furthermore, all three were developed between 1958 and 1968, and no new paradigms have been added in the many decades since.

# CONCLUSION

The three programming paradigms align well with the three major concerns of architecture: **function, separation of components, and data management**. Software is not a rapidly advancing technology; despite changes in tools and hardware, the basic building blocks of software—**sequence, selection, iteration, and indirection**—have remained consistent since the 1940s. This changelessness of the code is the reason the rules of software architecture, which are the rules for ordering and assembling these universal building blocks, are so consistent across different system types. The goal of this book is to describe these **timeless, changeless rules**.