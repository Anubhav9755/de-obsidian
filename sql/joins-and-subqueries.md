# Joins & Subqueries

## Anti-join

**What it is:**
Rows in table A that have NO matching row in table B.

**Why it matters:**
Common in DE for finding "records not yet processed" or "customers with no orders."

**Example:**
```sql
SELECT a.*
FROM orders a
LEFT JOIN shipments b ON a.order_id = b.order_id
WHERE b.order_id IS NULL;
```

**Gotchas:**
-

---

## Semi-join

**What it is:**
-

**Example:**
```sql

```

---

## Lateral join

**What it is:**
-

**Example:**
```sql

```

**Gotchas:**
-
