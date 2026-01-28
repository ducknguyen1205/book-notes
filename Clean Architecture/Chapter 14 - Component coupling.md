# COMPONENT COUPLING

This chapter introduces **three principles** that govern **relationships between components**.
These principles help manage the tension between **developability**, **maintainability**, and **architectural stability**.

The three principles are:

1. **The Acyclic Dependencies Principle (ADP)**
2. **The Stable Dependencies Principle (SDP)**
3. **The Stable Abstractions Principle (SAP)**

---

## THE ACYCLIC DEPENDENCIES PRINCIPLE (ADP)

> **Allow no cycles in the component dependency graph.**

---

### The “Morning After Syndrome”

This syndrome occurs when:

* Many developers work on interdependent components
* One developer changes something late
* Others arrive the next morning to find their code broken

As projects grow:

* Integration becomes chaotic
* Stable builds may be impossible for weeks

Two historical solutions emerged:

* **Weekly builds**
* **The Acyclic Dependencies Principle**

---

## THE WEEKLY BUILD

### How It Works

* Developers work in isolation most of the week
* Integration happens on Friday

### Problems

* Integration cost grows with project size
* Friday turns into Saturday
* Then into midweek
* Feedback slows
* Risks increase

Eventually, this approach collapses.

---

## ELIMINATING DEPENDENCY CYCLES

### The Solution

Partition the system into **releasable components**.

* Each component:

  * Has a release number
  * Can be developed independently
* Teams choose **when** to adopt new releases
* Integration happens incrementally

This works **only if the dependency graph has no cycles**.

---

## COMPONENT DEPENDENCY GRAPH (FIGURE 14.1)
![[Pasted image 20260122214042.png]]
* Components = nodes
* Dependencies = directed edges
* The graph is a **Directed Acyclic Graph (DAG)**

### Benefits

* Easy to see who is affected by a release
* Easy to test a component in isolation
* Clear build order:
  * Build bottom-up following dependencies

---

## THE EFFECT OF A CYCLE (FIGURE 14.2)
![[Pasted image 20260122214126.png]]
A new dependency creates a cycle:

* `Entities → Authorizer → Interactors → Entities`

### Consequences

* Components collapse into one large unit
* Teams must coordinate every change
* Unit testing requires many unrelated components
* Build order becomes unclear or impossible
* Build problems grow geometrically

Cycles **destroy component isolation**.

---

## BREAKING THE CYCLE

There are **two primary techniques**:

### 1. Apply the Dependency Inversion Principle (DIP)

* Create an interface in the stable component
* Implement it in the volatile component
* Dependency is inverted
* Cycle is broken
  *(Figure 14.3)*
![[Pasted image 20260122214329.png]]
---

### 2. Create a New Shared Component

* Move shared classes into a new component
* Both original components depend on it
  *(Figure 14.4)*
![[Pasted image 20260122214403.png]]
---

## THE “JITTERS”

* Component dependency structures are **not static**
* As requirements change:

  * New components appear
  * Dependencies shift
* Cycles must be detected and broken continuously

This instability is normal and expected.

---

## TOP-DOWN DESIGN DOES NOT WORK

Component structures:

* **Cannot be designed upfront**
* **Do not represent system functionality**
* Represent **buildability and maintainability**

They evolve as:

* Classes accumulate
* SRP and CCP guide grouping
* CRP influences reuse
* ADP forces cycle removal

The component graph **emerges**, it is not predefined.

---

## THE STABLE DEPENDENCIES PRINCIPLE (SDP)

> **Depend in the direction of stability.**

---

### Core Idea

* Volatile components should not be depended on by stable components
* Otherwise, volatility spreads upward
* A component becomes hard to change simply because others depend on it

---

## WHAT IS STABILITY?

Stability means:

> **Not easily changed**, not “changes infrequently”

A component is stable if:

* Many components depend on it
* Changing it requires a lot of coordination

---

## STABILITY DEFINITIONS

* **Fan-in**: Incoming dependencies
* **Fan-out**: Outgoing dependencies

### Instability Metric

```
I = Fan-out / (Fan-in + Fan-out)
```

* `I = 0` → maximally stable (many dependents, no dependencies)
* `I = 1` → maximally unstable (no dependents, many dependencies)

---

## SDP RULE

> **A component’s I value must be greater than the I values of the components it depends on.**

Dependencies must flow toward **lower instability**.

---

## NOT ALL COMPONENTS SHOULD BE STABLE

If everything were stable:

* Nothing could change
* The system would be frozen

### Ideal Configuration (Figure 14.8)
![[Pasted image 20260122214607.png]]
* Unstable components at the top
* Stable components at the bottom
* Dependency arrows point downward

---

## SDP VIOLATION (FIGURE 14.9)
![[Pasted image 20260122214658.png]]
* A stable component depends on a flexible one
* The flexible component becomes hard to change

### Fix (Using DIP)

* Introduce an interface in a new stable component
* Both components depend on the abstraction
  *(Figures 14.10–14.11)*

---

## ABSTRACT COMPONENTS

Sometimes components contain:

* Only interfaces
* No executable code

These:

* Are very stable
* Are ideal dependency targets

This is common in **statically typed languages**.

---

## THE STABLE ABSTRACTIONS PRINCIPLE (SAP)

> **A component should be as abstract as it is stable.**

---

## WHERE DO HIGH-LEVEL POLICIES GO?

* High-level business rules:

  * Should be stable
  * Should not change often
* Stability risks rigidity

### Solution (via OCP)

* Use **abstraction**
* Abstract classes allow extension without modification

---

## SAP + SDP = COMPONENT-LEVEL DIP

* **SDP** → dependencies flow toward stability
* **SAP** → stability implies abstraction
* Result:

  > **Dependencies flow toward abstractions**

---

## MEASURING ABSTRACTION

### Abstractness Metric

```
A = Na / Nc
```

* `Na`: number of abstract classes/interfaces
* `Nc`: total number of classes
* `A = 0` → fully concrete
* `A = 1` → fully abstract

---

## THE MAIN SEQUENCE (FIGURE 14.12)

* Graph with:

  * X-axis = Instability (I)
  * Y-axis = Abstractness (A)
* Ideal line from:

  * (0, 1) → stable & abstract
  * (1, 0) → unstable & concrete

---

## ZONES OF EXCLUSION (FIGURE 14.13)
![[Pasted image 20260122215001.png]]
### Zone of Pain (Near 0,0)

* Stable and concrete
* Hard to change and impossible to extend
* Example:

  * Database schemas
  * Concrete but widely depended-on utilities

### Zone of Uselessness (Near 1,1)

* Abstract but unused
* No dependents
* Often leftover, dead abstractions

---

## DISTANCE FROM THE MAIN SEQUENCE

### Distance Metric

```
D = |A + I − 1|
```

* `D = 0` → perfectly on the Main Sequence
* `D = 1` → maximally bad position

### Usage

* Identify problematic components
* Track architectural drift over time
* Analyze variance across the system
  *(Figures 14.14–14.15)*

---

## CONCLUSION

* These metrics are **guides**, not laws
* They encode architectural experience
* Used wisely, they:
  * Reveal hidden problems
  * Support maintainable design
  * Prevent architectural decay

Metrics inform judgment—they do not replace it.
