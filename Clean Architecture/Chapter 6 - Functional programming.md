## Functional Programming

In many ways, the concepts of functional programming predate programming itself. This paradigm is strongly based on the λ-calculus invented by Alonzo Church in the 1930s.

---

## SQUARES OF INTEGERS

To explain what functional programming is, the chapter compares **imperative** and **functional** solutions to a simple problem: printing the squares of the first 25 integers.

### Java (Imperative Style)

```java
public class Squint {
    public static void main(String args[]) {
        for (int i=0; i<25; i++)
            System.out.println(i*i);
    }
}
```

- Uses a **mutable variable** (`i`)
    
- The variable changes state during execution
    

---

### Clojure (Functional Style)

```clojure
(println (take 25 (map (fn [x] (* x x)) (range))))
```

Reformatted with comments:

```clojure
(println ;
___________________
(take 25 ;
_________________
(map (fn [x] (* x x)) ;
__
(range)))) ;
Print
the first 25
squares
of Integers
___________
```

Key observations:

- `println`, `take`, `map`, and `range` are all **functions**
    
- Functions are called using parentheses
    
- `(fn [x] (* x x))` is an **anonymous function** that computes a square
    

Execution flow (inside-out):

- `range` returns a never-ending list starting from 0
    
- `map` applies the squaring function to each element
    
- `take` selects the first 25 results
    
- `println` prints them
    

Important note:

- **Never-ending lists are lazily evaluated**
    
- Only the required elements are actually created
    

---

### Key Insight

> Variables in functional languages do not vary.

- Clojure variables are initialized once and never modified
- This contrasts with Java’s mutable loop variable

---

## IMMUTABILITY AND ARCHITECTURE

Why does immutability matter to architects?

Because:

- **All race conditions**
    
- **All deadlocks**
    
- **All concurrent update problems**
    

…are caused by **mutable variables**.

If variables never change:

- Race conditions cannot occur
    
- Deadlocks cannot exist
    
- Concurrent update problems disappear
    

Architectural implication:

- Concurrency is one of the hardest problems in system design
    
- Immutability dramatically reduces concurrency risk
    

### Is immutability practical?

- Yes, with **infinite storage and infinite CPU**
    
- In reality, compromises are required
    

---

## SEGREGATION OF MUTABILITY

A common compromise is **separating mutable and immutable components**.

- Immutable components:
    
    - Purely functional
        
    - No state changes
        
- Mutable components:
    
    - Allow controlled state mutation
        
    - Handle unavoidable state changes
        
![[Pasted image 20260108223701.png]]
### Figure 6.1 — Mutating state and transactional memory

- Mutable components are protected using **transactional memory**
    
- Memory is treated like a database
    
- Changes occur inside transactions
    

---

### Clojure Atom Example

```clojure
(def counter (atom 0)) ; initialize counter to 0
(swap! counter inc) ; safely increment counter.
```

Explanation:

- `atom` allows controlled mutation
    
- `swap!`:
    
    - Reads the current value
        
    - Computes a new value
        
    - Uses **compare-and-swap**
        
    - Retries if a conflict occurs
        

Limitations:

- Suitable only for simple cases
    
- Not sufficient for complex dependencies
    
- More advanced mechanisms are required for larger systems
    

Architectural guidance:

- Push **as much logic as possible** into immutable components
    
- Minimize the scope of mutation
    
- Strictly discipline mutable areas
    

---

## EVENT SOURCING

As storage and CPU become cheaper and faster:

- Mutable state becomes less necessary

### Traditional Approach (Mutable State)

- Store account balances
- Update balances on each transaction

### Event Sourcing Approach (Immutable State)

- Store **only transactions**
- Recompute state by replaying events

Example:

- To get an account balance:
    
    - Sum all transactions from the beginning
        

Challenges:

- Storage grows without bound
    
- Recomputing from day one is expensive
    

Practical optimizations:

- Periodically snapshot state (e.g., nightly)
    
- Replay only recent events
    

Key consequences:

- Data is **append-only**
    
- No updates
    
- No deletes
    
- Systems become **CR**, not CRUD
    
- **Concurrent update problems disappear**
    

Important observation:

- This is how **source control systems** already work
    

---

## CONCLUSION

Summary of programming paradigms:

- **Structured programming**  
    → Discipline imposed on **direct transfer of control**
    
- **Object-oriented programming**  
    → Discipline imposed on **indirect transfer of control**
    
- **Functional programming**  
    → Discipline imposed on **variable assignment**
    

Key takeaway:

- Each paradigm **removes dangerous freedoms**
    
- None of them increases expressive power
    
- They teach us **what not to do**
    

Final reflection:

- Software principles have not fundamentally changed since 1946
    
- Hardware and tools evolve, but software rules remain the same
    

At its core, software consists of:

- Sequence
    
- Selection
    
- Iteration
    
- Indirection
    

> Nothing more. Nothing less.

# Notes

## Sequence - Selection - Iteration - Indirection
### Example: Processing an Order Total

#### 1. Sequence

Operations executed **step by step in a fixed order**.

```java
double subtotal = calculateSubtotal(items);
double tax = calculateTax(subtotal);
double total = subtotal + tax;
```

- Each statement runs after the previous one
    
- Changing the order changes the result
    

---

#### 2. Selection

Choosing **one path or another** based on a condition.

```java
if (customer.isVip()) {
    total = total * 0.9; // apply discount
}
```

- Only one branch executes
    
- Decision is made at runtime
    

---

#### 3. Iteration

Repeating an operation over a collection or range.

```java
double calculateSubtotal(List<Item> items) {
    double sum = 0;
    for (Item item : items) {
        sum += item.getPrice();
    }
    return sum;
}
```

- Same logic applied multiple times
    
- Controlled repetition
    

---

#### 4. Indirection

Accessing behavior **through an abstraction**, not directly.

```java
interface TaxPolicy {
    double calculate(double amount);
}

class StandardTaxPolicy implements TaxPolicy {
    public double calculate(double amount) {
        return amount * 0.1;
    }
}
```

```java
TaxPolicy taxPolicy = policyFactory.getPolicy();
double tax = taxPolicy.calculate(subtotal);
```

- Caller does not know the concrete implementation
    
- Enables substitution, decoupling, and extensibility
    

---

### Why This Matters in Clean Architecture

- **Sequence, selection, and iteration** describe _what_ software does
    
- **Indirection** determines _how well_ software is structured
    
- Clean Architecture is largely about **controlling indirection** so that:
    
    - High-level policies do not depend on low-level details
        
    - Changes remain localized
