# COMPONENT COHESION

Deciding **which classes belong in which components** is a core architectural concern. Historically, this decision was often made ad hoc. This chapter introduces **three principles of component cohesion** that provide structured guidance:

* **REP – Reuse/Release Equivalence Principle**
* **CCP – Common Closure Principle**
* **CRP – Common Reuse Principle**

---

## THE REUSE / RELEASE EQUIVALENCE PRINCIPLE (REP)

> **The granule of reuse is the granule of release.**

### Meaning

A component that is intended to be reused **must be released, versioned, and documented** as a unit.

Modern dependency tools (Maven, Leiningen, RVM, etc.) exist because:

* Reusable components must be tracked by version
* Users need to know:

  * What changed
  * When new releases are available
  * Whether they should upgrade

### Architectural Implication

Classes grouped into a component must:

* Serve a **cohesive purpose**
* Be reasonable to **release together**
* Share:

  * Version number
  * Release notes
  * Compatibility expectations

If a component feels like a random collection of classes, it violates REP—and users will notice.

### Limitation

REP is intentionally vague. “Making sense” is hard to formalize.
Its weaknesses are addressed by the next two principles.

---

## THE COMMON CLOSURE PRINCIPLE (CCP)

> **Gather into components those classes that change for the same reasons and at the same times.
> Separate classes that change for different reasons.**

### Core Idea

CCP is the **Single Responsibility Principle applied to components**.

A component should have **one reason to change**, just as a class should.

### Why This Matters

For most systems:

* **Maintainability is more important than reuse**
* When requirements change, it is better if:
  * Changes affect **one component**
  * Only that component must be rebuilt, tested, and redeployed

Classes that always change together **belong together**.

---

## Relationship to OCP

* OCP: Classes should be *closed* to common changes
* CCP: Classes closed to the **same kinds of changes** should be in the same component

Since perfect closure is impossible:

* Architects choose **which changes to protect against**
* CCP limits the impact of unavoidable changes to **few components**

---

## SIMILARITY WITH SRP

The SRP and CCP are the same idea at different levels.

* **SRP** → separates *methods* into classes
* **CCP** → separates *classes* into components

### Shared Rule

> **Gather together things that change for the same reasons and at the same times.
> Separate things that change for different reasons.**

---

## THE COMMON REUSE PRINCIPLE (CRP)

> **Don’t force users of a component to depend on things they don’t need.**

### Core Idea

Classes that are **reused together** should be placed in the **same component**.

Reusable abstractions usually involve **collaborating classes**, not isolated ones.

### Example

* A container class
* Its iterator classes

These are tightly coupled and reused together → same component.

---

## What CRP Really Warns Against

When one component depends on another:

* It depends on **every class in that component**
* Even if it uses only one class

Consequences:

* Any change in the used component may force:

  * Recompilation
  * Revalidation
  * Redeployment

Even if the change is irrelevant to the user.

### CRP Rule (Negative Form)

Classes that are **not tightly bound** should **not** be in the same component.

Components should be **inseparable units of reuse**.

---

## RELATION TO ISP

CRP is the **component-level version of ISP**.

* **ISP**: Don’t depend on methods you don’t use
* **CRP**: Don’t depend on components that contain classes you don’t use

### Unified Sound Bite

> **Don’t depend on things you don’t need.**

---

## THE TENSION DIAGRAM FOR COMPONENT COHESION

The three principles **pull against each other**:

* **REP** → larger components (reuse-focused)
* **CCP** → larger components (change-focused)
* **CRP** → smaller components (dependency-focused)

### Figure 13.1 – Tension Triangle

* Each edge represents the **cost** of ignoring the opposite principle
* Over-emphasizing any two creates problems:

  * REP + CRP → too many components affected by changes
  * CCP + REP → too many unnecessary releases

---

## Finding Balance Over Time

Good architects:

* Choose a position inside the triangle
* Adjust as project priorities change

### Typical Evolution

* Early-stage projects:

  * Prioritize **CCP**
  * Developability > Reuse
* Mature projects:

  * Shift toward **REP**
  * Reuse becomes more important

Component structure is **dynamic**, not fixed.

---

## CONCLUSION

Component cohesion is **not simple**.

* It is not just “one component does one thing”
* It is a balance between:

  * Reusability
  * Developability
  * Deployment cost

This balance:

* Changes over time
* Evolves with project maturity
* Causes component boundaries to shift

Good architecture embraces this evolution rather than resisting it.

