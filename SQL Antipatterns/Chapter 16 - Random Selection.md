#### 1 Objective: Fetch a Sample Row

The objective is to efficiently retrieve a random sample row from a dataset using SQL, avoiding a full data transfer to the application.

#### 2 Antipattern: Sort Data Randomly

The common antipattern involves sorting the entire dataset randomly and then selecting the first row. This is typically achieved using the `ORDER BY RAND()` clause combined with `LIMIT 1`.

```sql
SELECT * FROM Bugs ORDER BY RAND() LIMIT 1;
```

The problem with this approach is its poor performance, especially with large datasets. Sorting the entire table randomly is resource-intensive and doesn't scale well. It cannot benefit from indexes.

#### 3 How to Recognize the Antipattern

- The query to select a random sample performs well during development but slows down significantly as the database grows.
- Application requires loading all rows into the application to randomly pick one.

#### 4 Legitimate Uses of the Antipattern

This approach might be acceptable for small, fixed-size datasets where performance is not critical, such as selecting a random item from a list of a few options.

#### 5 Solution: In No Particular Order

Several alternative solutions exist to fetch a random row more efficiently than sorting the entire dataset.

- **Choose a Random Key Value Between 1 and MAX**

This method selects a random integer between 1 and the maximum primary key value and retrieves the corresponding row.

```sql
SELECT b1.* FROM Bugs AS b1
JOIN (SELECT CEIL(RAND() * (SELECT MAX(bug_id) FROM Bugs)) AS rand_id) AS b2
ON (b1.bug_id = b2.rand_id);
```

==This approach assumes contiguous primary key values without gaps.==

- **Choose Next Higher Key Value**

If there are gaps in the primary key sequence, this method selects the next available key greater than or equal to a random value.

```sql
SELECT b1.* FROM Bugs AS b1
JOIN (SELECT CEIL(RAND() * (SELECT MAX(bug_id) FROM Bugs)) AS bug_id) AS b2
WHERE b1.bug_id >= b2.bug_id
ORDER BY b1.bug_id
LIMIT 1;
```

This solves the problem of gaps but introduces bias, as key values following a gap are selected more frequently.

- **Get a List of All Key Values, Choose One at Random**

This approach retrieves all primary key values into the application and selects one randomly. It requires two queries and can be inefficient for large tables.

```js
async function getRandomBug(pdo) {
  try {
    const bug_id_list = await pdo.query("SELECT bug_id FROM Bugs");
    const rand = Math.floor(Math.random() * bug_id_list.length);
    const rand_bug_id = bug_id_list[rand].bug_id;
    const stmt = await pdo.prepare("SELECT * FROM Bugs WHERE bug_id = ?");
    await stmt.execute([rand_bug_id]);
    const rand_bug = await stmt.fetch();
    return rand_bug;
  } catch (error) {
    console.error("Error:", error);
    return null;
  }
}
```

- **Choose a Random Row Using an Offset**

This method counts the total number of rows and selects a random offset to retrieve a row. It's suitable when contiguous key values cannot be assumed and ensures each row has an even chance of being selected.

```js
async function getRandomBugByOffset(pdo) {
  try {
    const rand = "SELECT ROUND(RAND() * (SELECT COUNT(*) FROM Bugs)) as random_offset";
    const offsetResult = await pdo.query(rand);
    const offset = offsetResult.random_offset;

    const sql = "SELECT * FROM Bugs LIMIT 1 OFFSET ?";
    const stmt = await pdo.prepare(sql);
    await stmt.execute([offset]);
    const rand_bug = await stmt.fetch();
    return rand_bug;
  } catch (error) {
    console.error("Error:", error);
    return null;
  }
}
```

- **Proprietary Solutions**

Some databases have proprietary solutions (e.g., `TABLESAMPLE` in SQL Server, `SAMPLE` in Oracle).

These might offer optimized ways to fetch sample rows.
