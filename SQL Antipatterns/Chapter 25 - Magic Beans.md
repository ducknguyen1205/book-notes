## 25.1 Objective: Simplify Models in MVC

Frameworks and design patterns exist to **reduce development time**, which is the biggest cost in software projects.
MVC helps by separating concerns:

* **Controller**: handles user input and delegates work
* **View**: presents output
* **Model**: contains business logic and database interaction

Controllers and views are easy to understand, but **models are often misunderstood**. Developers frequently oversimplify models by treating them as *only* database access objects.

---

## 25.2 Antipattern: The Model Is an Active Record

In simple cases, mapping a class directly to a database table works well. This is the **Active Record** pattern: a class represents one table and provides CRUD operations.

**Example (preserved):**

```php
$bugsTable = Doctrine_Core::getTable('Bugs');
$bugsTable->find(1234);
$bug = new Bugs();
$bug->summary = "Crashes when I save";
$bug->save();
```

The antipattern occurs when **all models are forced to be Active Records**, regardless of application complexity. This is a form of the **Golden Hammer** antipattern.

---

### Leaky Abstractions

Active Record is a **leaky abstraction**:

* Simple CRUD feels magical
* Complex SQL (JOIN, GROUP BY, full-text search) becomes awkward
* Frameworks add more SQL-like features, exposing the abstraction’s internals

At that point, using SQL directly may be clearer and safer.

---

### Active Record Couples Models to the Schema

Because each Active Record maps to a single table:

* Schema changes force changes to model classes and controllers
* Queries are duplicated across controllers
* Refactoring becomes expensive

---

### Active Record Exposes CRUD Functions

Active Record allows developers to bypass business rules.

**Intended behavior (email notification):**

```php
class CustomBugs extends BaseBugs
{
public function assignUser(Accounts $a)
{
$this->assigned_to = $a->account_id;
$this->save();
mail($a->email, "Assigned bug",
"You are now responsible for bug #{$this->bug_id}.");
}
}
```

**Bypassed behavior (no email):**

```php
$bugsTable = Doctrine_Core::getTable('Bugs');
$bugsTable->find(1234);
$bug->assigned_to = $user->account_id;
$bug->save();
```

Because CRUD methods are public, **business invariants can be violated**.

---

### Active Record Encourages an Anemic Domain Model

Models often contain **no real behavior**, only CRUD operations.
Business logic spreads across controllers, lowering cohesion.

**Example (procedural logic spread across controllers – preserved):**

```php
class AdminController extends Zend_Controller_Action
{
public function assignAction()
{
$bugsTable = Doctrine_Core::getTable("Bugs");
$bug = $bugsTable->find($_POST["bug_id"]);
$bug->Products[] = $_POST["product_id"];
$bug->assigned_to = $_POST["user_assigned_to"];
$bug->save();
}
}
```

This leads to:

* Code duplication
* Tight coupling
* Complex, tangled class relationships (Figure 25.2)

---

### Unit Testing Magic Beans Is Hard

Using Active Record as the model makes testing difficult:

* **Model tests** require a real database and fixtures
* **View tests** require HTML rendering and parsing
* **Controller tests** require fake HTTP requests and response inspection

This violates the isolation principle of unit testing.

---

## 25.3 How to Recognize the Antipattern

Warning signs:

* Passing SQL queries into models
* Copying complex queries across controllers
* Heavy reliance on database fixtures to test models

These indicate **models are really DAOs**, not domain models.

---

## 25.4 Legitimate Uses of the Antipattern

Active Record is valid when:

* Performing simple CRUD on single rows
* Building prototypes quickly

However, technical debt must be **paid back later via refactoring**.

---

## 25.5 Solution: The Model Has an Active Record

The key idea:

> **Models should use Active Record — not be Active Record**

Models represent **business concepts**, not tables.

---

### Grasping the Model (GRASP Principles)

#### Information Expert

* Business operations often involve multiple tables
* A model should aggregate multiple DAOs
* Relationship should be **HAS-A**, not **IS-A**

#### Creator

* Models should create and manage their own DAOs
* Controllers should not know how data is persisted

#### Low Coupling

* Database structure changes should affect one place only
* Complexity is unavoidable, but it must be well-contained

#### High Cohesion

* Model APIs should reflect **business intent**, not CRUD
* Prefer `assignUser()` over `save()`

---

### Putting the Domain Model into Action

A **domain model** encapsulates:

* Business rules
* Data access (as an internal detail)

**Refactored example (preserved):**

```php
class BugReport
{
protected $bugsTable;
protected $accountsTable;
protected $productsTable;
public function __construct()
{
$this->bugsTable = Doctrine_Core::getTable("Bugs");
$this->accountsTable = Doctrine_Core::getTable("Accounts");
$this->productsTable = Doctrine_Core::getTable("Products");
}
public function create($summary, $description, $reportedBy)
{
$bug = new Bugs();
$bug->summary = $summary
$bug->description = $description
$bug->status = "NEW";
$bug->reported_by = $reportedBy;
$bug->save();
}
public function assignUser($bugId, $assignedTo)
{
$bug = $bugsTable->find($bugId);
$bug->assigned_to = $assignedTo"];
$bug->save();
}
```

Controllers now call **business methods**, not database operations.

---

### Benefits of This Design

* Simpler class relationships (Figure 25.3)
* Controllers contain less logic
* Database queries are hidden inside models
* SQL can be used safely when necessary
* Better reuse and maintainability

---

### Testing Plain Objects

With models decoupled from DAOs:

* Models can be unit-tested without a database
* DAOs can be mocked or stubbed
* Controller tests become simpler and fewer

This leads to **faster, more reliable testing** and easier debugging.

---

## Final Takeaway

**Decouple your models from your tables.**

Active Record is a useful tool, but it should not define your domain.
Well-designed domain models improve:

* Flexibility
* Testability
* Maintainability
* Long-term development speed

This is how MVC delivers on its original promise.
