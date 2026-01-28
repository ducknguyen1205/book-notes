# WHAT IS ARCHITECTURE?

The word *architecture* often implies authority and power, but in software this perception is misleading.

## What a Software Architect Really Is

A **software architect is a programmer** and remains one.

* Architects do **not** step away from code
* They are typically the **best programmers**
* They continue writing code while:
  * Guiding design
  * Experiencing the consequences of architectural decisions
  * Protecting team productivity

An architect who does not write code cannot do the job properly.

---

## What Architecture Is

> **The architecture of a software system is the shape given to that system by those who build it.**

That shape consists of:

* Division into **components**
* Arrangement of those components
* Ways components **communicate**

---

## Purpose of Architecture

The purpose of architecture is **not primarily correctness**.

Many poorly architected systems work correctly.

### True Purpose

> **To support the system’s life cycle**:

* Development
* Deployment
* Operation
* Maintenance

The ultimate goal:

* Minimize lifetime cost
* Maximize programmer productivity

---

## DEVELOPMENT

Architecture must support **how teams work**.

### Small Teams

* Can move fast with:
  * Monoliths
  * Few boundaries
* Heavy architecture may slow early progress

### Large Teams

* Require:
  * Clear component boundaries
  * Stable interfaces
* Architecture often evolves into:
  * One component per team

This structure may optimize development speed but **not necessarily** deployment or maintenance.

---

## DEPLOYMENT

A system is only useful if it is **easy to deploy**.

### Architectural Goal

* Deploy with **a single action**

### Common Failure

Deployment is often ignored early.

Example:

* Microservices make development easier
* But:

  * Too many services
  * Complex startup ordering
  * Fragile configuration

Better early thinking could lead to:

* Fewer services
* Hybrid architectures
* Simpler deployment pipelines

---

## OPERATION

Architecture impacts operation **less** than other lifecycle phases.

* Operational problems can often be solved with more hardware
* Hardware is cheap; people are expensive

### Architecture’s Operational Role

Architecture should:

* **Reveal how the system operates**
* Make use cases and behaviors visible
* Improve understanding for developers

This aids development and maintenance.

---

## MAINTENANCE

Maintenance is the **largest long-term cost**.

Two major costs:

1. **Spelunking** – finding where to change the code
2. **Risk** – accidentally breaking existing behavior

### Architectural Solution

* Separate system into components
* Isolate components with stable interfaces
* Illuminate clear paths for change
* Reduce accidental damage

---

## KEEPING OPTIONS OPEN

Software has two values:

1. **Behavior**
2. **Structure** (more valuable)

Software is valuable because it can **change easily**.

### Policy vs Details

All systems contain:

* **Policy**

  * Business rules
  * Core behavior
  * True value of the system
* **Details**

  * Databases
  * Web frameworks
  * IO devices
  * Protocols
  * Servers

### Architect’s Goal

> **Make policy independent of details.**

This allows decisions about details to be:

* Delayed
* Deferred
* Changed later with minimal cost

---

## Examples of Deferred Decisions

* Database type (SQL, NoSQL, flat files)
* Web framework
* REST vs other protocols
* Dependency injection framework
* Microservices vs monolith

If policy does not know about these details, decisions can be postponed until more information is available.

---

## MAXIMIZING UNMADE DECISIONS

A good architect:

* Pretends decisions have **not** been made
* Shapes the system to keep options open
* Maximizes the number of decisions **not made**

This enables:

* Experimentation
* Better-informed choices
* Lower long-term cost

---

## DEVICE INDEPENDENCE (Historical Example)

### The Problem

Early programs:

* Were tied directly to hardware devices
* Used device-specific instructions

Changing devices required **rewriting programs**.

---

### The Solution

Operating systems introduced:

* IO abstractions
* Device-independent interfaces

Programs:

* Used abstract IO services
* Worked with printers, tape, or cards
* Without code changes

This was an early application of **OCP**.

---

## JUNK MAIL STORY

Programs:

* Formatted customer data
* Printed personalized ads

By abstracting IO:

* Same program printed to:

  * Line printers (testing)
  * Magnetic tape (production)
* Offline printers handled bulk work

### Lesson

Separating **policy (formatting)** from **detail (device)** enabled:

* Massive scalability
* Operational flexibility

---

## PHYSICAL ADDRESSING STORY

### The Mistake

* Business rules depended on:

  * Disk geometry
  * Cylinder/head/sector addressing

Changing disk hardware required:

* Data migration
* Widespread code changes

---

### The Fix

* Introduced **logical (relative) addressing**
* Isolated physical structure in one place
* Business rules became hardware-agnostic

### Result

* Disk changes no longer affected policy
* Major decision was deferred successfully

---

## CONCLUSION

Good architecture:

* Separates **policy from details**
* Makes details irrelevant to policy
* Defers decisions as long as possible

> **The more decisions you delay, the more flexibility and power you preserve.**

This principle applies:

* In small design choices
* In large system architecture
