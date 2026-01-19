### Rules of Normalization

Normalization involves well-defined rules to design data storage that avoids redundancy and prevents application errors. It is not about restricting freedom but about using "defensive design" to ensure simplicity and flexibility.

### A.1 What Does Relational Mean?

A "relation" refers to a table (a set of unique rows) and the relationships between columns. To be a proper relation suitable for normalization, a table must meet these criteria:

- **Rows Have No Order:** The order is unpredictable unless `ORDER BY` is used.
    
- **Columns Have No Order:** Accessing data should rely on names, not position.
    
- **Duplicate Rows Are Not Allowed:** Every row must be unique, enforced by a **primary key**.
    
- **Every Column Has One Type, and One Value per Row:** No custom attributes per row (like Entity-Attribute-Value) and no fields serving multiple different meanings.
    
- **Rows Have No Hidden Components:** Data should not rely on internal storage IDs (like `ROWNUM` or `OID`).
    

### A.2 Myths About Normalization

There are common misunderstandings about normalization:

- **Myth:** "Normalization makes a database slower." **Fact:** While joins take time, denormalization often causes more expensive problems with data integrity and complex queries.
    
- **Myth:** "Normalization requires pseudokeys." **Fact:** Pseudokeys are useful but distinct from normalization rules.
    
- **Myth:** "Normalization separates attributes like EAV." **Fact:** Normalization makes data _more_ readable and structured, unlike EAV.
    
- **Myth:** "3NF is enough." **Fact:** Ignoring higher forms (like 4NF) leads to data loss or redundancy in complex scenarios.
    

### A.3 What Is Normalization?

The goals of normalization are to represent facts clearly, prevent redundancy/anomalies, and support integrity constraints.

#### First Normal Form

The table must be a proper relation and have **no repeating groups**.

- **The Problem:** Storing multiple values in one column (e.g., "printing, crash") or using multiple columns for the same attribute (e.g., `tag1`, `tag2`, `tag3`).
    
- **The Solution:** Create a separate table where each value occupies a single row.
    

#### Second Normal Form

A table is in 2NF if it is in 1NF and has no partial key dependencies. This applies when using compound primary keys. Attributes must depend on the _whole_ key, not just part of it.

**The Problem:**  
In the SQL below, `coiner` (who coined the tag) depends only on the `tag`, not on the `bug_id`. Storing it here creates redundancy and potential anomalies.

```sql
CREATE TABLE BugsTags (
bug_id BIGINT NOT NULL,
tag VARCHAR(20) NOT NULL,
tagger BIGINT NOT NULL,
coiner BIGINT NOT NULL,
PRIMARY KEY (bug_id, tag),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (tagger) REFERENCES Accounts(account_id),
FOREIGN KEY (coiner) REFERENCES Accounts(account_id)
);
```

**The Solution:**  
Split the data. Move the attribute that depends only on part of the key (`coiner`) into its own table (`Tags`).

```sql
CREATE TABLE Tags (
tag VARCHAR(20) PRIMARY KEY,
coiner BIGINT NOT NULL,
FOREIGN KEY (coiner) REFERENCES Accounts(account_id)
);
CREATE TABLE BugsTags (
bug_id BIGINT NOT NULL,
tag VARCHAR(20) NOT NULL,
tagger BIGINT NOT NULL,
PRIMARY KEY (bug_id, tag),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (tag) REFERENCES Tags(tag),
FOREIGN KEY (tagger) REFERENCES Accounts(account_id)
);
```

#### Third Normal Form

A table is in 3NF if non-key attributes depend _only_ on the primary key (no transitive dependencies).

**The Problem:**  
Here, `assigned_email` depends on `assigned_to` (the engineer), not directly on the `bug_id`.

```sql
CREATE TABLE Bugs (
bug_id SERIAL PRIMARY KEY
-- . . .
assigned_to BIGINT,
assigned_email VARCHAR(100),
FOREIGN KEY (assigned_to) REFERENCES Accounts(account_id)
);
```

**The Solution:**  
Move the email address to the `Accounts` table. The `Bugs` table should only reference the `assigned_to` ID.

#### Boyce-Codd Normal Form

BCNF is a stricter version of 3NF. It requires that _every_ determinant (a column that determines the value of another) must be a candidate key. This usually applies when a table has multiple overlapping candidate keys.

#### Fourth Normal Form

4NF addresses **multivalued dependencies**. You should not store two independent many-to-many relationships in a single table.

**The Problem:**  
Trying to store reporters, assignees, and verifiers in one table forces a complex primary key and creates redundancy (e.g., if a bug has 1 reporter but 3 assignees, the reporter is repeated).

```sql
CREATE TABLE BugsAccounts (
bug_id BIGINT NOT NULL,
reported_by BIGINT,
assigned_to BIGINT,
verified_by BIGINT,
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
FOREIGN KEY (assigned_to) REFERENCES Accounts(account_id),
FOREIGN KEY (verified_by) REFERENCES Accounts(account_id)
);
```

**The Solution:**  
Split the independent relationships into separate tables.

```sql
CREATE TABLE BugsReported (
bug_id BIGINT NOT NULL,
reported_by BIGINT NOT NULL,
PRIMARY KEY (bug_id, reported_by),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
);
CREATE TABLE BugsAssigned (
bug_id BIGINT NOT NULL,
assigned_to BIGINT NOT NULL,
PRIMARY KEY (bug_id, assigned_to),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (assigned_to) REFERENCES Accounts(account_id)
);
CREATE TABLE BugsVerified (
bug_id BIGINT NOT NULL,
verified_by BIGINT NOT NULL,
PRIMARY KEY (bug_id, verified_by),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (verified_by) REFERENCES Accounts(account_id)
);
```

#### Fifth Normal Form

5NF deals with cases where cyclic dependencies cause redundancy. Facts about independent relationships should be isolated.

**The Problem:**  
This table tries to store which engineer is assigned to a bug, AND implicitly links the engineer to the product. It confuses the specific assignment with the general fact that "Engineer X works on Product Y."

```sql
CREATE TABLE BugsAssigned (
bug_id BIGINT NOT NULL,
assigned_to BIGINT NOT NULL,
product_id BIGINT NOT NULL,
PRIMARY KEY (bug_id, assigned_to),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (assigned_to) REFERENCES Accounts(account_id),
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

**The Solution:**  
Isolate the relationships. Record which engineer works on which product separately from which engineer is assigned to a specific bug.

```sql
CREATE TABLE BugsAssigned (
bug_id BIGINT NOT NULL,
assigned_to BIGINT NOT NULL,
PRIMARY KEY (bug_id, assigned_to),
FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
FOREIGN KEY (assigned_to) REFERENCES Accounts(account_id),
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
CREATE TABLE EngineerProducts (
account_id BIGINT NOT NULL,
product_id BIGINT NOT NULL,
PRIMARY KEY (account_id, product_id),
FOREIGN KEY (account_id) REFERENCES Accounts(account_id),
FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

#### Further Normal Forms

- **Domain-Key Normal Form (DKNF):** Ensures all constraints are logical consequences of domain and key definitions.
    
- **Sixth Normal Form (6NF):** Eliminates all join dependencies, typically used for tracking history (temporal data) where every attribute might change at different times.
    

### A.4 Common Sense

Normalization rules are not esoteric theories; they are "commonsense techniques" to reduce redundancy and ensure data consistency. They serve as a practical guide for designing better databases.