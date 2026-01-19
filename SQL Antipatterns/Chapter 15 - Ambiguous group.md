
#### 1 Objective: Get Row with Greatest Value per Group

The objective is to retrieve the entire row that contains the maximum value for a specific column within a group.

#### 2 Antipattern: Reference Nongrouped Columns

==The problem arises when a query uses `GROUP BY` but also selects columns that are not part of the grouping criteria or aggregate functions.== This can lead to ambiguous results.

For example, this query aims to find the latest bug reported for each product:

```sql
SELECT product_id, MAX(date_reported) AS latest, bug_id
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```

In standard SQL, this query would produce an error because `bug_id` is neither part of the `GROUP BY` clause nor an aggregate function. MySQL, however, might execute it but return an arbitrary `bug_id` from the group, which is misleading. The root cause is failing to follow the single-value rule. In others word, every column must have a single value per row group. Columns named in `GROUP BY` are guaranteed to have only one value per group, as are aggregate functions like `MAX()`.

#### 3 How to Recognize the Antipattern

The key symptom is using columns in the `SELECT` statement that are not in the `GROUP BY` clause and are not used with aggregate functions. Different database systems handle this differently. Some will raise an error, while others (like MySQL) may produce unreliable results.

Common error messages include:

- "Invalid expression in the select list (not contained in either an aggregate function or the GROUP BY clause)"
- "'bugs.b.bug_id' isn't in GROUP BY"
- "Column 'Bugs.bug_id' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause"
- "Not a GROUP BY expression"

#### 15.4 Legitimate Uses of the Antipattern

In MySQL, if the non-grouped columns are _functionally dependent_ on the grouped columns, the query might be valid. A functional dependency means the non-grouped column is uniquely determined by the grouped column. For example, if you are grouping by a table's primary key, any other column from that same table is functionally dependent on the primary key.

For example, the following query might work in MySQL if `account_name` is functionally dependent on `reported_by`:

```sql
SELECT b.reported_by, a.account_name
FROM Bugs b JOIN Accounts a ON (b.reported_by = a.account_id)
GROUP BY b.reported_by;
```

However, relying on this behavior is not portable and can lead to unexpected results.

#### 15.5 Solution: Use Columns Unambiguously

The best solution is to ensure that all columns in the `SELECT` statement are either part of the `GROUP BY` clause or used within an aggregate function. Here are some ways to achieve this:

- **Query Only Functionally Dependent Columns:** Only select columns that are functionally dependent on the grouping columns. However, this is not always possible or practical.
    
- **Use a Correlated Subquery:** A subquery can select the `bug_id` that corresponds to the maximum `date_reported` for each `product_id`.
    
    ```sql
    SELECT bp1.product_id, b1.date_reported AS latest, b1.bug_id
    FROM Bugs b1 JOIN BugsProducts bp1 USING (bug_id)
    WHERE NOT EXISTS
    (SELECT * FROM Bugs b2 JOIN BugsProducts bp2 USING (bug_id)
    WHERE bp1.product_id = bp2.product_id
    AND b1.date_reported < b2.date_reported);
    ```
    
- **Use a Derived Table:** Create a subquery (derived table) that finds the maximum `date_reported` for each `product_id`, then join this subquery with the `Bugs` table to retrieve the corresponding `bug_id`.
    
    ```sql
    SELECT m.product_id, m.latest, b1.bug_id
    FROM Bugs b1 JOIN BugsProducts bp1 USING (bug_id)
    JOIN (SELECT bp2.product_id, MAX(b2.date_reported) AS latest
    FROM Bugs b2 JOIN BugsProducts USING (bug_id)
    GROUP BY bp2.product_id) AS m
    ON (bp1.product_id = m.product_id AND b1.date_reported = m.latest);
    ```
    
- **Use an Outer Join:** Perform a `LEFT OUTER JOIN` to find the rows where a later date does not exist.
    
    ```sql
    SELECT bp1.product_id, b1.date_reported AS latest, b1.bug_id
    FROM Bugs b1 JOIN BugsProducts bp1 ON (b1.bug_id = bp1.bug_id)
    LEFT OUTER JOIN (Bugs AS b2 JOIN BugsProducts AS bp2 ON (b2.bug_id = bp2.bug_id))
    ON (bp1.product_id = bp2.product_id AND (b1.date_reported < b2.date_reported
    OR b1.date_reported = b2.date_reported AND b1.bug_id < b2.bug_id))
    WHERE b2.bug_id IS NULL;
    ```
    
- **Use an Aggregate Function for Extra Columns:** Apply an aggregate function (e.g., `MAX()`, `MIN()`, `GROUP_CONCAT()`) to the non-grouped columns to ensure a single value per group.
    
    ```sql
    SELECT product_id, MAX(date_reported) AS latest,
    MAX(bug_id) AS latest_bug_id
    FROM Bugs JOIN BugsProducts USING (bug_id)
    GROUP BY product_id;
    ```
    

# Notes

## Single-Value rule
The Single-Value Rule is a principle in SQL that governs how columns can be used in a `SELECT` statement when a `GROUP BY` clause is present.

### The Rule Explained

When you use a `GROUP BY` clause to aggregate rows, the Single-Value Rule dictates that any column in the `SELECT` list must be either:

- **An aggregate function:** A function that operates on a set of values to return a single value, such as `COUNT()`, `SUM()`, `AVG()`, `MAX()`, or `MIN()`.
    
- **Part of the `GROUP BY` clause:** The column is explicitly listed in the `GROUP BY` clause.
    

This rule ensures that for each group of rows, there is a single, unambiguous value to return for each column in the result set.

### Why it's Important

The Single-Value Rule prevents ambiguous query results. If you were to select a non-aggregated, non-grouped column, the database wouldn't know which value to display from the multiple rows within a group.

For example, consider a table of orders:

|OrderID|CustomerID|OrderDate|Amount|
|---|---|---|---|
|1|101|2025-12-20|50.00|
|2|102|2025-12-21|75.00|
|3|101|2025-12-22|120.00|

If you were to group by `CustomerID` to get the total amount for each customer, but also include the `OrderDate` in the `SELECT` list without aggregation, it would violate the Single-Value Rule. For `CustomerID` 101, there are two different `OrderDate` values, and the database wouldn't have a clear instruction on which one to show.

Most relational database management systems (RDBMS) enforce this rule. However, some, like older versions of MySQL, have been known to allow queries that violate this rule, often returning a value from the first row in the group, which can lead to unpredictable and misleading results. Modern SQL modes, like `ONLY_FULL_GROUP_BY` in MySQL, are used to enforce this standard SQL behavior.