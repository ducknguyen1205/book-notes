##### 10.1 Objective: Use Fractional Numbers Instead of Integers

The goal is to store non-integer numeric values and perform accurate arithmetic computations.

##### 10.2 Antipattern: Use FLOAT Data Type

The `FLOAT` data type, which encodes real numbers in a binary format according to IEEE 754, can lead to rounding issues. Not all decimal values can be precisely stored in binary, causing necessary rounding.

- **Example of Rounding**: While a query might return `59.95`, the actual stored value may be slightly different.
    
```sql
SELECT hourly_rate FROM Accounts WHERE account_id = 123;
```

Returns: `59.95`

Magnifying the value reveals the discrepancy:

```sql
 SELECT hourly_rate * 1000000000 FROM Accounts WHERE account_id = 123;
```
    
Returns: `59950000762.939`
    
- **Equality Comparisons**: Due to rounding, direct equality comparisons may fail.

```sql
SELECT * FROM Accounts WHERE hourly_rate = 59.95;
```
    
Result: empty set; no rows match

A workaround involves using a threshold:

```sql
SELECT * FROM Accounts WHERE ABS(hourly_rate - 59.95) < 0.000001;
```
    
- **Cumulative Impact**: Summing many `FLOAT` values can accumulate discrepancies.
    
```sql
SELECT SUM( b.hours * a.hourly_rate ) AS project_cost
FROM Bugs AS b
JOIN Accounts AS a ON (b.assigned_to = a.account_id);
```

##### 10.3 How to Recognize the Antipattern

The use of `FLOAT`, `REAL`, or `DOUBLE PRECISION` is questionable.

##### 10.4 Legitimate Uses of the Antipattern

`FLOAT` is suitable for a greater range of values than `INTEGER` or `NUMERIC`.

##### 10.5 Solution: Use NUMERIC Data Type

Use `NUMERIC` or `DECIMAL` for fixed-precision fractional numbers. These store values exactly, up to the specified precision and scale.

```sql
ALTER TABLE Bugs ADD COLUMN hours NUMERIC(9,2);

ALTER TABLE Accounts ADD COLUMN hourly_rate NUMERIC(9,2);
```

- **Exact Value Storage**: Values are stored exactly as specified.
    
```sql
SELECT hourly_rate FROM Accounts WHERE hourly_rate = 59.95;
```
    
Returns: `59.95`
- **Magnification**: Scaling the value yields the expected result.
    
```sql
SELECT hourly_rate * 1000000000 FROM Accounts WHERE hourly_rate = 59.95;
```
    
Returns: `59950000000`