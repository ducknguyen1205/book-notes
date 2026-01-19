# THE INTERFACE SEGREGATION PRINCIPLE (ISP)

The **Interface Segregation Principle (ISP)** is illustrated by the diagram in **Figure 10.1**.
![[Pasted image 20260116214021.png]]
## Core Idea

> **Clients should not be forced to depend on methods they do not use.**

---

## Figure 10.1 – The Problem

* Class **OPS** provides operations:

  * `op1`, `op2`, `op3`
* Users:

  * **User1** uses only `op1`
  * **User2** uses only `op2`
  * **User3** uses only `op3`

### Problem in Statically Typed Languages (e.g., Java)

* User1’s source code must depend on **OPS**
* This creates **implicit dependencies** on `op2` and `op3`
* If `op2` or `op3` changes:

  * User1 must be recompiled
  * User1 must be redeployed
* This happens even though User1 does not use those operations

---

## Figure 10.2 – Segregated Interfaces (The Solution)

* Split OPS into multiple interfaces:

  * `U1Ops` → `op1`
  * `U2Ops` → `op2`
  * `U3Ops` → `op3`

### Result

* User1 depends only on:

  * `U1Ops`
  * `op1`
* User1 no longer depends on OPS or unrelated operations
* Changes to `op2` or `op3` do **not** affect User1

This reduces **unnecessary recompilation and redeployment**.

---

## ISP AND LANGUAGE

### Statically Typed Languages (Java, C#)

* Source code dependencies are explicit
* Imports and declarations create coupling
* ISP is critical to avoid forced recompilation

### Dynamically Typed Languages (Python, Ruby)

* Dependencies are resolved at runtime
* No compile-time declarations
* Less forced coupling

This might suggest that ISP is merely a **language-level concern**.

---

## ISP AND ARCHITECTURE

The deeper motivation behind ISP goes beyond programming languages.

### Architectural Insight

> **It is harmful to depend on modules that contain more than you need.**

This applies to:

* Source code modules
* Libraries
* Frameworks
* Services
* Entire components

---

## Figure 10.3 – A Problematic Architecture
![[Pasted image 20260116214148.png]]
* System **S** depends on framework **F**
* Framework **F** depends on database **D**
* Database **D** contains features unused by **F** and **S**

### Consequences

* Changes in unused parts of **D** can:

  * Force redeployment of **F**
  * Force redeployment of **S**
* Failures in unused features of **D** can:

  * Break **F**
  * Break **S**

This is an architectural violation of ISP.

---

## CONCLUSION

The key lesson of ISP is broader than interfaces:

> **Depending on something that carries baggage you do not need creates hidden risks.**

These risks include:

* Unnecessary redeployments
* Fragile systems
* Unexpected failures

This idea is expanded further in **Chapter 13: The Common Reuse Principle**.

