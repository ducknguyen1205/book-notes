
This chapter addresses the common challenges and misconceptions surrounding NULL values in SQL.

1 Objective: Distinguish Missing Values

- The goal is to effectively manage situations where data is missing or inapplicable in a database.

2 Antipattern: Use Null as an Ordinary Value, or Vice Versa

- **The core issue:** Many developers misunderstand how SQL handles NULL, treating it like zero, an empty string, or false, which leads to errors.

*   **Using Null in Expressions:**
    *   Arithmetic operations with NULL result in NULL, not a usable value.
    *Example:*
    ```sql
    SELECT hours + 10 FROM Bugs;
    ```
    This SQL expression with return null and not, for example, 10.
*   **Searching Nullable Columns:**
    *   Direct comparisons (= or <>) with NULL always result in "unknown," not true or false.
    *Example:*
    ```sql
    SELECT * FROM Bugs WHERE assigned_to = 123;
    SELECT * FROM Bugs WHERE NOT (assigned_to = 123);
    ```
*   **Avoiding the Issue:**
    *   Some developers disallow NULLs altogether, substituting them with arbitrary values. This is a bad solution because it introduces ambiguity and can lead to incorrect calculations.
    *Example:*
    
```sql
CREATE TABLE Bugs (
-- other columns
assigned_to BIGINT UNSIGNED NOT NULL,
hours NUMERIC(9,2) NOT NULL,
FOREIGN KEY (assigned_to) REFERENCES Accounts(account_id)
);
```

```sql
INSERT INTO Bugs (assigned_to, hours) VALUES (-1, -1);
```

The hours column is numeric, so you’re restricted to a numeric value to mean “unspecified.” It has to have no meaning in that column, so you chose a negative value. But the value -1 would throw off calculations such as SUM( ) or AVG( ). You have to exclude rows with this value, using special-case expressions, which is what you were trying to avoid by prohibiting null.

3 How to Recognize the Antipattern

- **Common phrases indicating this antipattern:**
    
    - “How do I find rows where no value has been set in the assigned_to (or other) column?”
    - “The full names of some users appear blank in the application presentation, but I can see them in the database.”
    - “The report of total hours spent working on this project includes only a few of the bugs that we completed! Only those for which we assigned a priority are included.”
    - “It turns out we can’t use the string we’ve been using to represent unknown in the Bugs table, so we need to have a meeting to discuss what new special value we can use and estimate the development time to migrate our data and convert our code to use that value.”

4 Legitimate Uses of the Antipattern

- Using special values to represent NULL may be required when importing/exporting data or handling user input.

5 Solution: Use Null as a Unique Value

- **Key principle:** Understand and respect SQL's three-valued logic (true, false, unknown).

*Handling Null in Scalar Expressions:* The key concept for understanding how null values behave in boolean expressions is that null is neither true nor false.
![[Pasted image 20251222095940.png]]

![[Pasted image 20251222100019.png]]

Searching for Null: You must make queries using the `IS NULL` predicate.
Example:
```sql
SELECT * FROM Bugs WHERE assigned_to IS NULL;
SELECT * FROM Bugs WHERE assigned_to <> 'NULL';
```

*Declaring Columns NOT NULL:* You should make sure that the primary key values are unique across all the split tables.
*Example:*
```sql
SELECT a.* FROM Accounts a
LEFT OUTER JOIN (
SELECT * FROM Bugs_2008
UNION ALL
SELECT * FROM Bugs_2009
UNION ALL
SELECT * FROM Bugs_2010
) t ON (a.account_id = t.reported_by);
```

*   **Dynamic Defaults:** Use `COALESCE()` to substitute a default value for NULL in a specific query without altering the underlying data.
*Example:*
```sql
SELECT first_name || COALESCE(' ' || middle_initial || ' ', ' ') || last_name AS full_name FROM Accounts;
```

# Note

## The right result for the wrong reason

The source explains how a programmer can make a mistake by putting quotes around the `NULL` keyword in a `WHERE` clause, but still get the intended result "by serendipity". Here's a breakdown:

- **The Incorrect Query:** The SQL query `SELECT * FROM Bugs WHERE assigned_to <> 'NULL'` is syntactically incorrect because it compares the column `assigned_to` to the _string_ `'NULL'` rather than the SQL keyword `NULL`.
    
- **Intended Result:** The programmer intends to exclude rows where `assigned_to` is `NULL`.
    
- **How it Works (Sometimes):**
    
    - **Case 1: `assigned_to` is NULL:** When `assigned_to` is `NULL`, the comparison `assigned_to <> 'NULL'` evaluates to _unknown_, which is treated as false, thus excluding the row from the result. This achieves the programmer's intention.
    - **Case 2: `assigned_to` is an Integer:** When `assigned_to` is an integer, the string `'NULL'` is converted to integer `0` in most databases. Since the integer value of `assigned_to` is greater than zero, the condition is always true. Again, this may also achieve the programmer's intention.
- **The Problem:** The "correct" result is coincidental and relies on specific data types and database behaviors. The query `SELECT * FROM Bugs WHERE assigned_to = 'NULL'` will _never_ return the intended results.

## `IS DISTINCT FROM`

### What it does:

`IS DISTINCT FROM` compares two values and returns `TRUE` if they are different, treating `NULL` as a regular value that can be compared.

**Key difference from `<>` or `!=`:**

- `a <> b` returns `NULL` if either value is `NULL`
- `a IS DISTINCT FROM b` returns `FALSE` if both are `NULL`, and `TRUE` if only one is `NULL`

### Example:

```sql
-- Regular comparison
NULL <> NULL     → NULL (not TRUE or FALSE)
5 <> NULL        → NULL

-- IS DISTINCT FROM
NULL IS DISTINCT FROM NULL     → FALSE (they're the same)
5 IS DISTINCT FROM NULL        → TRUE (they're different)
5 IS DISTINCT FROM 10          → TRUE (they're different)
5 IS DISTINCT FROM 5           → FALSE (they're the same)
```

### Database support:

- **PostgreSQL** ✅
- **SQLite** ✅
- **H2** ✅
- **Apache Derby** ✅
- **Oracle** ❌ (but has alternative syntax)
- **MySQL/MariaDB** ❌ (use `<=>` operator instead)
- **SQL Server** ❌ (use `IS NULL` checks manually)

**Note:** There's also `IS NOT DISTINCT FROM`, which does the opposite - returns `TRUE` if values are the same (treating `NULL` equals `NULL`).