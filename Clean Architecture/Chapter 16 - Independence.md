# CHAPTER 16 — INDEPENDENCE

A good architecture must support **four things simultaneously**:
* **Use cases**
* **Operation**
* **Development**
* **Deployment**

The central theme of this chapter is that **independence** is how architecture satisfies all four.

---

## USE CASES

The **first priority** of architecture is to support the **use cases** of the system.

* A shopping cart system should *look like* a shopping cart system.
* The system’s **intent** must be visible at the architectural level.
* Use cases should be:

  * First-class elements
  * Clearly named
  * Easy to find
* Developers should not have to “hunt” for business behavior.

Good architecture **reveals behavior**, even though it doesn’t strongly control behavior itself.

---

## OPERATION

Architecture has a **real impact** on how the system runs.

Examples of operational requirements:

* High throughput (e.g., 100,000 requests/sec)
* Low latency (e.g., millisecond queries)
* Scalability

Possible runtime structures:

* Single monolith
* Multiple processes
* Many threads
* Distributed services / microservices

### Key Rule

> **A good architecture leaves the choice of operational structure open.**

If the system is tightly coupled to one execution model, changing later becomes very difficult.

Properly isolated components make it possible to move:

* From monolith → services
* From services → monolith

---

## DEVELOPMENT

Architecture must support **team independence**.

### Conway’s Law

> Systems mirror the communication structure of the organizations that build them.

To reduce team interference:

* Partition the system into **well-isolated components**
* Assign components to teams
* Allow teams to work independently

Decoupled architecture works regardless of whether teams are:

* Feature teams
* Layer teams
* Component teams

---

## DEPLOYMENT

A good architecture enables **easy and immediate deployment**.

### Goal

> **Immediate deployment after build**

Bad architectures require:

* Manual configuration
* Fragile scripts
* Carefully ordered startup steps

Good architectures:

* Isolate components cleanly
* Clearly define startup and integration points

---

## LEAVING OPTIONS OPEN

The difficulty:

* Use cases change
* Operational needs change
* Team structures change
* Deployment needs change

### Architectural Strategy

> **Leave as many options open as possible, for as long as possible.**

This is achieved through **decoupling**.

---

## DECOUPLING LAYERS (HORIZONTAL)

Different parts of the system change for **different reasons**:

* UI changes independently of business rules
* Business rules change independently of databases
* Domain rules change independently of application rules

Therefore, separate:

* UI
* Application-specific business rules
* Application-independent (domain) rules
* Database / infrastructure

These form **horizontal layers**.

---

## DECOUPLING USE CASES (VERTICAL)

Use cases also change independently.

* “Add Order” evolves differently from “Delete Order”
* Each use case is a **vertical slice** through the layers

Architecture should:

* Keep use cases separate
* Avoid coupling their UI, rules, or data access

Result:

* New use cases can be added without breaking old ones

---

## DECOUPLING MODE

Decoupling for use cases also helps **operations**.

If components are independent:

* High-throughput parts can be scaled separately
* UI, rules, and database can run on different servers

To achieve this, components must not assume:

* Same address space
* Same process
* Same deployment unit

---

## INDEPENDENT DEVELOPABILITY

When layers and use cases are decoupled:

* Teams do not interfere with each other
* Changes are localized
* Parallel development becomes practical

---

## INDEPENDENT DEPLOYABILITY

With proper decoupling:

* Components can be deployed independently
* New use cases can be added by:

  * Adding new jars
  * Adding new services
* Existing components remain untouched

---

## DUPLICATION (IMPORTANT WARNING)

Not all duplication is bad.

### Two Types

1. **True duplication**

   * Changes must always be mirrored
   * Must be eliminated

2. **Accidental duplication**

   * Looks similar today
   * Changes for different reasons
   * Should often remain separate

##### One-Sentence Rule of Thumb

> **If two pieces of code must change together, merge them.  
> If they might change for different reasons, keep them separate.**

This distinction is critical to preserving **independence** in Clean Architecture.

### Common Traps

* Sharing UI code between similar use cases
* Passing database records directly to the UI
* Avoiding view models because they “look the same”

These shortcuts **increase coupling** and make future changes harder.

---

## DECOUPLING MODES (THREE LEVELS)

### 1. Source-Level Decoupling

* Separate source modules
* Same process, same executable
* Communication via function calls
* Typical monolith

### 2. Deployment-Level Decoupling

* Separate deployable units (JARs, DLLs, Gems)
* May run in same or different processes
* Independent builds and deployments

### 3. Service-Level Decoupling

* Independent services
* Communicate via network messages
* No shared binaries or source code

---

## WHICH MODE IS BEST?

There is **no universal answer**.

* Early systems may only need source-level decoupling
* Later, deployment-level or service-level may be required
* Needs may also **shrink** over time

### Key Insight

> Decoupling mode is an architectural option that should remain flexible.

Starting with microservices by default:

* Is expensive
* Encourages coarse-grained boundaries
* Often wastes development effort

---

## PREFERRED STRATEGY

* Start with **source-level decoupling**
* Design so services *could* exist
* Delay creating services until truly necessary
* Move between modes as needs change

A good architecture allows:

* Monolith → deployable units → services
* And back again if needed

---

## CONCLUSION

A good architecture:

* Supports use cases clearly
* Scales operationally
* Enables independent teams
* Simplifies deployment
* Avoids premature decisions
* Keeps options open

> **Independence is the foundation that allows systems to evolve safely over time.**
