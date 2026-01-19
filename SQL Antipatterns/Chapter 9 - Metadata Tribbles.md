
The problem: striving for **scalability** by cloning tables or columns.

#### 9.1 Objective: Support Scalability

The aim is to structure a database to improve query performance and support tables that grow steadily.

#### 9.2 Antipattern: Clone Tables or Columns

The core issue is trying to have few rows in every table which leads to two forms of the antipattern.

- Splitting a single long table into multiple smaller tables, using table names based on distinct data values in one of the table’s attributes.
- Splitting a single column into multiple columns, using column names based on distinct values in another attribute.

##### Spawning Tables

To split data into separate tables, you’d need some policy for which rows belong in which tables. For example, you could split them up by the year in the date_reported column:

```
CREATE TABLE Bugs_2008 ( . . . );
CREATE TABLE Bugs_2009 ( . . . );
CREATE TABLE Bugs_2010 ( . . . );
```

As you insert rows into the database, it’s your responsibility to use the correct table, depending on the values you insert:

```
INSERT INTO Bugs_2010 (..., date_reported, ...) VALUES (..., '2010-06-01', ...);
```

Fast forward to January 1 of the next year. Your application starts getting an error from all new bug reports, because you didn’t remember to create the Bugs_2011 table.

```
INSERT INTO Bugs_2011 (..., date_reported, ...) VALUES (..., '2011-02-20', ...);
```

##### Managing Data Integrity

Suppose your boss is trying to count bugs reported during the year, but his numbers don’t adding up. After investigating, you discover that some 2010 bugs were entered in the Bugs_2009 table by mistake. The following query should always return an empty result, and if it doesn’t, you have a problem:

```sql
SELECT * FROM Bugs_2009
WHERE date_reported NOT BETWEEN '2009-01-01' AND '2009-12-31';
```

There’s no way to limit the data relative to the name of its table automatically, but you can declare a CHECK constraint in each of your tables:

```sql
CREATE TABLE Bugs_2009 (
-- other columns
date_reported DATE CHECK (EXTRACT(YEAR FROM date_reported) = 2009)
);
CREATE TABLE Bugs_2010 (
-- other columns
date_reported DATE CHECK (EXTRACT(YEAR FROM date_reported) = 2010)
);
```

##### Synchronizing Data

One day, your customer support analyst asks to change a bug report date. It’s in the database as reported on 2010-01-03, but the customer who reported it actually sent it in by fax a week earlier, on 2009-12-27. You could change the date with a simple UPDATE:

```sql
UPDATE Bugs_2010
SET date_reported = '2009-12-27'
WHERE bug_id = 1234;
```

But this correction makes the row an invalid entry in the Bugs_2010 table. You would need to remove the row from one table and insert it into the other table, in the infrequent case that a simple UPDATE would cause this anomaly.

```sql
INSERT INTO Bugs_2009 (bug_id, date_reported, ...)
SELECT bug_id, date_reported, ...
FROM Bugs_2010
WHERE bug_id = 1234;
DELETE FROM Bugs_2010 WHERE bug_id = 1234;
```

##### Ensuring Uniqueness

You should make sure that the primary key values are unique across all the split tables. If you need to move a row from one table to another, you need some assurance that the primary key value doesn’t conflict with another row.

```sql
CREATE TABLE BugsIdGenerator (bug_id SERIAL PRIMARY KEY);
INSERT INTO BugsIdGenerator (bug_id) VALUES (DEFAULT);
ROLLBACK;
INSERT INTO Bugs_2010 (bug_id, . . .)
VALUES (LAST_INSERT_ID(), . . .);
```

##### Querying Across Tables

Inevitably, your boss needs a query that references multiple tables. For example, he may ask for a count of all open bugs regardless of the year they were created. You can reconstruct the full set of bugs using a UNION of all the split tables and query that as a derived table:

```sql
SELECT b.status, COUNT(*) AS count_per_status FROM (
SELECT * FROM Bugs_2008
UNION
SELECT * FROM Bugs_2009
UNION
SELECT * FROM Bugs_2010 ) AS b
GROUP BY b.status;
```

##### Synchronizing Metadata

Your boss tells you to add a column to track the hours of work required to resolve each bug.

```sql
ALTER TABLE Bugs_2010 ADD COLUMN hours NUMERIC(9,2);
```

##### Managing Referential Integrity

If a dependent table like Comments references Bugs, the dependent table cannot declare a foreign key. A foreign key must specify a single table, but in this case the parent table is split into many.

```sql
CREATE TABLE Comments (
comment_id SERIAL PRIMARY KEY,
bug_id BIGINT UNSIGNED NOT NULL,
FOREIGN KEY (bug_id) REFERENCES Bugs_????(bug_id)
);
```

The split table may also have problems being a dependent instead of a parent. For example, Bugs.reported_by references the Accounts table. If you want to query all bugs reported by a given person regardless of the year, you need a query like the following:

```sql
SELECT * FROM Accounts a
JOIN (
SELECT * FROM Bugs_2008
UNION ALL
SELECT * FROM Bugs_2009
UNION ALL
SELECT * FROM Bugs_2010
) t ON (a.account_id = t.reported_by)
```

##### Identifying Metadata Tribbles Columns

Columns can be Metadata Tribbles, too. You can create a table containing columns that are bound to propagate by their nature, as we saw in the story at the beginning of this chapter.

```sql
CREATE TABLE ProjectHistory (
bugs_fixed_2008 INT,
bugs_fixed_2009 INT,
bugs_fixed_2010 INT
);
```

#### 9.3 How to Recognize the Antipattern

- “Then we need to create a table (or column) per . . . ”
- “What’s the maximum number of tables (or columns) that the database supports?”
- “We found out why the application failed to add new data this morning: we forgot to create a new table for the new year."
- “How do I run a query to search many tables at once? All the tables have the same columns.”
- “How do I pass a parameter for a table name? I need to query a table name appended with the year number dynamically.”

#### 9.4 Legitimate Uses of the Antipattern

One good use of manually splitting tables is for archiving—removing historical data from day-to-day use.

#### 9.5 Solution: Partition and Normalize

There are better ways to improve performance if a table gets too large, instead of splitting the table manually. These include horizontal partitioning, vertical partitioning, and using dependent tables.

##### Using Horizontal Partitioning

You can gain the benefits of splitting a large table without the drawbacks by using a feature that is called either horizontal partitioning or sharding.

```sql
CREATE TABLE Bugs (
bug_id SERIAL PRIMARY KEY,
-- other columns
date_reported DATE
) PARTITION BY HASH ( YEAR(date_reported) )
PARTITIONS 4;
```

##### Using Vertical Partitioning

Whereas horizontal partitioning splits a table by rows, vertical partitioning splits a table by columns. Splitting a table by columns can have advantages when some columns are bulky or seldom needed.

```sql
CREATE TABLE ProductInstallers (
product_id BIGINT UNSIGNED PRIMARY KEY,
installer_image BLOB,
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

```sql
CREATE TABLE Bugs (
bug_id SERIAL PRIMARY KEY, -- fixed length data type
summary CHAR(80), -- fixed length data type
date_reported DATE, -- fixed length data type
reported_by BIGINT UNSIGNED, -- fixed length data type
FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
);
CREATE TABLE BugDescriptions (
bug_id BIGINT UNSIGNED PRIMARY KEY,
description VARCHAR(1000), -- variable length data type
resolution VARCHAR(1000) -- variable length data type
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id)
);
```

##### Fixing Metadata Tribbles Columns

Similar to the solution we saw in Chapter 8, Multicolumn Attributes, the remedy for Metadata Tribbles columns is to create a dependent table.

```sql
CREATE TABLE ProjectHistory (
project_id BIGINT,
year SMALLINT,
bugs_fixed INT,
PRIMARY KEY (project_id, year),
FOREIGN KEY (project_id) REFERENCES Projects(project_id)
);
```