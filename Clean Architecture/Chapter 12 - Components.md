# COMPONENTS

## What Is a Component?

**Components are the units of deployment.**

They are the **smallest entities that can be deployed independently** in a system.

Examples by platform:

* Java → `.jar`
* Ruby → `.gem`
* .NET → `.dll`
* Compiled languages → groups of binary files
* Interpreted languages → groups of source files

Regardless of technology, components are the **granularity of deployment**.

Components may be:

* Linked into a single executable
* Packaged into an archive (e.g., `.war`)
* Loaded dynamically as plugins (`.jar`, `.dll`, `.exe`)

A key property of **well-designed components**:

> They retain the ability to be **independently deployable** and therefore **independently developable**.

---

## A BRIEF HISTORY OF COMPONENTS

### Early Programming: Fixed Memory Locations

Early programs were **not relocatable**.

* Programmers explicitly chose memory addresses
* The `*200` directive specified where code would be loaded
* Libraries were included as **source code**, not binaries

#### Problems

* Memory was expensive and limited
* Compilers made multiple passes over source code
* Reading large source libraries from slow devices took hours

---

### Separating Libraries from Applications

To reduce compile time:

* Libraries were compiled separately
* Loaded at fixed addresses (e.g., 2000₈)
* Applications referenced them via symbol tables

**Memory layout issues followed:**

* Applications outgrew available address space
* Code had to jump around libraries
* Memory fragmentation worsened as programs grew

This approach was not sustainable.

---

## RELOCATABILITY

### The Breakthrough

**Relocatable binaries** solved the problem.

* Compiler emitted relocation metadata
* Loader adjusted memory addresses at load time
* Programs and libraries could be loaded anywhere

### Linking at Load Time

* External references and definitions were recorded
* Loader resolved references after loading
* Only required functions needed to be loaded

This led to the **linking loader**.

---

## LINKERS

As programs grew larger:

* Load-time linking became too slow
* Libraries resided on slow tapes and disks
* Linking could take over an hour

### Solution

* Separate **linking** from **loading**
* Introduce the **linker**
* Output: linked relocatable executable
* Loader only performed fast relocation

---

## The Compile–Link Bottleneck Returns

By the 1980s:

* Programs reached hundreds of thousands of lines
* `.c` → `.o` → linker → executable
* Compilation and linking again took hours

This led to the observation:

> **Programs grow to fill all available compile and link time.**

(Murphy’s Law of Program Size)

---

## Moore’s Law Changes Everything

From the late 1980s to mid-1990s:

* CPUs became dramatically faster
* Memory became cheap
* Disks became smaller and faster
* Caching became common

Result:

* Linking time dropped to seconds
* Load-time linking became practical again

---

## The Return of Load-Time Linking

This era introduced:

* Shared libraries
* ActiveX
* `.jar` files
* Plugin architectures

Systems could now:

* Link components at runtime
* Load plugins dynamically
* Extend behavior without rebuilding the whole system

---

## Modern Component Plugin Architecture

Today:

* Components are routinely shipped as plugins
* Examples:

  * Minecraft mods (`.jar`)
  * Visual Studio extensions (`.dll`)

Component-based architectures are now **the default**, not the exception.

---

## CONCLUSION

Modern software components:

* Are **dynamically linked**
* Can be **plugged together at runtime**
* Represent the architectural building blocks of systems

> After 50 years of evolution, **component plugin architecture** is now easy, fast, and commonplace.

