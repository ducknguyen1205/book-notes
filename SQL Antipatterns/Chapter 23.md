
## 23.1 Objective: Write Less Code

The goal is to reduce unnecessary code, but not at the expense of reliability. Writing less code is beneficial only if it does not remove essential error handling and diagnostics.

---

## 23.2 Antipattern: Making Bricks Without Straw

Also known as **See No Evil**. This antipattern occurs when developers:

- Ignore return values or exceptions from database APIs
    
- Debug SQL by reading application code instead of the SQL itself
### Diagnoses Without Diagnostics

Ignoring return values hides failures such as:

- Connection errors
    
- SQL syntax errors
    
- Constraint or permission violations
    

Result: crashes, blank screens, or confusing behavior instead of clear error messages.

### Lines Between the Reading

Debugging SQL by inspecting string-building code is error-prone.

Example bug:

```sql
SELECT * FROM BugsWHERE bug_id = 1234
```

The error is obvious in SQL, but hard to see in application code.

---

## 23.3 How to Recognize the Antipattern

Common signals:

- Program crashes after database queries
    
- Developers show code instead of the generated SQL
    
- Error handling is intentionally omitted
    

---

## 23.4 Legitimate Uses of the Antipattern

Ignoring errors is acceptable only when:

- There is no meaningful recovery action
    
- Exceptions are intentionally propagated to higher-level code
    

---

## 23.5 Solution: Recover from Errors Gracefully

Key practices:

- Always check return values and handle exceptions
    
- Fail fast with meaningful diagnostics
    
- Debug using the actual SQL, not the code that builds it
    
- Log SQL safely (logs, debugger, server-side logging)
    

**Core takeaway:**

> Less code is not better if it makes failures invisible. Visibility and recovery matter more than brevity.