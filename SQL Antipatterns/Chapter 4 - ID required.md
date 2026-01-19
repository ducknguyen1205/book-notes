
Recently I answered a question that I see frequently, from a software developer trying to ==prevent duplicate rows==. At first I thought he must lack a primary key. But that wasn’t his problem. In his content management database, he stored articles for publishing on a website. He used an intersection table for a many-to-many association between a table of articles and a table of tags.

```sql
CREATE TABLE ArticleTags (
id SERIAL PRIMARY KEY,
article_id BIGINT UNSIGNED NOT NULL,
tag_id BIGINT UNSIGNED NOT NULL,
FOREIGN KEY (article_id) REFERENCES Articles (id),
FOREIGN KEY (tag_id) REFERENCES Tags (id)
);
```

He was getting incorrect results from queries when counting the num- ber of articles with a given tag. He knew that there were only five articles with the “economy” tag, but the query was telling him there were seven.

```sql
SELECT tag_id, COUNT(*) AS articles_per_tag FROM ArticleTags WHERE tag_id = 327;
```

When he queried all the rows matching that tag_id, he saw that the tag was associated with one particular article in triplicate; three rows showed the same association, although they had different values for id.

```
id tag_id article_id
22 327 1234
23 327 1234
24 327 1234
```

This table had a primary key, but that primary key wasn’t preventing duplicates in the columns that mattered. ==One remedy might be to create a UNIQUE constraint over the other two columns, but given that, why is the id column needed at all?==

#### 4.1 Objective: Establish Primary Key Conventions

The objective is to make sure every table has a primary key, but confusion about the nature of a primary key has resulted in an antipattern.

- A **primary key** is integral to good database design and is guaranteed to be unique, which prevents duplicate rows.
- The tricky part is choosing a column to serve as the primary key.
- A new column is needed in such tables to store an artificial value that has no meaning in the domain modeled by the table. This column is used as the primary key, so you can address rows uniquely while allowing any other attribute column to contain duplicates, if that’s appropriate. _This type of primary key column is sometimes called a pseudokey or a surrogate key_.

##### Do I Really Need a Primary Key?

I’ve heard some software developers claim that their table doesn’t need a primary key.

Sometimes these programmers want to avoid the imagined overhead of maintaining a unique index, or else they have tables with no columns they can use for this purpose. A primary key constraint is important when you need to do the following:

- Prevent a table from containing duplicate rows
- Reference individual rows in queries
- Support foreign key references If you don’t use primary key constraints, you create a chore for yourself: checking for duplicate rows.

```
SELECT bug_id FROM Bugs GROUP BY bug_id HAVING COUNT(*) > 1;
```

How frequently should you run this check? What should you do with a duplicate when you find one? A table without a primary key is like organizing your MP3 col-lection with no song titles. You can still listen to the music, but you can’t find the one you want or keep duplicates out of your collection. Pseudokeys weren’t standardized until SQL:2003, so each database uses its own extension to SQL to implement them. Even the terminology for pseudokeys is vendor-dependent, as shown by the following table:

##### Feature Supported by Database Brands

```
AUTO_INCREMENT MySQL
GENERATOR Firebird, InterBase
IDENTITY DB2, Derby, Microsoft SQL Server, Sybase
ROWID SQLite
SEQUENCE DB2, Firebird, Informix, Ingres, Oracle, PostgreSQL
SERIAL MySQL, PostgreSQL
```

Pseudokeys are a useful feature, but they aren’t the only solution for declaring a primary key.

#### 4.2 Antipattern: One Size Fits All

Books, articles, and programming frameworks have established a cultural convention that every database table must have a primary key column with the following characteristics:

- The primary key’s column name is id.
- Its data type is a 32-bit or 64-bit integer.
- Unique values are generated automatically.

The presence of a column named id in every table is so common that this has become synonymous with a primary key. Programmers learning SQL get the false idea that a primary key always means a column defined in this manner.

```sql
CREATE TABLE Bugs (
id SERIAL PRIMARY KEY,
description VARCHAR(1000),
-- . . .
);
```

Adding an id column to every table causes several effects that make its use seem arbitrary.

##### Making a Redundant Key

==You might see an id column defined as the primary key simply for the sake of tradition, even when another column in the same table could be used as the natural primary key.== The other column may even be defined with a UNIQUE constraint. For example, in the Bugs table, the application might label bugs using a string with a mnemonic for the project the bug belongs to, or other identifying information.

```sql
CREATE TABLE Bugs (
id SERIAL PRIMARY KEY,
bug_id VARCHAR(10) UNIQUE,
description VARCHAR(1000),
-- . . .
);
INSERT INTO Bugs (bug_id, description, ...)
VALUES ('VIS-078', 'crashes on save', ...);
```

The bug_id column in the previous example has similar usage to the id, in that it serves to identify each row uniquely.

##### Allowing Duplicate Rows

A compound key consists of multiple columns. One typical use for a compound key is in an intersection table like BugsProducts. The primary key should ensure that a given combination of values for bug_id and product_id appears only once in the table, even though each value may appear many times in different pairings. However, when you use the mandatory id column as the primary key, the constraint no longer applies to two columns that should be unique.

```sql
CREATE TABLE BugsProducts (
id SERIAL PRIMARY KEY,
bug_id BIGINT UNSIGNED NOT NULL,
product_id BIGINT UNSIGNED NOT NULL,
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
INSERT INTO BugsProducts (bug_id, product_id)
VALUES (1234, 1), (1234, 1), (1234, 1); -- duplicates are permitted
```

Duplicates in this intersection table cause unintended results when you use the table to match Bugs to Products. To prevent duplicates, you could declare a UNIQUE constraint over the two columns besides id:

```sql
CREATE TABLE BugsProducts (
id SERIAL PRIMARY KEY,
bug_id BIGINT UNSIGNED NOT NULL,
product_id BIGINT UNSIGNED NOT NULL,
UNIQUE KEY (bug_id, product_id),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

==But if you need a unique constraint over those two columns anyway, the id column is superfluous.==

##### Obscuring the Meaning of the Key

_The word code has a number of definitions, one of which is a way to communicate a message with brevity or secrecy. In programming, we should have the opposite goal—to make meaning clearer_. The name id is so generic that it holds no meaning. This is especially important when you join two tables and they have the same primary key column name.

```
SELECT b.id, a.id
FROM Bugs b
JOIN Accounts a ON (b.assigned_to = a.id)
WHERE b.status = 'OPEN';
```

How do you tell the bug id from the account id in your application code if you reference columns by name instead of by ordinal position? This is a problem especially in dynamic languages like PHP, when a query result is an associative array: one column overwrites the other unless you specify column aliases in your query. The name of the id column doesn’t help make the query any clearer. But if the columns were named bug_id and account_id, the reader would have a much easier time reading the query results. We use a primary key to address individual rows of a table, so the column’s name should give a clue about the type of entity in that table.

##### Using USING

You’re probably familiar with the SQL syntax for a join, using the key- words JOIN and ON preceding an expression to evaluate matching rows in the two tables.

```
SELECT * FROM Bugs AS b JOIN BugsProducts AS bp ON (b.bug_id = bp.bug_id);
```

SQL also supports a more concise syntax for expressing a join between two tables. You can rewrite the previous query in the following way if the columns have the same name in both tables:

```
SELECT * FROM Bugs JOIN BugsProducts USING (bug_id);
```

However, if all tables are required to define a pseudokey primary key named id, then a foreign key column in a dependent table can never use the same name as the primary key it references. Instead, you must always use the more verbose ON syntax:

```
SELECT * FROM Bugs AS b JOIN BugsProducts AS bp ON (b.id = bp.bug_id);
```

##### Compound Keys Are Hard

Some developers refuse to use compound keys because they say these keys are too hard to use. Any expression that compares a key to another must compare all columns. A foreign key that references a compound

##### Special Scope for Sequences

Some people allocate a value for a new row by taking the greatest value currently in use and adding one.

```
SELECT MAX(bug_id) + 1 AS next_bug_id FROM Bugs;
```

This isn’t reliable when you have concurrent clients both query-ing for the next value to use. The same value could be used by both clients. This is called a race condition. To avoid the race condition, you have to block concurrent inserts while you read the current maximum value and then use it in a new row. To do this, you have to lock the whole table— row-level locking isn’t enough. Table locks create a bottleneck because they cause concurrent clients to queue up for access.

Sequences solve this by operating outside of transaction scope. They never allocate the same value to multiple clients and therefore never roll back allocation of a value, whether or not you commit that value in a row. Because sequences work this way, multiple clients can generate unique values concurrently and be assured they won’t try to use the same value. Most databases support some function to return the last value a sequence generated. For example, MySQL calls this function LAST_INSERT_ID( ), Microsoft SQL Server uses SCOPE_IDENTITY( ), and Oracle uses SequenceName.CURRVAL( ). These functions return the value generated during the current session, even if other clients generate their own values concur-rently. No race condition exists.

primary key must itself be a compound foreign key. It requires more typing to use compound keys. This refusal is like a mathematician refusing to use two-dimensional or three-dimensional coordinates, instead performing all calculations as though objects exist within a one-dimensional, linear space. It’s true that this would make a lot of geometry and trigonometry much simpler, but it fails to describe real-world objects that we need to work with.

#### 4.3 How to Recognize the Antipattern

The symptom of this antipattern is easy to recognize: tables use the overly generic name id for the primary key. There’s virtually no reason to prefer this column name over one that is more descriptive. The following can also be evidence of the antipattern:

- “I don’t think I need a primary key in this table.” _The developer who says this is confusing the term primary key with pseudokey. Every table must have a primary key constraint to prevent duplicate rows and identify individual rows. They might want to use a natural key or a compound key instead_.
- “How did I get duplicate many-to-many associations?” _An intersection table for a many-to-many relationship should declare a primary key constraint, or at least a unique key constraint, over the set of foreign key columns_.
- “I read that database theory says I should move values to a lookup table and refer to them by ID. But I don’t want to do that because I have to do a join every time I want the actual values.” _This is a common misunderstanding of database design theory called normalization, which has nothing to do with pseudokeys in reality. For more on this, see Appendix A, on page 294_.

#### 4.4 Legitimate Uses of the Antipattern

_Some object-relational frameworks simplify development by assuming convention over configuration. They expect every table to define its primary key in the same way: as an integer pseudokey column named id_. If you use such a framework, you may want to conform to its conven-tions, because this gives you access to other desirable features of the framework. There’s also nothing wrong with using a pseudokey, or assigning val- ues from an auto-incrementing integer mechanism. But not every table needs a pseudokey, and it’s not necessary to name every pseudokey id. A pseudokey is a good choice as a surrogate for a natural key that’s too long to be practical. For example, for a table that records attributes of a file on the filesystem, the path of the file might be a good natural key, but it would be costly to index a string column that long.

#### 4.5 Solution: Tailored to Fit

A primary key is a constraint, not a data type. You can declare a pri- mary key on any column or set of columns, as long as the data types support indexing. You should also be able to define a column as an auto-incrementing integer without making it the primary key of the table. The two concepts are independent. Don’t let inflexible conventions get in the way of good design.

##### Tell It Like It Is

==Choose sensible names for your primary key. The name should convey the type of entity that the primary key identifies. For example, the primary key of the Bugs table should be bug_id.== Use the same column name in foreign keys where possible. This often means that the name of a primary key should be unique within your schema; no two tables should use the same name for their primary key, unless one is also a foreign key referencing the other. However, there are exceptions: sometimes it is appropriate for a foreign key to be named differently from the primary key it references, for instance to be descriptive of the nature of the association.

```
CREATE TABLE Bugs (
-- . . .
reported_by BIGINT UNSIGNED NOT NULL,
FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
);
```

An industry standard exists to describe naming conventions for meta- data. The standard, called ISO/IEC 11179,1 is a guideline for “managing classification schemes” in information technology systems. In other words, this is how you should name your tables and columns sensibly. Like most ISO standards, this document is nearly impenetrable, but _Joe Celko applies it practically to SQL in his book SQL Programming Style [Cel05]_.

##### Be Unconventional

Object-relational frameworks expect you to use a pseudokey named id, but they also allow you to override this and declare a different name instead. The following example uses Ruby on Rails:2

```
class Bug < ActiveRecord::Base
set_primary_key "bug_id"
end
```

Some developers think that specifying the primary key column is necessary only when supporting legacy databases where they can’t use their preferred conventions. In fact, supporting sensible column names is also important in new projects.

##### Embrace Natural Keys and Compound Keys

==If your table contains an attribute that’s guaranteed to be unique, is non-null, and can serve to identify the row, don’t feel obligated to add a pseudokey solely for the sake of tradition.== Practically speaking, it’s not uncommon for every attribute in a table to be subject to change or to be nonunique. Databases tend to evolve dur- ing the lifetime of a project, and decision makers may not respect the sanctity of a natural key. Sometimes a column that at first seemed like it would be a good natural key turns out to have legitimate duplicates. In those cases, a pseudokey is the only solution. Use compound keys when they’re appropriate. When a row is best identified by the combination of multiple attribute columns, as in the BugsProducts table, use those columns in a compound primary key.

```
CREATE TABLE BugsProducts (
bug_id BIGINT UNSIGNED NOT NULL,
product_id BIGINT UNSIGNED NOT NULL,
PRIMARY KEY (bug_id, product_id),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
INSERT INTO BugsProducts (bug_id, product_id)
VALUES (1234, 1), (1234, 2), (1234, 3);
```

```
INSERT INTO BugsProducts (bug_id, product_id)
VALUES (1234, 1); -- error: duplicate entry
```

Note that foreign keys that reference a compound primary key also need to be compound. This may seem clumsy to duplicate these columns in dependent tables, but they can have advantages too: you might sim- plify a query that would have required a join to fetch attributes of the referenced row. _Conventions are good only if they are helpful._