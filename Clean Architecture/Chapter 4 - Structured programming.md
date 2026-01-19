
Edsger Wybe Dijkstra (born 1930) was the Netherlands' first programmer in 1952. Despite early concerns that programming lacked discipline, Dijkstra’s boss suggested he might be the one to discover such disciplines. Dijkstra made his great discoveries in a primitive environment where the edit/compile/test loop often took hours or even days.

He recognized that programming is inherently difficult because any complex program contains **too many details for a human brain to manage**, inevitably leading to errors.

### PROOF

Dijkstra's solution was to apply the mathematical discipline of **proof** to programming. He envisioned building a **Euclidian hierarchy of postulates, theorems, corollaries, and lemmas**. Programmers would construct systems using established, proven structures and then mathematically prove the correctness of the code connecting them.

During this investigation, Dijkstra discovered that certain uses of **goto statements** prevented modules from being recursively decomposed into smaller, provable units. However, other uses of the `goto` statement did not hinder proofs and corresponded directly to simple selection and iteration control structures, such as **if/then/else** and **do/while**.

These fundamental control structures, when combined with sequential execution, had already been shown by Böhm and Jacopini to be sufficient to construct **all programs**. This realization marked the birth of structured programming.

Dijkstra’s proof techniques involved:

- **Sequential Statements:** Proved correct through **simple enumeration** (tracing inputs to outputs).
- **Iteration (Loops):** Proved correct using **induction**, which requires proving the base case (1) by enumeration, and then proving that if $N$ is correct, $N + 1$ is also correct by enumeration.

### A HARMFUL PROCLAMATION

In 1968, Dijkstra wrote a letter to the editor of CACM titled **“Go To Statement Considered Harmful”**. This led to a significant, decade-long battle within the programming world.

**Rule/Principle:** Dijkstra eventually won this argument, and as computer languages evolved, the `goto` statement virtually disappeared. Most modern languages today enforce structured programming by eliminating the option for **undisciplined direct transfer of control**. Even related structures like named breaks in Java or exceptions are not the "utterly unrestricted transfers of control" of older languages like Fortran or COBOL.

### FUNCTIONAL DECOMPOSITION

Structured programming’s discipline allows modules to be **recursively decomposed into provable units**. This means systems can undergo **functional decomposition**, where a large problem is broken down into high-level functions, which are further broken down into lower-level functions, _ad infinitum_. By adhering to these restrictions, programmers can break large proposed systems into modules and components composed of **tiny provable functions**.

### NO FORMAL PROOFS

Despite Dijkstra's groundwork, the dream of a full **Euclidean hierarchy of theorems was never built**, and few programmers today believe that formal mathematical proofs are the appropriate method for producing high-quality software.

### SCIENCE TO THE RESCUE

The sources argue that formal proofs are not the only successful strategy for demonstrating correctness; the **scientific method** is another highly successful strategy. Scientific theories are **falsifiable but not provable**; they are deemed "true enough" if they resist all efforts to prove them false, such as Newton’s laws of motion and gravity,.

**Rule/Principle:** Software development is fundamentally like a science. We show correctness by relentlessly testing the program and **failing to prove incorrectness**.

Crucially, proofs of incorrectness (testing) can only be applied to **provable programs**. A program that is not mathematically provable (e.g., one with unrestrained `goto` statements) cannot be deemed correct, regardless of how much testing is applied. Structured programming's true value is that it forces us to recursively decompose a program into a set of **small provable functions** that can then be subjected to tests, allowing them to be deemed **correct enough for our purposes**.

### CONCLUSION

The enduring value of structured programming is its ability to create **falsifiable units of programming**. This concept is why functional decomposition remains a best practice at the architectural level. Software architects apply restrictive disciplines, similar to structured programming, at a higher level to ensure that **modules, components, and services are easily falsifiable (testable)**.