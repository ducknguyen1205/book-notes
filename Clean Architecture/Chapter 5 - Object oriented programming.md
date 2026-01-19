
This chapter explains **what Object-Oriented Programming (OO) really means from a software architect’s perspective**. It argues that many common definitions of OO are misleading, and concludes that **the true power of OO is polymorphism, which enables dependency inversion and plugin architectures**.

---

## What Is OO?

Common definitions of OO are examined and rejected:

- **“The combination of data and function”** – Unsatisfactory, because calling `o.f()` versus `f(o)` is not fundamentally different.
    
- **“Modeling the real world”** – Vague and undefined, offering no concrete technical meaning.
    
- **Encapsulation, Inheritance, Polymorphism** – Often cited as the three pillars of OO. The chapter analyzes each one critically.
    

---

## ENCAPSULATION?

Encapsulation is the idea of drawing a boundary around data and functions so that internal details are hidden and only a public interface is exposed.

### Encapsulation in C (Perfect Encapsulation)

C allowed _perfect encapsulation_ through header (`.h`) and implementation (`.c`) files.

### Example (C)

```c
// point.h
struct Point;
struct Point* makePoint(double x, double y);
double distance (struct Point *p1, struct Point *p2);
```

```c
// point.c
#include "point.h"
#include <stdlib.h>
#include <math.h>
struct Point {
    double x,y;
};
struct Point* makepoint(double x, double y) {
    struct Point* p = malloc(sizeof(struct Point));
    p->x = x;
    p->y = y;
    return p;
}
double distance(struct Point* p1, struct Point* p2) {
    double dx = p1->x - p2->x;
    double dy = p1->y - p2->y;
    return sqrt(dx*dx+dy*dy);
}
```

Users of `point.h` cannot see the internal structure of `Point`. This is **true encapsulation**.

### Encapsulation in C++ (Weakened)

C++ requires member variables to be declared in header files, breaking full encapsulation.

```cpp
// point.h
class Point {
public:
    Point(double x, double y);
    double distance(const Point& p) const;
private:
    double x;
    double y;
};
```

Clients now **know that `x` and `y` exist**, even if they cannot access them directly. Changing these members forces recompilation.

Java and C# go further by **eliminating header/implementation separation entirely**, weakening encapsulation even more.

**Conclusion:** OO does _not_ provide stronger encapsulation than C. In many cases, it provides weaker encapsulation.

---

## INHERITANCE?

Inheritance is often seen as a core OO feature, but its underlying mechanism existed before OO languages.

### Inheritance-like Behavior in C

C programmers manually achieved inheritance by struct layout conventions.

### Example

```c
// namedPoint.h
struct NamedPoint;
struct NamedPoint* makeNamedPoint(double x, double y, char* name);
void setName(struct NamedPoint* np, char* name);
char* getName(struct NamedPoint* np);
```

```c
// namedPoint.c
#include "namedPoint.h"
#include <stdlib.h>
struct NamedPoint {
    double x,y;
    char* name;
};
struct NamedPoint* makeNamedPoint(double x, double y, char* name) {
    struct NamedPoint* p = malloc(sizeof(struct NamedPoint));
    p->x = x;
    p->y = y;
    p->name = name;
    return p;
}
void setName(struct NamedPoint* np, char* name) {
    np->name = name;
}
char* getName(struct NamedPoint* np) {
    return np->name;
}
```

```c
// main.c
#include "point.h"
#include "namedPoint.h"
#include <stdio.h>
int main(int ac, char** av) {
    struct NamedPoint* origin = makeNamedPoint(0.0, 0.0, "origin");
    struct NamedPoint* upperRight = makeNamedPoint(1.0, 1.0, "upperRight");
    printf("distance=%f\n",
        distance(
            (struct Point*) origin,
            (struct Point*) upperRight));
}
```

Here, `NamedPoint` **masquerades as `Point`** because its first fields match `Point` exactly.

This technique:

- Existed long before OO
    
- Is how C++ implements single inheritance internally
    
- Is inconvenient and unsafe compared to true OO inheritance

**Conclusion:** OO inheritance adds convenience and safety, but not a fundamentally new concept.

---

## POLYMORPHISM?

Polymorphism existed long before OO via **function pointers**.

### Example: Copy Program in C

```c
#include <stdio.h>
void copy() {
    int c;
    while ((c=getchar()) != EOF)
        putchar(c);
}
```

`getchar()` and `putchar()` behave differently depending on the underlying device.

### How Polymorphism Works in C

UNIX enforces a standard interface for IO devices:

```c
struct FILE {
    void (*open)(char* name, int mode);
    void (*close)();
    int (*read)();
    void (*write)(char);
    void (*seek)(long index, int mode);
};
```

```c
struct FILE console = {open, close, read, write, seek};
```

```c
extern struct FILE* STDIN;
int getchar() {
    return STDIN->read();
}
```

This is **polymorphism implemented with function pointers**.

### OO’s Contribution

OO did not invent polymorphism, but it:

- Makes polymorphism **safe**
    
- Eliminates manual conventions
    
- Prevents subtle and dangerous bugs

**Key Insight:** OO imposes discipline on indirect transfer of control.

---

## THE POWER OF POLYMORPHISM

Because polymorphism decouples callers from implementations:

- Programs do not depend on device details
    
- New implementations can be added without recompilation
    
- Components become **plugins**

This idea originated from the need for **device independence** in early systems (cards → tape → console, etc.).

OO enables **plugin architectures everywhere**, not just in operating systems.

---

## DEPENDENCY INVERSION

### Without Polymorphism

- Source code dependencies follow the flow of control
    
- High-level modules depend on low-level modules
    
![[Pasted image 20251228103731.png]]
**Figure 5.1:** Source code dependencies versus flow of control

### With Polymorphism

By inserting interfaces:

- Source code dependencies can be reversed
    
- Flow of control remains unchanged
    
![[Pasted image 20251228103817.png]]
**Figure 5.2:** Dependency inversion

This gives architects **complete control** over dependency direction.

### Architectural Impact

You can design systems where:

- UI depends on business rules
    
- Database depends on business rules
    
- Business rules depend on nothing
    
![[Pasted image 20251228103919.png]]
**Figure 5.3:** Database and UI depend on business rules

This enables:

- Plugin-based architecture
    
- Independent deployment
    
- Independent development by separate teams
    

---

## CONCLUSION

From an architectural perspective:

- Encapsulation: Not improved by OO
    
- Inheritance: Slight improvement in convenience
    
- Polymorphism: The real value of OO
    

**Final Definition:**

> Object-Oriented Programming is the ability, through safe and convenient polymorphism, to gain absolute control over source code dependencies.

This allows architects to build systems where:

- High-level policies are independent of low-level details
    
- Details become plugins
    
- Systems are flexible, deployable, and evolvable

# Notes

## High-level policies VS Low-level details VS plugins examples

### Example 1: Payment Processing (Classic, Easy to Visualize)

### High-level policy (business rule)

> “Charge a customer a given amount.”

This is a **business rule**. It should not care _how_ the payment is processed.

```ts
// UseCase / Business Rule
export interface PaymentGateway {
  charge(amount: number): void;
}

export class ProcessPayment {
  constructor(private gateway: PaymentGateway) {}

  execute(amount: number) {
    if (amount <= 0) {
      throw new Error("Invalid amount");
    }
    this.gateway.charge(amount);
  }
}
```

### Low-level details (plugins)

```ts
// Stripe implementation (detail)
export class StripeGateway implements PaymentGateway {
  charge(amount: number) {
    console.log(`Charging ${amount} via Stripe`);
  }
}
```

```ts
// PayPal implementation (detail)
export class PaypalGateway implements PaymentGateway {
  charge(amount: number) {
    console.log(`Charging ${amount} via PayPal`);
  }
}
```

### Why this matches the principle

- **High-level policy**: `ProcessPayment`
    
- **Low-level details**: `StripeGateway`, `PaypalGateway`
    
- `ProcessPayment`:
    
    - Does **not import Stripe**
        
    - Does **not know PayPal exists**
        
- Payment providers are **plugins** that can be swapped at runtime.
    

➡️ Stripe can be replaced with PayPal **without touching business rules**.

---

### Example 2: Database as a Plugin (Very Important in Clean Architecture)

### High-level policy

> “Store and retrieve users.”

```ts
// Business rule interface
export interface UserRepository {
  save(user: User): void;
  findById(id: string): User | null;
}
```

```ts
// Use case
export class RegisterUser {
  constructor(private repo: UserRepository) {}

  execute(user: User) {
    if (!user.email.includes("@")) {
      throw new Error("Invalid email");
    }
    this.repo.save(user);
  }
}
```

### Low-level details (plugins)

```ts
// MySQL implementation
export class MySQLUserRepository implements UserRepository {
  save(user: User) {
    console.log("Saving user to MySQL");
  }
  findById(id: string) {
    return null;
  }
}
```

```ts
// MongoDB implementation
export class MongoUserRepository implements UserRepository {
  save(user: User) {
    console.log("Saving user to MongoDB");
  }
  findById(id: string) {
    return null;
  }
}
```

### Why this matters architecturally

- The **business rule never imports MySQL or MongoDB**
    
- Databases are **details**
    
- Details are **replaceable plugins**
    

➡️ You can migrate from MySQL → MongoDB **without changing business logic**.

This is exactly what Figure 5.3 in the book is describing.

---

### Example 3: UI as a Plugin (Web, CLI, API)

### High-level policy

> “Create an order.”

```ts
export class CreateOrder {
  execute(productId: string, quantity: number) {
    if (quantity <= 0) {
      throw new Error("Invalid quantity");
    }
    return { productId, quantity };
  }
}
```

### Low-level details (plugins)

```ts
// REST Controller
app.post("/orders", (req, res) => {
  const useCase = new CreateOrder();
  const order = useCase.execute(req.body.productId, req.body.quantity);
  res.json(order);
});
```

```ts
// CLI
const useCase = new CreateOrder();
const order = useCase.execute("p1", 2);
console.log(order);
```

```ts
// GraphQL resolver
createOrder(_, args) {
  const useCase = new CreateOrder();
  return useCase.execute(args.productId, args.quantity);
}
```

### Key observation

- The **use case does not depend on HTTP, CLI, or GraphQL**
    
- UI technologies are **replaceable plugins**
    

➡️ The same business rule works everywhere.
### Mapping Back to Chapter 5 Terms

| Clean Architecture Term | In the Examples                                        |
| ----------------------- | ------------------------------------------------------ |
| High-level policy       | Use cases (`ProcessPayment`, `RegisterUser`)           |
| Low-level details       | Stripe, MySQL, S3, REST controllers                    |
| Polymorphism            | Interfaces (`PaymentGateway`, `UserRepository`)        |
| Dependency inversion    | Business depends on interface, not implementation      |
| Plugin architecture     | Details can be added/replaced without recompiling core |

### One-Sentence Summary (Architect View)

> High-level policies define **what the system does**, while low-level details define **how it is done**, and polymorphism allows the “how” to be plugged in without ever changing the “what”.

## "New implementations can be added without recompilation"
### What “without recompilation” actually means

It **does NOT** mean:

* “You never recompile anything ever again”

It **DOES** mean:

* You do **not recompile the high-level policy module**
* You only compile (or build) the **new low-level plugin**
* The existing business logic binary/artifact remains unchanged

This is exactly the property that **plugin architectures** provide.

---

### Historical Example (From the Book’s Logic)

### The C `copy` program

```c
void copy() {
    int c;
    while ((c=getchar()) != EOF)
        putchar(c);
}
```

When a **new IO device** (e.g., tape, console, printer) is added:

* The `copy` program:

  * Is **not recompiled**
  * Is **not modified**
* Only a **new device driver** is compiled and linked

Why?

* `copy` depends only on the **interface** (`read`, `write`)
* Devices conform to that interface

➡️ Devices are **runtime plugins**

---

### Modern Interpretation (OO Systems)

### Key condition

> **High-level modules depend only on abstractions.**

As long as:

* The abstraction remains unchanged
* The contract is honored

Then:

* New implementations can be added independently

---

### Example 1: Database Driver as a Plugin (Very Concrete)

### Business rules (compiled once)

```ts
export interface UserRepository {
  save(user: User): void;
}
```

```ts
export class RegisterUser {
  constructor(private repo: UserRepository) {}

  execute(user: User) {
    this.repo.save(user);
  }
}
```

This code is compiled into:

```
business-rules.jar
```

### New database appears later

```ts
// New implementation added later
export class PostgresUserRepository implements UserRepository {
  save(user: User) {
    console.log("Saving to PostgreSQL");
  }
}
```

This is compiled into:

```
postgres-repo.jar
```

### Result

* `business-rules.jar` → **NOT recompiled**
* New DB support works immediately
* Only wiring/configuration changes

➡️ This is exactly what the book means.

---

### Example 2: Runtime Plugin Loading (Most Literal Form)

### High-level policy (stable binary)

```java
public interface PaymentGateway {
    void charge(int amount);
}
```

```java
public class ProcessPayment {
    private final PaymentGateway gateway;

    public ProcessPayment(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public void execute(int amount) {
        gateway.charge(amount);
    }
}
```

Compiled into:

```
core-payment.jar
```

### New plugin delivered later

```java
public class CryptoPaymentGateway implements PaymentGateway {
    public void charge(int amount) {
        System.out.println("Charging crypto");
    }
}
```

Compiled into:

```
crypto-plugin.jar
```

### Deployment

```bash
java -cp core-payment.jar:crypto-plugin.jar Main
```

No recompilation of:

* `ProcessPayment`
* `PaymentGateway`

Only **new JAR added to classpath**.

---

### Example 3: Backend Service (Your Day-to-Day Reality)

### Service A (business logic)

* Already deployed
* Already tested
* Already compiled

```ts
export interface NotificationSender {
  send(message: string): void;
}
```

### Later: new channel requested

* Email exists
* SMS exists
* **Now Slack is needed**

```ts
export class SlackNotificationSender implements NotificationSender {
  send(message: string) {
    console.log("Sending Slack message");
  }
}
```

### What changes?

* New module/package is built
* Configuration points to new implementation

What does **not** change?

* Use cases
* Core logic
* Existing binaries

➡️ That is **zero recompilation of high-level policy**.

---

### Why This Was Impossible (or Painful) Without OO Polymorphism

Without polymorphism:

```ts
if (type === "mysql") { ... }
else if (type === "mongo") { ... }
```

* Adding a new implementation:

  * Requires changing this code
  * Forces recompilation
  * Risks regressions

With polymorphism:

```ts
repo.save(user);
```

* New behavior added externally
* No recompilation
* No modification

---

### Clean Architecture Interpretation

When Uncle Bob says:

> “New implementations can be added without recompilation”

He is really saying:

> **High-level policy binaries are closed for modification but open for extension through plugins.**

This is:

* **Dependency Inversion Principle**
* **Open/Closed Principle**
* **Plugin Architecture**

All at once.

---

### Practical Rule You Can Use

You have achieved this property **if and only if**:

* You can add a new database / UI / gateway
* Without changing
  * Use cases
  * Entities
  * Business rules
* And without recompiling their artifacts

If you **must recompile business logic**, then **dependency inversion is broken**.

---

### One-Line Architect Summary

> If adding a new feature requires recompiling business rules, then that feature is not a plugin.