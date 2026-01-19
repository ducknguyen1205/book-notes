## Diplomatic Immunity

This chapter illustrates how ignoring software engineering best practices—especially for database and SQL work—creates **technical debt**, increasing risk, maintenance cost, and long-term failure. The story emphasizes that database code is just as critical as application code and must follow the same discipline.

---

## 24.1 Objective: Employ Best Practices

Professional software development requires:

- Source code control (e.g., Git, Subversion)
    
- Automated testing (unit or functional tests)
    
- Documentation, specifications, and code comments

Using best practices may take more time initially, but it **reduces rework, risk, and long-term cost**. Skipping them for speed almost always leads to failure.

---

## 24.2 Antipattern: Make SQL a Second-Class Citizen

This antipattern assumes that **database code is exempt from normal engineering rules**. Common reasons include:

- Separation of roles between developers and DBAs
    
- SQL being treated as a “special” or embedded language
    
- Poor tooling for database development
    
- Over-reliance on a single DBA as the knowledge holder

The result is a fragile system built on a database that is poorly understood and risky to change.

---

## 24.3 How to Recognize the Antipattern

Typical warning signs:

- “Lightweight” processes that skip key practices
    
- DBAs excluded from source control or tooling
    
- Unknown purpose of tables or columns
    
- Need for tools to compare and synchronize schemas
    

These indicate **missing documentation, poor change control, and unmanaged schema evolution**.

---

## 24.4 Legitimate Uses of the Antipattern

Truly **temporary or one-off SQL code** (e.g., ad-hoc queries) may not need full process overhead.

Rule of thumb:

- If you cannot delete the code immediately after use, it should be:
    
    - Stored in source control
        
    - Briefly documented
        

Temporary code that survives becomes real code and must be treated as such.

---

## 24.5 Solution: Establish a Big-Tent Culture of Quality

Quality assurance consists of:

1. Clearly written requirements
    
2. Proper design and implementation
    
3. Validation and testing against requirements
    

Databases must follow the **same QA lifecycle** as application code.

---

### Exhibit A: Documentation

There is **no such thing as self-documenting code**. Database documentation is essential because code alone cannot explain intent, missing features, or business rules.

Key documentation elements:

#### Entity-Relationship Diagrams

- Show tables and relationships
    
- May be split into multiple diagrams for large schemas
    
- Can be generated via reverse-engineering tools
    

#### Tables, Columns, and Views

- Purpose of each table
    
- Expected row counts and queries
    
- Column meanings, valid values, units, nullability, constraints
    
- Purpose and usage of views
    

#### Relationships

- Business rules behind foreign keys
    
- Explanation of nullable vs non-nullable relationships
    
- Documentation of implicit relationships without constraints
    

#### Triggers

- Business rules enforced
    
- Validation, transformation, logging behavior
    

#### Stored Procedures

- Purpose and side effects
    
- Input/output parameters
    
- Security and performance considerations
    

#### SQL Security

- Users, roles, privileges
    
- Access control strategies
    
- SQL Injection prevention
    
- Network and authentication security
    

#### Database Infrastructure

- RDBMS brand/version
    
- Server topology and replication
    
- Backup policies and connection details
    

#### Object-Relational Mapping

- Business rules implemented in ORM layers
    
- Validation, transformation, caching, and logging logic
    

Despite being difficult, **database documentation is more critical than any other documentation**.

---

### Trail of Evidence: Source Code Control

Database code must be versioned like application code.

Include in source control:

- Data definition scripts (`CREATE TABLE`, etc.)
    
- Triggers and stored procedures
    
- Bootstrap (seed) data
    
- ER diagrams and documentation
    
- DBA operational scripts (backup, reporting, validation)
    

Database and application code **must live in the same repository** to ensure compatibility across revisions.

---

### Schema Evolution Tools

Database schema changes should be automated using migration tools.

**Example (Rails migration – preserved verbatim):**

```ruby
class AddHoursToBugs < ActiveRecord::Migration
def self.up
add_column :bugs, :hours, :decimal
end
def self.down
remove_column :bugs, :hours
end
end
```

Migrations:

- Apply incremental upgrades and downgrades
    
- Track schema versions in the database
    
- Reduce manual synchronization errors
    

Similar tools exist in Django, Doctrine, ASP.NET, and others.

---

### Burden of Proof: Testing

Databases should be tested **independently of application code**, using isolation principles.

**Preserved PHPUnit test example:**

```php
<?php
require_once "PHPUnit/Framework/TestCase.php";
class DatabaseTest extends PHPUnit_Framework_TestCase
{
protected $pdo;
public function setUp()
{
$this->pdo = new PDO("mysql:dbname=bugs", "testuser", "xxxxxx");
}
public function testTableFooExists()
{
$stmt = $this->pdo->query("SELECT COUNT(*) FROM Bugs");
$err = $this->pdo->errorInfo();
$this->assertType("object", $stmt, $err[2]);
$this->assertEquals("PDOStatement", get_class($stmt));
}
public function testTableFooColumnBugIdExists()
{
$stmt = $this->pdo->query("SELECT COUNT(bug_id) FROM Bugs");
$err = $this->pdo->errorInfo();
$this->assertType("object", $stmt, $err[2]);
$this->assertEquals("PDOStatement", get_class($stmt));
}
static public function main()
{
$suite = new PHPUnit_Framework_TestSuite(__CLASS__);
$result = PHPUnit_TextUI_TestRunner::run($suite);
}
}
DatabaseTest::main();
```

Database testing checklist:

- Tables, columns, and views existence
    
- Constraints via negative tests
    
- Trigger side effects
    
- Stored procedure logic and side effects
    
- Bootstrap data presence
    
- Query correctness
    
- ORM behavior and validation
    

Failed tests often indicate **connection to the wrong database instance**—always verify configuration first.

---

### Case Load: Working in Multiple Branches

Best practices:

- Separate database instances per environment and developer
    
- Configurable database connections per application revision
    
- Use virtualization to replicate production environments
    

There is no technical excuse for skipping database best practices today.

---

## Final Takeaway

**Database code is first-class code.**  
Apply the same rigor—documentation, source control, migrations, and testing—to SQL as you do to application code. Ignoring this creates invisible technical debt that eventually becomes unavoidable.