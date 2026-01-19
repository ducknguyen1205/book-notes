# THE LISKOV SUBSTITUTION PRINCIPLE (LSP)

In 1988, **Barbara Liskov** defined subtypes with the following substitution property:

> If for each object `o1` of type **S** there exists an object `o2` of type **T** such that, for all programs **P** defined in terms of **T**, the behavior of **P** is unchanged when `o1` is substituted for `o2`, then **S** is a subtype of **T**.

This idea is known as the **Liskov Substitution Principle (LSP)**.

In short:

> **Subtypes must be substitutable for their base types without altering program behavior.**

---

## GUIDING THE USE OF INHERITANCE

### License Example

* Base class: `License`
* Subtypes:

  * `PersonalLicense`
  * `BusinessLicense`
* Method: `calcFee()`

The **Billing application** uses `License` without knowing which subtype it receives.

### Why This Conforms to LSP

* Each subtype uses a different fee calculation algorithm
* The Billing system’s behavior does **not change** depending on the subtype
* All subtypes are safely substitutable for `License`

---

## THE SQUARE / RECTANGLE PROBLEM

### Description

* `Rectangle`: width and height can be changed independently
* `Square`: width and height must always be equal

### Why It Violates LSP

User code expects a `Rectangle` and does the following:

```java
Rectangle r = ...
r.setW(5);
r.setH(2);
assert(r.area() == 10);
```

If `r` is actually a `Square`:

* Setting width changes height
* The assertion fails

### Consequence

* The User must add logic to detect whether the object is a `Square`
* Behavior now depends on the concrete type
* The subtype is **not substitutable**

This is a direct violation of LSP.

---

## LSP AND ARCHITECTURE

LSP is **not limited to inheritance**.

It applies to:

* Interfaces and implementations
* Classes sharing method signatures
* REST services exposing the same endpoints
* Any situation where clients rely on substitutability

Architecturally, LSP violations are dangerous because they **force special-case handling** into the system.

---

## EXAMPLE LSP VIOLATION

### Taxi Dispatch Aggregator

* System aggregates multiple taxi companies
* Dispatch is done via a **REST interface**
* Dispatch URI is stored per driver
* System appends:

  * pickupAddress
  * pickupTime
  * destination

All taxi companies are expected to support the **same REST contract**.

---

### The Violation

* One company (Acme) uses `dest` instead of `destination`
* Their service does **not conform to the expected interface**

### Immediate Impact

The system must add special-case logic:

```java
if (driver.getDispatchUri().startsWith("acme.com")) ...
```

### Why This Is Architecturally Dangerous

* Hard-coded vendor knowledge in core logic
* Security risks
* Fragile system
* Explodes as more exceptions appear (e.g., mergers, rebranding)

---

## Architectural Fallout

To fix the violation, the architect must introduce:

* A dispatch command creation module
* Configuration-driven behavior based on URI patterns
* Additional complexity that **would not exist** if LSP were respected

Example configuration:

```
URI        Dispatch Format
acme.com   /pickupAddress/%s/pickupTime/%s/dest/%s
*          /pickupAddress/%s/pickupTime/%s/destination/%s
```

This mechanism exists **only because substitutability was broken**.

---

## CONCLUSION

* LSP is fundamentally about **substitutability**
* Violations force systems to:

  * Add conditionals
  * Introduce configuration tables
  * Increase architectural complexity

> **Even small LSP violations can pollute an entire architecture with defensive mechanisms.**

For this reason, LSP must be enforced **not only in class design**, but also at the **architectural level**.