**Introduction**  
A common issue occurs when using wildcards in joins. A programmer encountered an error where book titles appeared null because both the `Books` and `Authors` tables had a column named `title`. The application overwrote the book title with the author’s title (which was often empty).

The code causing the issue:

```sql
SELECT * FROM Books b JOIN Authors a ON (b.author_id = a.author_id);
```

The fix involves using aliases to distinguish columns:

```sql
SELECT b.title, a.title AS salutation
FROM Books b JOIN Authors a ON (b.author_id = a.author_id);
```

### 19.1 Objective: Reduce Typing

Developers often use wildcards to avoid typing out long lists of columns for queries or inserts.

Instead of typing this:

```sql
SELECT bug_id, date_reported, summary, description, resolution,
reported_by, assigned_to, verified_by, status, priority, hours
FROM Bugs;
```

They prefer the implicit method:

```sql
SELECT * FROM Bugs;
```

Similarly, for inserting data, the explicit method is longer:

```sql
INSERT INTO Accounts (account_name, first_name, last_name, email,
password, portrait_image, hourly_rate) VALUES
('bkarwin', 'Bill', 'Karwin', 'bill@example.com', SHA2('xyzzy'), NULL, 49.95);
```

So developers use the shorter implicit version:

```sql
INSERT INTO Accounts VALUES (DEFAULT,
'bkarwin', 'Bill', 'Karwin', 'bill@example.com', SHA2('xyzzy'), NULL, 49.95);
```

### 19.2 Antipattern: a Shortcut That Gets You Lost

Using implicit columns creates maintenance hazards. If the database schema changes, the code breaks.

**Breaking Refactoring**  
If a new column is added to the table:

```sql
ALTER TABLE Bugs ADD COLUMN date_due DATE;
```

The previous `INSERT` statement will fail because the number of values no longer matches the number of columns:

```sql
INSERT INTO Bugs VALUES (DEFAULT, CURDATE(), 'New bug', 'Test T987 fails...',
NULL, 123, NULL, NULL, DEFAULT, 'Medium', NULL);
-- SQLSTATE 21S01: Column count doesn't match value count at row 1
```

Additionally, if code relies on column position (ordinals) rather than names:

```php
<?php
$stmt = $pdo->query("SELECT * FROM Bugs WHERE bug_id = 1234");
$row = $stmt->fetch();
$hours = $row[10];
```

And a column is dropped:

```sql
ALTER TABLE Bugs DROP COLUMN verified_by;
```

The application will read the wrong data because the `hours` column is no longer at position 10.

**Hidden Costs**  
Fetching all columns (`SELECT *`) wastes network bandwidth and memory, especially if the query returns large amounts of data (like TEXT fields) that the application doesn't actually use.

### 19.3 How to Recognize the Antipattern

You are likely facing this issue if:

- Application code breaks after simple database changes (like adding or removing columns).
    
- Network traffic is excessively high because queries are fetching more data than is displayed.
    

### 19.4 Legitimate Uses of the Antipattern

Wildcards are acceptable for ad hoc queries, testing, or debugging.

In some join scenarios, it is acceptable to use a wildcard for one specific table while listing specific columns from another to save time:

```sql
SELECT b.*, a.first_name, a.email
FROM Bugs b JOIN Accounts a
ON (b.reported_by = a.account_id);
```

### 19.5 Solution: Name Columns Explicitly

The best practice is to always list the specific columns you need. This decouples your application from the physical storage order and count of columns in the database.

**Mistake-Proofing**  
Explicitly naming columns protects your code. If columns are reordered or added, your query still works. If a column is dropped, you get a clear error message helping you fix the specific line of code.

Explicit `SELECT`:

```sql
SELECT bug_id, date_reported, summary, description, resolution,
reported_by, assigned_to, verified_by, status, priority, hours
FROM Bugs;
```

Explicit `INSERT`:

```sql
INSERT INTO Accounts (account_name, first_name, last_name, email,
password_hash, portrait_image, hourly_rate)
VALUES ('bkarwin', 'Bill', 'Karwin', 'bill@example.com',
SHA2('xyzzy'), NULL, 49.95);
```

**You Ain’t Gonna Need It (Bandwidth Efficiency)**  
By not using wildcards, you naturally stop fetching data you don't use. This improves performance and scalability.

```sql
SELECT date_reported, summary, description, resolution, status, priority
FROM Bugs;
```

**Giving Up Wildcards**  
Eventually, if you need to use a column alias, apply a function, or optimize performance, you have to stop using `*` anyway. It is better to be explicit from the start to make future changes easier.

