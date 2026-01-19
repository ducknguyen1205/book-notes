## SOLID Principles – Overview

Good software systems start with **clean code**, but clean code alone is not sufficient. Even well-written code can form a poor system if it is arranged badly. This is where the **SOLID principles** apply.

SOLID principles guide how **functions and data are arranged into classes (or modules)** and how those groupings relate to one another.  
The word _class_ does **not** imply object-oriented programming only. A class is simply a **coupled grouping of functions and data**, which exists in all software systems.

### Goals of the SOLID Principles

SOLID aims to create **mid-level software structures** that:

- Tolerate change
    
- Are easy to understand
    
- Form the basis of reusable components
    

“Mid-level” means these principles apply **above individual functions**, at the **module level**, but **below system architecture**.

Even well-designed mid-level components can create system-wide problems. For this reason, SOLID principles are followed later by **component-level principles** and **high-level architectural principles**.

### Historical Context

- Developed gradually starting in the late 1980s
    
- Stabilized in the early 2000s
    
- Named “SOLID” after Michael Feathers noticed the acronym in 2004
    

The following chapters in the book expand on each principle, focusing on **architectural implications** rather than repeating basic definitions.

---

## Executive Summary of SOLID

- **SRP – Single Responsibility Principle**  
    Each module should have **one, and only one, reason to change**, driven by a specific group of stakeholders.
    
- **OCP – Open-Closed Principle**  
    Software should be **open for extension** but **closed for modification**. Behavior should change by adding code, not modifying existing code.
    
- **LSP – Liskov Substitution Principle**  
    Subtypes must be substitutable for their base types without breaking correctness.
    
- **ISP – Interface Segregation Principle**  
    Clients should not be forced to depend on interfaces they do not use.
    
- **DIP – Dependency Inversion Principle**  
    High-level policy should not depend on low-level details; **details should depend on policies**.
    

---

# SRP: THE SINGLE RESPONSIBILITY PRINCIPLE

The **Single Responsibility Principle (SRP)** is often misunderstood because of its name.  
It **does not** mean “a module should do only one thing.”

That idea applies to **functions**, not to SRP.

### Traditional Definition

> A module should have one, and only one, reason to change.

Systems change because of **people** who request those changes.  
Therefore, the principle can be rephrased as:

> A module should be responsible to one, and only one, user or stakeholder.

However, multiple people often want the same change. Therefore, the correct abstraction is an **actor**:

### Final Definition of SRP

> **A module should be responsible to one, and only one, actor.**

### What Is a Module?

- Usually: a **source file**
    
- More generally: a **cohesive set of functions and data**
    

The word **cohesive** is central.  
Cohesion means the code exists to serve **one actor**.

---

## Symptoms of SRP Violation

### SYMPTOM 1: ACCIDENTAL DUPLICATION

**Example: Employee class in a payroll system**

The class contains:

- `calculatePay()` – used by Accounting (CFO)
    
- `reportHours()` – used by HR (COO)
    
- `save()` – used by DBAs (CTO)
    

Each method serves a **different actor**, yet they are placed in the same class.

#### Problem

To avoid duplication, a shared method `regularHours()` is created and used by both `calculatePay()` and `reportHours()`.

Later:

- CFO requests a change to how regular hours are calculated
    
- HR does **not** want this change
    

A developer modifies `regularHours()` for Accounting, unaware that HR also depends on it.  
Accounting validates the change, the system is deployed, and HR reports become incorrect.

#### Result

- HR data is wrong
    
- Budget decisions are affected
    
- COO is understandably upset
    

#### Root Cause

Code used by **different actors** was placed too close together.

**SRP demands separation of code based on actors.**

---

### SYMPTOM 2: MERGES

When one source file serves multiple actors:

- Multiple teams modify the same file
    
- Changes collide
    
- Merges become frequent

**Example:**

- DBAs change database schema handling
    
- HR changes report format
    
- Both modify the `Employee` class
    

#### Risks

- Merges are inherently risky
    
- Even with good tools, errors can slip in
    
- Multiple executives are exposed to failure from unrelated changes
    

Again, the cause is **multiple actors sharing the same module**.

---

## SOLUTIONS

### Solution 1: Separate Responsibilities into Different Classes

- Move each responsibility into its own class
    
- All classes share `EmployeeData` (a simple data structure)
    
- Classes do **not** know about each other
    
![[Pasted image 20260110222212.png]]
**Diagram (Figure 7.3):**  
Three independent classes operate on shared data but are not coupled to each other.

**Benefit:**

- Eliminates accidental duplication
    
- Isolates changes by actor

**Downside:**

- More objects to manage

---

### Solution 2: Use the Facade Pattern

To reduce complexity:

- Introduce `EmployeeFacade`
    
- Facade creates and delegates to the specialized classes
    
![[Pasted image 20260110222309.png]]
**Diagram (Figure 7.4):**  
Facade coordinates interactions without containing business logic.

---

### Solution 3: Keep Core Business Logic Close to Data

Alternative approach:

- Keep the most important business method in `Employee`
    
- Move lesser responsibilities to separate classes
    
- `Employee` acts as a Facade
    
![[Pasted image 20260110222625.png]]
**Diagram (Figure 7.5):**  
Primary responsibility remains central; others are delegated.

---

### Clarification: “One Function per Class” Is a Misconception

Each responsibility may require **many private helper methods**.

Each class defines a **scope**:

- Inside the scope: many related functions
    
- Outside the scope: internal details are hidden
    

---

## CONCLUSION

The **Single Responsibility Principle** applies at multiple levels:

- **Function / Class level** → SRP
    
- **Component level** → Common Closure Principle
    
- **Architectural level** → Axis of Change and Architectural Boundaries
    

SRP is fundamentally about **separating code based on who needs it to change**.  
This idea scales from small modules to entire system architectures.