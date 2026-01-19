### Introduction

A developer attempts to answer a complex request from a boss (product counts, developer stats, bug reports) using a single, monolithic query to be "efficient."

```sql
SELECT COUNT(bp.product_id) AS how_many_products,
COUNT(dev.account_id) AS how_many_developers,
COUNT(b.bug_id)/COUNT(dev.account_id) AS avg_bugs_per_developer,
COUNT(cust.account_id) AS how_many_customers
FROM Bugs b JOIN BugsProducts bp ON (b.bug_id = bp.bug_id)
JOIN Accounts dev ON (b.assigned_to = dev.account_id)
JOIN Accounts cust ON (b.reported_by = cust.account_id)
WHERE cust.email NOT LIKE '%@example.com'
GROUP BY bp.product_id;
```

The results are incorrect because the joins create duplicates and logical errors, leading to the "Spaghetti Query" problem.

### 18.1 Objective: Decrease SQL Queries

Programmers often assume that performing a task in a single SQL query is always more elegant and efficient than running multiple queries. They try to avoid splitting tasks, believing that more queries equal poor performance.

### 18.2 Antipattern: Solve a Complex Problem in One Step

Trying to do too much in one statement leads to complexity and errors.

**Unintended Products**  
Joining tables without strict restrictions causes a **Cartesian product**, where every row in one table matches every row in another.

_Example:_ A query tries to count fixed and open bugs simultaneously.

```sql
SELECT p.product_id,
COUNT(f.bug_id) AS count_fixed,
COUNT(o.bug_id) AS count_open
FROM BugsProducts p
LEFT OUTER JOIN Bugs f ON (p.bug_id = f.bug_id AND f.status = 'FIXED')
LEFT OUTER JOIN Bugs o ON (p.bug_id = o.bug_id AND o.status = 'OPEN')
WHERE p.product_id = 1
GROUP BY p.product_id;
```

Because there are 12 fixed bugs and 7 open bugs, the Cartesian product multiplies them (12 × 7), resulting in 84 for both counts instead of the actual values.

```text
product_id count_fixed count_open
1 84 84
```

You can see the cross-matching of rows in this visualization:

![[Pasted image 20251226092217.png]]

If we remove the grouping, we see the raw Cartesian matches:

```sql
SELECT p.product_id, f.bug_id AS fixed, o.bug_id AS open
FROM BugsProducts p
JOIN Bugs f ON (p.bug_id = f.bug_id AND f.status = 'FIXED')
JOIN Bugs o ON (p.bug_id = o.bug_id AND o.status = 'OPEN')
WHERE p.product_id = 1;
```

**As Though That Weren’t Enough. . .**  
Complex queries are hard to write, debug, and modify. The database engine also struggles to optimize "monster" queries, often making them slower than several simple ones.

### 18.3 How to Recognize the Antipattern

Suspect a Spaghetti Query if you hear:

- “Why are my sums and counts impossibly large?” (Indicates a Cartesian product).
    
- “I’ve been working on this monster SQL query all day!”
    
- “We can’t add anything to our database report, because it will take too long to figure out how to recode the SQL query.”
    
- “Try putting another DISTINCT into the query.” (Used to mask duplication errors).
    

### 18.4 Legitimate Uses of the Antipattern

A single complex query may be necessary if:

- You are using a reporting tool or framework that demands a single data source.
    
- You need the database to return all results in a specific sorted order.
    

### 18.5 Solution: Divide and Conquer

Follow the **Law of Parsimony**: simpler is better. Split complex tasks into smaller, manageable queries.

**One Step at a Time**  
Instead of one faulty join, run two separate queries to get accurate counts.

```sql
SELECT p.product_id, COUNT(f.bug_id) AS count_fixed
FROM BugsProducts p
LEFT OUTER JOIN Bugs f ON (p.bug_id = f.bug_id AND f.status = 'FIXED')
WHERE p.product_id = 1
GROUP BY p.product_id;
```

```sql
SELECT p.product_id, COUNT(o.bug_id) AS count_open
FROM BugsProducts p
LEFT OUTER JOIN Bugs o ON (p.bug_id = o.bug_id AND o.status = 'OPEN')
WHERE p.product_id = 1
GROUP BY p.product_id;
```

This approach is easier to verify, maintain, and often performs better.

**Look for the UNION Label**  
If you need a single result set, use `UNION` to combine results from simple queries.

```sql
(SELECT p.product_id, f.status, COUNT(f.bug_id) AS bug_count
FROM BugsProducts p
LEFT OUTER JOIN Bugs f ON (p.bug_id = f.bug_id AND f.status = 'FIXED')
WHERE p.product_id = 1
GROUP BY p.product_id, f.status)
UNION ALL
(SELECT p.product_id, o.status, COUNT(o.bug_id) AS bug_count
FROM BugsProducts p
LEFT OUTER JOIN Bugs o ON (p.bug_id = o.bug_id AND o.status = 'OPEN')
WHERE p.product_id = 1
GROUP BY p.product_id, o.status)
ORDER BY bug_count;
```

**Solving Your Boss’s Problem**  
The correct way to handle the scenario in the introduction is to calculate each metric separately:

- How many products:
    

```sql
SELECT COUNT(*) AS how_many_products
FROM Products;
```

- How many developers fixed bugs:
    

```sql
SELECT COUNT(DISTINCT assigned_to) AS how_many_developers
FROM Bugs
WHERE status = 'FIXED';
```

- Average number of bugs fixed per developer:
    

```sql
SELECT AVG(bugs_per_developer) AS average_bugs_per_developer
FROM (SELECT dev.account_id, COUNT(*) AS bugs_per_developer
FROM Bugs b JOIN Accounts dev
ON (b.assigned_to = dev.account_id)
WHERE b.status = 'FIXED'
GROUP BY dev.account_id) t;
```

- How many of our fixed bugs were reported by customers:
    

```sql
SELECT COUNT(*) AS how_many_customer_bugs
FROM Bugs b JOIN Accounts cust ON (b.reported_by = cust.account_id)
WHERE b.status = 'FIXED' AND cust.email NOT LIKE '%@example.com';
```

**Writing SQL Automatically—with SQL**  
You can use SQL to generate code for repetitive tasks, such as mass updates that require different values for different rows.

```sql
SELECT CONCAT('UPDATE Inventory '
' SET last_used = ''', MAX(u.usage_date), '''',
' WHERE inventory_id = ', u.inventory_id, ';') AS update_statement
FROM ComputerUsage u
GROUP BY u.inventory_id;
```

This generates a script of simple statements that are easy to execute:

```sql
update_statement
UPDATE Inventory SET last_used = ’2002-04-19’ WHERE inventory_id = 1234;
UPDATE Inventory SET last_used = ’2002-03-12’ WHERE inventory_id = 2345;
UPDATE Inventory SET last_used = ’2002-04-30’ WHERE inventory_id = 3456;
UPDATE Inventory SET last_used = ’2002-04-04’ WHERE inventory_id = 4567;
. . .
```

# Notes

## Cartesian & Cartesian product
### Cartesian (Hệ tọa độ Đề-cát)

"Cartesian" refers to the familiar graph with an **x-axis** (trục hoành) and a **y-axis** (trục tung). It's like a map for finding any point using a pair of numbers, such as (3, 4). This system is named after the mathematician René Descartes.

### Cartesian Product (Tích Đề-cát)

A Cartesian Product is about creating all possible pairs from two groups.

Imagine you have:
*   **Shirts (Áo):** {Red, Blue}
*   **Pants (Quần):** {Black, White}

The Cartesian Product is every possible outfit combination:
*   (Red Shirt, Black Pants)
*   (Red Shirt, White Pants)
*   (Blue Shirt, Black Pants)
*   (Blue Shirt, White Pants)

In short:
*   **Cartesian:** A graph system for finding points.
*   **Cartesian Product:** A way to list all possible combinations.