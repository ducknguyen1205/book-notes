## Architecture Is About Boundaries

> **Software architecture is the art of drawing boundaries.**

Boundaries:

* Separate software elements
* Prevent one side from knowing about the other
* Protect **core business rules** from volatile decisions

Some boundaries are drawn:

* Very early (even before code)
* Much later, as the system evolves

The purpose of boundaries is to:

* **Defer decisions**
* Prevent those decisions from **polluting business logic**

---

## Why Boundaries Matter

The architect’s goal is to:

> **Minimize the human effort required to build and maintain the system**

The main enemy is **coupling**, especially coupling to **premature decisions**.

### Premature Decisions Include

* Frameworks
* Databases
* Web servers
* Utility libraries
* Dependency injection tools
* Architectural “styles” (e.g., 3-tier, SOA)

Good architecture makes these:

* **Ancillary** (phụ trợ)
* **Deferrable**
* **Replaceable**

---

## A COUPLE OF SAD STORIES

### Company P — Premature Distribution

**What happened**

* Desktop app rewritten as a “three-tier architecture”
* Assumed future server farms
* Domain objects duplicated across GUI, middleware, database tiers
* Heavy serialization, messaging, and networking

**Result**

* Enormous development overhead
* All deployments ran on a single server
* Distribution complexity was never needed

**Lesson**

> **Premature architectural decisions multiply development effort.**

---

### Company W — Premature SOA

**What happened**

* Small business forced into enterprise-scale SOA
* Every change required:

  * Service discovery
  * Message creation
  * Multiple service calls
  * Complex testing setup

**Result**

* Extreme coupling
* Massive redeployment effort
* Developer productivity collapsed

**Lesson**

> **Tools that promise scalability can destroy productivity if adopted too early.**

---

## AN ARCHITECTURAL SUCCESS — FITNESSE

### Core Strategy

* **Defer decisions aggressively**
* Draw boundaries early between:

  * Business rules
  * Databases
  * Web frameworks

---

### “Download and Go” Rule

* Only one JAR file
* No complex setup
* Influenced all architectural decisions

---

### Database Decision Deferred

#### Design

* Business rules depend on a `WikiPage` interface
* Multiple implementations:

  * `MockWikiPage`
  * `InMemoryPage`
  * `FileSystemWikiPage`
  * (Later) `MySqlWikiPage`

#### Benefits

* 18 months of development with **no database**
* Faster tests
* No schema or connection issues
* Database choice postponed—and eventually avoided

---

## WHERE TO DRAW BOUNDARIES

> **Draw boundaries between things that matter and things that don’t.**

### What “Doesn’t Matter” to Business Rules

* GUI
* Database
* Frameworks
* IO mechanisms

Business rules need only:

* **Simple interfaces**
* No knowledge of implementation details

---

## DATABASE BEHIND AN INTERFACE
![[Pasted image 20260131094831.png]]
![[Pasted image 20260131094845.png]]
![[Pasted image 20260131094930.png]]
### Diagram Explanation (Figures 17.1–17.3)

* `BusinessRules` depend on `DatabaseInterface`
* `DatabaseAccess` implements the interface
* Boundary is drawn **below the interface**
* Dependency arrows point **toward business rules**

### Key Insight

> The database depends on the business rules, not the other way around.

This allows:

* Database replacement
* Deferred technology decisions
* Independent testing

---

## INPUT AND OUTPUT ARE IRRELEVANT

### Core Idea

> **IO does not define the system.**

The real system is:

* The **model**
* The **business rules**

GUIs are just:

* One possible way to interact with that model

---

## GUI AS A PLUGIN

### Diagram Explanation (Figure 17.4)

* GUI depends on BusinessRules
* BusinessRules do not depend on GUI

This allows:

* GUI replacement (web, desktop, CLI, API)
* Business rules remain untouched

---

## PLUGIN ARCHITECTURE

By treating GUI and database as plugins:

* The system becomes extensible
* Core business rules are protected
![[Pasted image 20260131095550.png]]
### Diagram Explanation (Figure 17.5)

* Business rules at the center
* Plugins connect outward
* Dependency arrows point inward

---

## THE PLUGIN ARGUMENT (RESHARPER EXAMPLE)

### ReSharper vs Visual Studio

* ReSharper depends on Visual Studio
* Visual Studio does **not** depend on ReSharper

### Result

* ReSharper cannot break Visual Studio
* Visual Studio can break ReSharper

### Desired Asymmetry

> **Core business rules should be immune to plugins.**

---

## BOUNDARIES AND AXES OF CHANGE

Boundaries are drawn where:

* Change rates differ
* Reasons for change differ

Examples:

* GUI vs business rules
* Business rules vs database
* Business rules vs frameworks

This is **SRP at the architectural level**.

---

## CONCLUSION

To draw boundaries correctly:

1. Partition the system into components
2. Identify **core business rules**
3. Treat everything else as plugins
4. Arrange dependencies so arrows point **toward the core**
5. Defer decisions as long as possible

These boundaries:

* Prevent change propagation
* Reduce coupling
* Preserve flexibility
* Minimize long-term cost

> **Good architecture is not about choosing technologies—it is about keeping yourself free from them.**

