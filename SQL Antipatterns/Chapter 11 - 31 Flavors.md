##### 11.1 Objective: Restrict a Column to Specific Values

Restricting a column’s values to a fixed set of values is very useful.

##### 11.2 Antipattern: Specify Values in the Column Definition

Many people choose to specify the valid data values when they define the column using check constraints, ENUM data type or triggers.

```sql
CREATE TABLE PersonalContacts (
-- other columns
salutation VARCHAR(4)
CHECK (salutation IN ('Mr.', 'Mrs.', 'Ms.', 'Dr.', 'Rev.')),
);
```

```sql
CREATE TABLE Bugs (
-- other columns
status ENUM('NEW', 'IN PROGRESS', 'FIXED'),
);
```

**Problems with this approach**:

- **Enumerated list of values that are currently allowed in the status column?**
- **Adding a New Flavor**: There’s no syntax to add or remove a value from an ENUM or check constraint; you can only redefine the column with a new set of values.
    
```sql
ALTER TABLE Bugs MODIFY COLUMN status
ENUM('NEW', 'IN PROGRESS', 'FIXED', 'DUPLICATE');
```
    
- **Old Flavors Never Die**: _If you make a value obsolete, you could upset historical data_.
- **Portability Is Hard**: _Check constraints, domains, and UDTs are not supported uniformly among brands of SQL databases_.

	In MySQL’s implementation, you declare the values as strings, but internally the column is stored as the ordinal number of the string in the enumerated list. The storage is therefore compact, but when you sort a query by this column, the result is ordered by the ordinal value, not alphabetically by the string value. You may not expect this behavior.

##### 11.3 How to Recognize the Antipattern

- “We have to take the database offline so we can add a new choice in one of our application’s menus. It should take no more than thirty minutes, if all goes well.”
- “The status column can have one of the following values. We should not need to revise this list.”
- “The list of values in the application code got out of sync with the business rules in the database—again.”

##### 11.4 Legitimate Uses of the Antipattern

ENUM may cause fewer problems if the set of values is unchanging.

##### 11.5 Solution: Specify Values in Data

_There’s a better solution to restrict values in a column: create a lookup table with one row for each value you allow in the Bugs.status column. Then declare a foreign key constraint on Bugs.status referencing the new table_.

```sql
CREATE TABLE BugStatus (
status VARCHAR(20) PRIMARY KEY
);
```

```sql
INSERT INTO BugStatus (status) VALUES ('NEW'), ('IN PROGRESS'), ('FIXED');
CREATE TABLE Bugs (
-- other columns
status VARCHAR(20),
FOREIGN KEY (status) REFERENCES BugStatus(status)
ON UPDATE CASCADE
);
```

**Advantages of using a lookup table**:

- Querying the Set of Values: The set of permitted values is now stored in data, not metadata.
    
    ```sql
    SELECT status FROM BugStatus ORDER by status;
    ```
    
- Updating the Values in the Lookup Table:
    
    ```sql
    INSERT INTO BugStatus (status) VALUES ('DUPLICATE');
    ```
    
    ```sql
    UPDATE BugStatus SET status = 'INVALID' WHERE status = 'BOGUS';
    ```
    
- Supporting Obsolete Values: Add another attribute column to the lookup table to designate some values as obsolete.
    
    ```sql
    ALTER TABLE BugStatus ADD COLUMN active
    ENUM('INACTIVE', 'ACTIVE') NOT NULL DEFAULT 'ACTIVE';
    ```
    
    ```sql
    UPDATE BugStatus SET active = 'INACTIVE' WHERE status = 'DUPLICATE';
    ```
    
    ```sql
    SELECT status FROM BugStatus WHERE active = 'ACTIVE';
    ```
    
- Portability Is Easy: _the lookup table solution relies only on the standard SQL feature of declarative referential integrity using foreign key constraints. This makes the solution more portable_.