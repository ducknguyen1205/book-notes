# THE OPEN-CLOSED PRINCIPLE (OCP)

The **Open-Closed Principle (OCP)** was introduced in 1988 by **Bertrand Meyer**.

## Definition

> **A software artifact should be open for extension but closed for modification.**

This means the behavior of a system should be extendable **without changing existing code**.

This is a fundamental motivation for software architecture. If small requirement changes cause large modifications throughout the system, the architecture has failed.

Although OCP is often taught at the **class and module level**, its **most powerful impact is at the architectural level**.

---

## A THOUGHT EXPERIMENT

### Scenario

A system displays a **financial summary on a web page**:

* Scrollable content
* Negative numbers shown in red

New requirement:

* Generate a **black-and-white printed report**
* Proper pagination
* Page headers and footers
* Column labels
* Negative numbers shown in parentheses

### Key Question

How much **existing code** must be modified?

A well-designed architecture should reduce this to **as close to zero as possible**.

---

## Applying SRP First

To achieve OCP, we first apply the **Single Responsibility Principle (SRP)**.

### Insight

Report generation consists of **two separate responsibilities**:

1. **Calculation of reportable data**
2. **Presentation of that data** (web vs printer)
![[Pasted image 20260112221733.png]]
Diagram: Figure 8.1 – Applying the SRP

* An **analysis process** converts raw financial data into reportable data
* Separate **reporter processes** format that data for:

  * Web output
  * Printer output

This separation ensures that changes in presentation do not affect calculations.

---

## Architectural Organization
![[Pasted image 20260112221837.png]]
Diagram: Figure 8.2 – Partitioning into Components

Processes are:

* Partitioned into **classes**
* Grouped into **components**

Key components:

* **Controller**
* **Interactor**
* **Database**
* **Presenters**
* **Views**

Notation:

* `<I>` = interface
* `<DS>` = data structure
* Open arrow = “uses” relationship
* Closed arrow = “implements / inherits” relationship

All arrows represent **source-code dependencies**.

---

## Dependency Direction Matters

An arrow from A to B means:

* A’s source code mentions B
* B does not know about A

Example:

* `FinancialDataMapper` depends on `FinancialDataGateway`
* `FinancialDataGateway` does not depend on `FinancialDataMapper`

---

## Component Dependency Rule
![[Pasted image 20260112222048.png]]
Diagram: Figure 8.3 – Unidirectional Component Relationships

All component dependencies cross boundaries in **one direction only**.

### Rule

> **If component A should be protected from changes in component B, then component B must depend on component A.**

#### Explain

##### Example: Business Rules vs Database

###### Components

- **A: Interactor (Business Rules)**
    
- **B: Database (Persistence Implementation)**

###### Requirement

You want to:

- Change the database (MySQL → PostgreSQL, or SQL → NoSQL)
    
- **Without changing business rules**
    

Therefore:

- **Interactor must be protected from Database changes**
    

---

##### Correct Dependency Direction (Protected)

```text
Database  ───►  Interactor
```

##### How this works

- Interactor defines an interface:
    
    ```java
    interface OrderRepository {
        Order findById(Id id);
    }
    ```
    
- Database component implements that interface:
    
    ```java
    class SqlOrderRepository implements OrderRepository {
        ...
    }
    ```
##### Result

- Changing SQL to NoSQL only affects the **Database component**
    
- **Interactor code does not change**
    
- Interactor is **closed for modification**, but behavior is **extended**
    

---

##### Wrong Dependency Direction (Not Protected)

```text
Interactor  ───►  Database
```

##### Consequence

- Database schema or technology changes
    
- Interactor must be modified
    
- Business rules become unstable
##### One-Sentence Summary

To protect **A (business rules)** from **B (details)**, **B must depend on A**, usually by implementing interfaces defined by A.

---

## Protection Hierarchy

The system is organized into a **hierarchy of protection**:

* **Interactor**

  * Most protected
  * Contains **business rules**
  * Highest-level policies
  * Should not be affected by changes anywhere else

* **Controller**

  * Protected from Presenters
  * Peripheral to Interactor

* **Presenters**

  * Protected from Views
  * Central to Views but peripheral to Controllers

* **Views**

  * Least protected
  * Lowest-level details

Higher-level components are **shielded from changes** in lower-level components.

---

## Why the Interactor Is Central

The **Interactor**:

* Encapsulates core business logic
* Represents the system’s central concern
* Other components handle peripheral concerns (UI, database, formatting)

This is OCP at the architectural level.

---

## DIRECTIONAL CONTROL

The complexity in the design exists to ensure **dependencies point in the correct direction**.

### Example

* `FinancialDataGateway` interface:

  * Prevents dependency from Interactor → Database
  * Inverts the dependency using DIP

The same pattern is applied for:

* Report presenters
* View interfaces

This ensures **high-level policies do not depend on low-level details**.

---

## INFORMATION HIDING

### FinancialReportRequester Interface

Purpose:

* Prevents the Controller from knowing internal details of the Interactor

Without it:

* Controller would have **transitive dependencies** on `FinancialEntities`

### Why This Matters

Transitive dependencies violate a key principle:

> Software entities should not depend on things they do not directly use.

This idea reappears later in:

* Interface Segregation Principle (ISP)
* Common Reuse Principle (CRP)

Even while protecting the Interactor, we must **also protect the Controller** by hiding Interactor internals.

---

## CONCLUSION

The **Open-Closed Principle** is a primary driver of system architecture.

### Goal

* Enable **easy extension**
* Minimize the **impact of change**

### How This Is Achieved

* Partition the system into components
* Arrange components into a **dependency hierarchy**
* Protect **high-level components** from changes in **low-level components**

This is how OCP scales from classes to **entire system architectures**.

