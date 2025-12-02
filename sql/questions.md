# What are the default isolation levels for Postgres, SQLite, Oracle DBMD, MySQL?

### Default Isolation Levels

#### PostgreSQL
**Default:** `READ COMMITTED`
- Each statement sees only rows committed before it began.
- Prevents dirty reads, but not non-repeatable reads or phantom reads.

#### SQLite
**Default:** `SERIALIZABLE` (but with caveats)
- SQLite uses SERIALIZABLE as the default, but concurrency is limited because writes lock the whole DB (unless WAL mode is enabled).
- No dirty reads; MVCC is minimal.

#### Oracle
**Default:** `READ COMMITTED`
- Oracle uses snapshot consistency for every statement.
- Prevents dirty and non-repeatable reads (stronger than standard READ COMMITTED).

#### MySQL (InnoDB)
**Default:** `REPEATABLE READ`
- MySQL’s REPEATABLE READ is non-standard; prevents phantoms via next-key locks.

---

### Comparison Table

| DB | Default isolation | Notes |
|----|-------------------|-------|
| PostgreSQL | `READ COMMITTED` | Classic MVCC, prevents dirty reads |
| SQLite | `SERIALIZABLE` | Limited concurrency, DB-level write locks |
| Oracle | `READ COMMITTED` | Snapshot-per-statement, strong READ COMMITTED |
| MySQL (InnoDB) | `REPEATABLE READ` | Non-standard, prevents phantom reads |


# What are the principles relational DBs should conform to?

## Codd’s 12 Rules (Principles of Relational Databases)

### Rule 0 — Foundation rule
A system qualifies as relational only if it can manage data entirely through relational means.

### Rule 1 — Information rule
All information is represented logically as values in tables (relations).

### Rule 2 — Guaranteed access rule
Every datum is accessible using a combination of table name, primary key, and column name.

### Rule 3 — Systematic treatment of nulls
Nulls represent missing or inapplicable information and must be handled uniformly.

### Rule 4 — Active online catalog
The database catalog (schema) is itself stored as tables and can be queried with the same language used for user data.

### Rule 5 — Comprehensive data sublanguage rule
There must be a single language (like SQL) for defining data, manipulating data, defining views, constraints, authorization, and transactions.

### Rule 6 — View updating rule
Views that are theoretically updatable must be updatable by the system.

### Rule 7 — High-level insert, update, delete
Set-based operations must be supported; SQL statements operate on sets, not individual records.

### Rule 8 — Physical data independence
Physical storage changes (indexes, files, optimization) must not affect applications.

### Rule 9 — Logical data independence
Changes to the logical schema (tables, views) must not break applications if semantics remain intact.

### Rule 10 — Integrity independence
Integrity constraints are stored in the catalog and enforced by the DBMS, not by application code.

### Rule 11 — Distribution independence
The system should behave the same whether data is distributed across multiple locations or centralized.

### Rule 12 — Non-subversion rule
No low-level access mechanism may bypass relational rules or constraints.


# What is the difference between cursor pagination and offset/limit pagination?

Offset/limit pagination uses OFFSET n LIMIT m to skip a number of rows 
and return the next batch.

Cursor-based pagination uses a stable value (a "cursor") from the last 
fetched record — often an ID or timestamp — to fetch the next set.


🔹 Offset / Limit Pagination

How it works:
The client requests pages by telling the database: “skip X rows, then return Y rows.”

```sql
SELECT * FROM orders ORDER BY created_at LIMIT 20 OFFSET 40;
```

Pros:
* Very simple to implement
* Works with any sorted dataset
* Easy to jump to arbitrary pages (page 100, etc.)

Cons:
* Slow for large offsets — the DB still walks/skips rows internally
* Not stable: if new data is inserted, pages shift, causing duplicates or missing rows
* Poor performance under high load
* Can break with concurrent writes (non-repeatable pagination)

🔹 Cursor-Based Pagination

How it works:
Instead of "page number", the API returns a token (cursor).
Usually this cursor contains the last record’s unique sort value (id, created_at, etc.).

```sql
SELECT * FROM orders
WHERE created_at > $cursor
ORDER BY created_at
LIMIT 20;
```

Pros:
* Highly performant — uses indexed range scans
* Constant-time page lookup, no matter how large the table grows
* Stable under concurrent inserts — no shifting pages
* Good choice for infinite-scroll
* High scalability (Twitter, Facebook, Instagram all use cursors)

Cons:
* Harder to implement
* Cannot easily “jump to page 100” — no absolute page numbers
* Requires stable sorting key (monotonic field like ID or timestamp)

