# THE DEPENDENCY INVERSION PRINCIPLE (DIP)

The **Dependency Inversion Principle (DIP)** states:

> **The most flexible systems are those in which source code dependencies refer only to abstractions, not to concretions.**

---

## DIP AND PROGRAMMING LANGUAGES

### Statically Typed Languages (e.g., Java)

* `import`, `use`, and `include` statements should reference:

  * Interfaces
  * Abstract classes
  * Abstract declarations
* Source code should **not depend on concrete implementations**

### Dynamically Typed Languages (e.g., Python, Ruby)

* The same rule applies
* A *concrete module* is one where the called functions are implemented
* Even without explicit imports, dependencies still exist

---

## Realism and Stable Concretes

Treating DIP as an absolute rule is unrealistic.

### Example: `String` in Java

* `String` is concrete
* It is extremely stable
* Changes are rare and controlled

Therefore:

* Dependencies on **stable platform facilities** (OS, runtime, standard libraries) are acceptable
* Dependencies on **volatile concretes** (actively developed code) are the real problem

---

## STABLE ABSTRACTIONS

### Key Observation

* Changing an interface forces changes to implementations
* Changing an implementation usually does **not** force interface changes
* Interfaces are therefore **less volatile**

### Architectural Implication

> **Stable architectures depend on stable abstractions and avoid volatile concretions.**

---

## Practical Rules Derived from DIP

1. **Don’t refer to volatile concrete classes**
   Depend on interfaces instead
   Often enforces use of **Abstract Factories**

2. **Don’t derive from volatile concrete classes**
   Inheritance creates rigid dependencies
   Use with extreme care

3. **Don’t override concrete functions**
   Overriding inherits dependencies
   Prefer abstract functions with multiple implementations

4. **Never mention the name of anything concrete and volatile**
   This is the principle distilled into a single rule

---

## FACTORIES

### The Object Creation Problem

* Creating an object usually requires referencing a concrete class
* This creates an unavoidable dependency

### Solution: Abstract Factory Pattern
![[Pasted image 20260116214855.png]]
**Figure 11.1 – Abstract Factory Structure**

* `Application` depends on `Service` (interface)
* `ServiceFactory` is an interface
* `ServiceFactoryImpl` implements `ServiceFactory`
* `ServiceFactoryImpl` creates `ConcreteImpl`
* `ConcreteImpl` implements `Service`

### Result

* Application never depends on `ConcreteImpl`
* Object creation is isolated in the concrete component

---

## ARCHITECTURAL BOUNDARY

The curved line in Figure 11.1 represents an **architectural boundary**.

* One side: **Abstract component**

  * High-level business rules
* Other side: **Concrete component**

  * Implementation details

### Dependency Direction

* **Source code dependencies** cross the boundary toward abstractions
* **Flow of control** crosses in the opposite direction

#### Explain 
- **Source code dependency** = _who knows about whom at compile time_
- **Flow of control** = _who calls whom at runtime_

This inversion is why the principle is called **Dependency Inversion**.

---

## CONCRETE COMPONENTS

* Some DIP violations are unavoidable
* These violations should be:

  * Isolated
  * Grouped into a small number of concrete components

### Example: `main`

* Typically contains:

  * Object creation
  * Wiring
* Often the only place where concretes are referenced directly

In Figure 11.1:

* `main` instantiates `ServiceFactoryImpl`
* Stores it as a `ServiceFactory`
* Application accesses it only through the abstraction

---

## CONCLUSION

* DIP is the **most visible organizing principle** in Clean Architecture
* Architectural boundaries emerge from DIP
* Dependency direction becomes a strict rule

In later chapters, this idea evolves into the **Dependency Rule**:

> **Source code dependencies must always point toward more abstract, higher-level policies.**

