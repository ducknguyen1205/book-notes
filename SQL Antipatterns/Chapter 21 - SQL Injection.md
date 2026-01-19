## Background: Why SQL Injection Matters

The chapter opens with a real-world example of **Albert Gonzalez**, whose massive data thefts were enabled by **SQL Injection**, one of the most common and dangerous web vulnerabilities.  
The key lesson: SQL Injection is not exotic hacking—it exploits **ordinary programming mistakes**.

---

## 21.1 Objective: Write Dynamic SQL Queries

Dynamic SQL is created when application variables are interpolated into SQL strings.

```php
<?php
$sql = "SELECT * FROM Bugs WHERE bug_id = $bug_id";
$stmt = $pdo->query($sql);
```

**Key idea**:

- Dynamic SQL is powerful and normal.
    
- The real challenge is **security**, not functionality.
    
- SQL Injection happens when software allows **unintended behavior**, not intended queries.
    

---

## 21.2 Antipattern: Execute Unverified Input As Code

### What Is SQL Injection?

SQL Injection occurs when user input **changes the syntax** of an SQL statement before it is parsed.

```sql
SELECT * FROM Bugs WHERE bug_id = 1234; DELETE FROM Bugs
```

The database executes **both statements**, not just the intended one.

---

### Accidents May Happen

Example of accidental injection due to an apostrophe:

```php
<?php
$project_name = $_REQUEST["name"];
$sql = "SELECT * FROM Projects WHERE project_name = '$project_name'";
```

```sql
SELECT * FROM Projects WHERE project_name = 'O'Hare'
```

**Lesson**:

- Not all SQL Injection is malicious.
    
- Syntax errors are safer than valid but unintended queries.
    

---

### The Top Web Security Threat

Password update example:

```php
<?php
$password = $_REQUEST["password"];
$userid = $_REQUEST["userid"];
$sql = "UPDATE Accounts SET password_hash = SHA2('$password')
WHERE account_id = $userid";
```

Attack input:

```
userid=123 OR TRUE
```

Resulting SQL:

```sql
UPDATE Accounts SET password_hash = SHA2('xyzzy')
WHERE account_id = 123 OR TRUE;
```

**Key insight**:

> SQL Injection works because the **query syntax is altered before parsing**.

---

## The Quest for a Cure

No single technique fully prevents SQL Injection.  
**Defense must be layered**.

---

## Escaping Values

Escaping prevents quotes from terminating strings.

```sql
SELECT * FROM Projects WHERE project_name = 'O''Hare'
```

```sql
SELECT * FROM Projects WHERE project_name = 'O\'Hare'
```

Using PDO:

```php
<?php
$project_name = $pdo->quote($_REQUEST["name"]);
$sql = "SELECT * FROM Projects WHERE project_name = $project_name";
```

**Limitations**:

- Works best for strings.
    
- Fails for numeric values and edge cases.
    
- Can be bypassed in rare character encoding scenarios.
    

---

## Query Parameters

Preferred defense technique.

```php
<?php
$stmt = $pdo->prepare("SELECT * FROM Projects WHERE project_name = ?");
$params = array($_REQUEST["name"]);
$stmt->execute($params);
```

**Why it works**:

- SQL is parsed **before** values are supplied.
    
- Parameters are always treated as **literal values**.
    

### What Parameters Cannot Do

They **cannot** replace:

- Lists of values
    
- Table names
    
- Column names
    
- SQL keywords
    

Examples that **do not work**:

```php
$stmt = $pdo->prepare("SELECT * FROM Bugs WHERE bug_id IN ( ? )");
```

```sql
SELECT * FROM Bugs WHERE bug_id IN ( '1234,3456,5678' )
```

```php
$stmt = $pdo->prepare("SELECT * FROM ? WHERE bug_id = 1234");
```

```sql
SELECT * FROM 'Bugs' WHERE bug_id = 1234
```

---

## What Was My Complete Query?

Important clarification:

- Prepared SQL **never contains parameter values**
    
- SQL and values are stored separately
    
- Best debugging approach: **log query + parameters separately**
    

---

## Stored Procedures

Stored procedures are **not automatically safe**.

Unsafe example:

```sql
CREATE PROCEDURE UpdatePassword(input_password VARCHAR(20),
input_userid VARCHAR(20))
BEGIN
SET @sql = CONCAT('UPDATE Accounts
SET password_hash = SHA2(', QUOTE(input_password), ')
WHERE account_id = ', input_userid);
PREPARE stmt FROM @sql;
EXECUTE stmt;
END
```

**Key point**:

- Dynamic SQL inside stored procedures has the **same risks** as application code.
    

---

## Data Access Frameworks

Frameworks **do not prevent SQL Injection automatically**.

Analogy:

> A framework prevents SQL Injection like a toothbrush prevents cavities—you must use it correctly and consistently.

---

## 21.3 How to Recognize the Antipattern

If SQL statements are built using:

- String concatenation
    
- Variable interpolation
    

→ **Assume SQL Injection risk exists**

### Rule #31: Check the Back Seat

Even data fetched safely from the database can later be used **unsafely** in dynamic SQL.

```php
$sql2 = "SELECT * FROM Bugs WHERE MATCH(description) AGAINST ('"
. $row["last_name"] . "')";
```

---

## 21.4 Legitimate Uses of the Antipattern

There are **no legitimate uses** of SQL Injection vulnerabilities.

Security is always the developer’s responsibility.

---

## 21.5 Solution: Trust No One

This section is the **core of the chapter**.

---

### Filter Input

Remove invalid characters **before** using input.

```php
<?php
$bugid = filter_input(INPUT_GET, "bugid", FILTER_SANITIZE_NUMBER_INT);
```

```php
<?php
$bugid = intval($_GET["bugid"]);
```

```php
<?php
if (preg_match("/[_[:alnum:]]+/", $_GET["order"], $matches)) {
$sortorder = $matches[1];
}
```

**Principle**:

- Accept only what is valid
    
- Reject everything else
    

---

### Parameterize Dynamic Values

Best solution for values.

```php
<?php
$sql = "UPDATE Accounts SET password_hash = SHA2(?) WHERE account_id = ?";
$stmt = $pdo->prepare($sql);
$stmt->execute(array($_REQUEST["password"], $_REQUEST["userid"]));
```

Malicious input becomes harmless:

```sql
UPDATE Accounts SET password_hash = SHA2('xyzzy')
WHERE account_id = '123 OR TRUE'
```

---

### Quoting Dynamic Values (Rare Cases)

Used only when parameters cause optimizer issues.

```php
<?php
$quoted_active = $pdo->quote($_REQUEST["active"]);
$sql = "SELECT * FROM Accounts WHERE is_active = {$quoted_active}";
```

**Guideline**:

- Use mature, well-tested quoting functions only
    
- Never write your own quoting logic
    

---

### Parameterizing an IN() Predicate

You must generate placeholders dynamically.

```php
$sql = "SELECT * FROM Bugs WHERE bug_id IN ("
. join(",", array_fill(0, count($bug_list), "?")) . ")";
$stmt = $pdo->prepare($sql);
$stmt->execute($bug_list);
```

---

### Isolate User Input from Code

Identifiers and keywords cannot be parameterized.  
Use **mapping arrays** instead.

```php
$sortorders = array( "status" => "status", "date" => "date_reported" );
$directions = array( "up" => "ASC", "down" => "DESC" );
```

Only predefined values are allowed into SQL:

```php
$sql = "SELECT * FROM Bugs ORDER BY {$sortorder} {$direction}";
```

**Benefits**:

- No user input touches SQL directly
    
- Works for columns, keywords, and expressions
    
- Decouples UI from database internals
    

---

### Get a Buddy to Review Your Code

Code reviews are the **most effective** way to find SQL Injection.

Inspection checklist:

1. Find dynamic SQL
    
2. Trace all input sources
    
3. Treat all external data as unsafe
    
4. Use parameters or robust escaping
    
5. Inspect stored procedures too
    