

The chapter discusses the **Keyless Entry antipattern**, where developers skip defining foreign key constraints in SQL databases. While this might seem simpler initially, it leads to problems with data integrity and requires manual code to enforce relationships.

**5.1 Objective: Simplify Database Architecture**

The objective is to simplify database architecture but avoiding foreign keys can make data relationships hard to maintain. Referential integrity ensures that values in a foreign key column exist in the parent table.

**5.2 Antipattern: Leave Out the Constraints**

Leaving out constraints requires writing code to ensure referential integrity manually, which is error-prone.

- **Assuming Flawless Code**: Some developers write code to ensure data relationships are always satisfied. This requires extra `SELECT` queries and can still fail due to concurrency.
    
```sql
SELECT account_id FROM Accounts WHERE account_id = 1;

INSERT INTO Bugs (reported_by) VALUES (1);

SELECT bug_id FROM Bugs WHERE reported_by = 1;

DELETE FROM Accounts WHERE account_id = 1;
```
    
- **Checking for Mistakes**: Some developers use scripts to report corrupted data. This requires writing queries for every referential relationship.
    
```sql
SELECT b.bug_id, b.status
FROM Bugs b LEFT OUTER JOIN BugStatus s
ON (b.status = s.status)
WHERE s.status IS NULL;
```
    
It also requires determining how often to run these checks and how to correct any broken references.

```sql
UPDATE Bugs SET status = DEFAULT WHERE status = 'BANANA';
```
    
- **"It’s Not My Fault!"**: Perfect code is rare, and broken references can be introduced through ad hoc SQL statements.
    
- **Catch-22 Updates**: Foreign key constraints can make it inconvenient to update related columns in multiple tables.
    
```sql
DELETE FROM BugStatus WHERE status = 'BOGUS'; -- ERROR!
DELETE FROM Bugs WHERE status = 'BOGUS';
DELETE FROM BugStatus WHERE status = 'BOGUS'; -- retry succeeds

UPDATE BugStatus SET status = 'INVALID' WHERE status = 'BOGUS'; -- ERROR!
UPDATE Bugs SET status = 'INVALID' WHERE status = 'BOGUS'; -- ERROR!
```
    

**5.3 How to Recognize the Antipattern**

- Looking for values in one table that don't exist in another.
- Checking if a value exists in one table before inserting into another.
- Believing that foreign keys slow down the database.

**5.4 Legitimate Uses of the Antipattern**

If the database doesn’t support foreign key constraints or if using an ultra-flexible database design. Also, you may want to look at Chapter 6, _Entity-Attribute-Value, on page 73 and Chapter 7, Polymorphic_ Associations, on page 89.

**5.5 Solution: Declare Constraints**

Use foreign key constraints to enforce referential integrity.

```sql
CREATE TABLE Bugs (
-- . . .
reported_by BIGINT UNSIGNED NOT NULL,
status VARCHAR(20) NOT NULL DEFAULT 'NEW',
FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
FOREIGN KEY (status) REFERENCES BugStatus(status)
);
```

This helps avoid data integrity mistakes.

- **Supporting Multitable Changes**: Foreign keys support cascading updates.
    
    ```sql
    CREATE TABLE Bugs (
    -- . . .
    reported_by BIGINT UNSIGNED NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'NEW',
    FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,
    FOREIGN KEY (status) REFERENCES BugStatus(status)
    ON UPDATE CASCADE
    ON DELETE SET DEFAULT
    );
    ```
    
- **Overhead? Not Really**: Foreign keys improve performance and help maintain consistent referential integrity.