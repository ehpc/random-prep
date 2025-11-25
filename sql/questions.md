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

