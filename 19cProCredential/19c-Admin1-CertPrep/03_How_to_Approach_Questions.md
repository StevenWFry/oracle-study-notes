# Lesson 3 - How to Approach Questions (In Which We Learn That Oracle Is Not Trying to Trick You — It Is Just Testing Whether You Actually Know Things)

This module walks through nine sample exam questions with full explanations. More importantly, it lays out the strategy for surviving the exam — not just the content, but the approach. Because knowing Oracle Database architecture and knowing how to read an Oracle exam question are two related but distinct skills.

By the end of this lesson, you should be able to:

- Apply a question strategy when you are unsure of an answer
- Recognise the "pairs" pattern Oracle uses to defeat guessing
- Work through the reasoning behind each sample question
- Identify the most common traps in each topic area

---

## 1. Exam Survival Strategy

### Flag and Continue

If you cannot answer a question immediately: **do not get stuck**. Flag it, move on, and come back. Running out of time on question 14 because you were wrestling with question 8 is an own goal.

### The Pairs Pattern (Why Guessing Doesn't Work)

Oracle deliberately constructs options in pairs where similar-looking answers have opposite implications. They use three flavours of pairs:

| Pair Type | What It Means |
|-----------|--------------|
| **Neither is true** | Both look plausible; neither is correct |
| **Only one is true** | The classic trap — they look equivalent but one has a subtle flaw |
| **Both are true** | You need to select both or you lose points |

This is explicitly designed to defeat guessing strategies. If you assume "one of this pair must be right," you will get burned. Know the material, then pick.

### Two Question Types

- **Learning style:** Testing facts, definitions, components — "what is X" or "what does Y do"
- **Scenario style:** A situation is described, you answer based on context — closer to real DBA work

Oracle tests **recognition and application**, not memorisation. You need to understand concepts well enough to reason through unfamiliar scenarios, not recite parameter names.

---

## 2. Sample Question 1 — Server Architecture (Sessions, Processes, and the UGA)

**Which two statements are true about Oracle database server architecture?**

```
A. A server process represents the state of a user's login to an instance.
B. A server process is always associated with a session.
C. Each session has its own User Global Area (UGA).
D. A session represents the state of a user's login to an instance.
E. The entire data dictionary is always cached in the large pool.
F. The entire data dictionary is always cached in the shared pool.
```

### Analysis

**E and F** are a *neither-is-true* pair. The data dictionary is **never fully cached** anywhere. Only selected rows are cached in the shared pool. Cross both off immediately.

**A and D** are opposites on a subtle point. In shared server architecture, there is **no dedicated server process** tied to your session. The *session* holds the state of the user's login — not the server process. So:
- A → **False** (it's the session, not the server process)
- D → **True**

**C** — Each session stores its state in the **User Global Area (UGA)**. In dedicated server mode, the UGA is part of the PGA. In shared server mode, it moves into the **SGA large pool** so multiple server processes can access it. Either way, each session has its own UGA.
- C → **True**

**B** — In shared server architecture, sessions are served by whichever shared server process is available. There is no permanent server process attached to a given session.
- B → **False**

> ✅ **Correct answers: C and D**

---

## 3. Sample Question 2 — Shutdown IMMEDIATE

**Which two statements are true about Oracle database during SHUTDOWN IMMEDIATE?**

```
A. New connection requests are unsuccessful.
B. Uncommitted transactions are rolled back automatically.
C. All connections remain until all transactions issue ROLLBACK or COMMIT.
D. Uncommitted transactions are committed automatically.
E. It does not force a checkpoint.
```

### Analysis

Every shutdown mode blocks new connections from the moment it begins. **A is always true** — you don't need to memorise this per-mode, it's universal.

`SHUTDOWN IMMEDIATE` does **not** wait for users to disconnect or for transactions to finish. It:
- Rolls back all uncommitted transactions — **B is true**
- Does **not** wait for users (that's `SHUTDOWN NORMAL`) — **C is false**
- Does **not** auto-commit transactions (that would be chaos) — **D is false**
- Forces a **full checkpoint** before closing, so no instance recovery is needed at next startup — **E is false**

> ✅ **Correct answers: A and B**

**Quick shutdown mode reference:**

| Mode | Waits for sessions? | Waits for transactions? | Forces checkpoint? | Instance recovery needed? |
|------|--------------------|-----------------------|-------------------|--------------------------|
| NORMAL | Yes | Yes | Yes | No |
| TRANSACTIONAL | No | Yes | Yes | No |
| **IMMEDIATE** | **No** | **No (rolls back)** | **Yes** | **No** |
| ABORT | No | No | No | **Yes** |

---

## 4. Sample Question 3 — Row Migration vs Row Chaining

**Which three statements are true about Oracle database block space management?**

```
A. A row in one extent can be migrated to a block in a different extent.
B. An INSERT statement can result in a migrated row.
C. An UPDATE statement cannot cause chained rows to occur.
D. A row in one extent can be migrated to a block in the same extent.
E. An INSERT statement can result in a chained row.
F. A migrated row is never chained.
```

### Analysis

First, know your terms:

- **Row migration** — an `UPDATE` makes a row too large for its current block. Oracle moves the *entire row* to a new block. The original block keeps a pointer. The row fits in the new block as a single piece.
- **Row chaining** — a row is too large to fit in *any single block*. It is split across multiple blocks and stored in pieces.

**A and D** are a *both-are-true* pair. When Oracle migrates a row, it goes wherever space is available — same extent or different extent. The exam is checking whether you think there's a constraint. There isn't. Both are true.

**B** — `INSERT` cannot cause migration. Migration only happens when an existing row grows via `UPDATE`. Insert can cause chaining (if the new row is immediately too large for a block), but not migration.
- B → **False**

**C** — `UPDATE` absolutely can cause chaining. If an `UPDATE` grows a row beyond the size of any single block, the row chains across multiple blocks.
- C → **False**

**E** — If you `INSERT` a row that is too long to fit in any single block, it is chained immediately at insert time. B and E are a *one-is-true* pair — only E survives.
- E → **True**

**F** — A migrated row can later be updated to grow even larger than any block, at which point it also becomes chained. Migration and chaining are not mutually exclusive.
- F → **False**

> ✅ **Correct answers: A, D, and E**

---

## 5. Sample Question 4 — Transaction Commits (DDL Auto-Commit and Shutdown Behaviour)

**A session has an in-flight transaction that has updated 5,000 rows. Private temporary tables are not used. In which three situations does the transaction commit?**

```
A. When a DDL statement is executed successfully by the same user in a different session.
B. When a DDL statement is executed successfully by the same user in the same session.
C. When a DML statement is executed successfully by the same user in a different session.
D. When a DML statement is executed successfully by the same user in the same session.
E. When a DBA issues SHUTDOWN NORMAL and the session terminates normally.
F. When a DBA issues SHUTDOWN TRANSACTIONAL and the session terminates normally.
```

### Analysis

The key rule: **DDL in the same session causes an implicit commit** of any pending DML. This does *not* apply across sessions. The "private temporary tables" note is a hint — DDL on private temp tables is an exception because they are memory-only structures, but that exception doesn't apply here.

- **A** — DDL in a *different* session does not affect this session's transaction. → **False**
- **B** — DDL in the *same* session triggers implicit commit. → **True**

DML never auto-commits anything:
- **C** → **False**
- **D** → **False**

For the shutdown questions: `SHUTDOWN NORMAL` waits for all sessions to disconnect voluntarily. `SHUTDOWN TRANSACTIONAL` waits for in-flight transactions to complete. In both cases, when the session terminates normally, the transaction commits.

- **E** → **True**
- **F** → **True**

> ✅ **Correct answers: B, E, and F**

---

## 6. Sample Question 5 — Index Administration (Invisible vs Unusable)

**Which two statements are true about indexes and their administration?**

```
A. An index can be scanned to satisfy a query without the indexed table being accessed.
B. A non-unique index can be converted to unique using DDL.
C. An index is always rebuilt using the existing index as the source for keys and row IDs.
D. An invisible index is maintained when DML is performed on its associated table.
E. An unusable index is maintained when DML is performed on its associated table.
```

### Analysis

**A** — This is called **index-only access** (or index fast full scan for coverage). If the query's projection and filter columns are all present in the index, *and* NULL values are excluded, Oracle can satisfy the query from index blocks alone without touching the table. → **True**

**B** — `ALTER INDEX` does not have a clause to change uniqueness. You cannot convert a non-unique index to a unique one via DDL. Drop and recreate. → **False**

**C** — If an index is marked **unusable**, Oracle cannot use it as the source for a rebuild. It scans the table instead. The "always" makes C false. → **False**

**D and E** — This is the key distinction to know cold:

| Index State | Used by optimizer? | Maintained by DML? | Has a segment? |
|-------------|-------------------|-------------------|---------------|
| **Invisible** | No (unless `OPTIMIZER_USE_INVISIBLE_INDEXES=TRUE`) | **Yes** | Yes |
| **Unusable** | No | **No** | Usually no |

- **D** → **True** — invisible indexes are maintained, just ignored by the optimiser
- **E** → **False** — unusable indexes are not maintained and usually have no segment

> ✅ **Correct answers: A and D**

---

## 7. Sample Question 6 — Sequences

**Which three statements are true about sequences in a single-instance Oracle database?**

```
A. A sequence that starts with 1 and increments by 1 can never have gaps.
B. A sequence can issue the same number multiple times.
C. Sequence numbers that are allocated are rolled back if the transaction fails.
D. A sequence can provide numeric values for more than one column.
E. A sequence defined with CACHE 1000000 stores 1,000,000 values in the row cache.
F. A sequence can provide numeric values for more than one table.
```

### Analysis

**A** — Even simple 1-increment sequences can have gaps. If a transaction selects a sequence number and then rolls back, that number is gone. Instance crashes also discard the cached range. → **False**

**B** — Sequences can be defined with `CYCLE`. When they reach `MAXVALUE`, they wrap back to `MINVALUE` and start issuing numbers they've used before. → **True**

**C** — Sequence numbers are **independent of transactions**. Rolling back a transaction does not return the sequence value. This is intentional — sequences are designed to be gapless in successful operation, not rollback-aware. → **False**

**D and F** — A sequence is not bound to any table or column. Multiple tables, multiple columns, multiple applications can all draw from the same sequence. (Oracle internally uses sequences to back identity columns, but that's a specific case — in general, sequences are free agents.) → Both **True**

**E** — Oracle does not store all cached values in memory. It stores the **last generated value** and a **range**. `CACHE 1000000` means Oracle pre-allocates a range of 1 million values, but they aren't all sitting in the row cache waiting. → **False**

> ✅ **Correct answers: B, D, and F**

---

## 8. Sample Question 7 — CREATE TABLE AS SELECT (Scenario Style)

**A table SALES has columns PRODUCT_ID, CUSTOMER_ID, TIME_ID, CHANNEL_ID, PROMO_ID, QUANTITY_SOLD, PRICE, AMOUNT_SOLD. All columns except PRICE have NOT NULL constraints. The table has no primary key and has 55,000 rows.**

```sql
CREATE TABLE mysales (prod_id, cust_id, quantity_sold, price)
AS SELECT product_id, customer_id, quantity_sold, price
   FROM sales
   WHERE 1 = 2;
```

**Which two statements are true?**

```
A. mysales is created with no rows.
B. mysales is created with no constraints.
C. mysales has NOT NULL constraints on the first three columns.
D. mysales fails because 1 cannot equal 2.
E. mysales is created with 55,000 rows because WHERE 1=2 is ignored.
```

### Analysis

`WHERE 1 = 2` is a classic technique for creating a table structure with no data. It is a valid — if always-false — predicate. The table is created; it just has zero rows.

- **D** → **False** — the statement executes successfully
- **E** → **False** — the WHERE clause is evaluated and controls row count; it is not ignored
- **A** → **True** — no rows because `1=2` is never true

For constraints: `CREATE TABLE AS SELECT` **inherits NOT NULL constraints** from the source table, *unless* those NOT NULL constraints derive from a primary key (in which case they are not copied). The question explicitly states SALES has **no primary key** — so the NOT NULL constraints on `PRODUCT_ID`, `CUSTOMER_ID`, and `QUANTITY_SOLD` (all NOT NULL in the source) transfer to the new table. `PRICE` was nullable in the source, so it stays nullable.

- **B** → **False** — NOT NULL constraints are inherited from non-PK sources
- **C** → **True** — the first three columns carry their NOT NULL constraints over

> ✅ **Correct answers: A and C**

---

## 9. Sample Question 8 — Non-Standard Block Size Tablespace

**You plan to create:**

```sql
CREATE TABLESPACE etl_tbs
  DATAFILE '/u03/oracle/data/etl_01.dbf' SIZE 500M
  BLOCKSIZE 16K;
```

**The standard database block size is 8K. The /u03 filesystem is on a storage array. No subdirectories exist under /flash.**

**Which two requirements must be met?**

```
A. DB_CACHE_SIZE must be set to 16K.
B. The /u03 filesystem must use a 16K block size.
C. DB_16K_CACHE_SIZE must be set to a non-zero value.
D. DB_16K_CACHE_SIZE must be set to a value greater than 500MB.
E. The /u03 filesystem must exist and be mounted.
```

### Analysis

**A** — `DB_CACHE_SIZE` is the cache for the **standard** block size (8K here). It has nothing to do with 16K tablespaces. → **False**

**B** — The filesystem block size has no relationship to the Oracle block size. Oracle manages its own blocks within the datafile. → **False**

**C** — This is the key rule: for every non-standard block size tablespace in use, the corresponding `DB_nK_CACHE_SIZE` parameter **must be set to a non-zero value**. Without `DB_16K_CACHE_SIZE` configured, the tablespace cannot be created. → **True**

**D** — The cache does not need to be as large as the datafile. It just needs to be non-zero. → **False**

**E** — The filesystem must physically exist and be mounted at `/u03` before Oracle can write a datafile there. It will not create mount points for you. → **True**

> ✅ **Correct answers: C and E**

---

## 10. Sample Question 9 — Table Compression for DSS Workloads

**You need compression for a DSS workload with analytic query access, requiring the greatest compression with the least CPU. Which option meets this requirement?**

```
A. ROW STORE COMPRESS BASIC
B. COLUMN STORE COMPRESS FOR ARCHIVE HIGH
C. ROW STORE COMPRESS ADVANCED
D. COLUMN STORE COMPRESS FOR QUERY LOW
E. COLUMN STORE COMPRESS FOR ARCHIVE LOW
F. COLUMN STORE COMPRESS FOR QUERY HIGH
```

### Analysis

Work through the requirements as filters:

**Step 1 — DSS + analytic queries → Column Store**
Row store compression (A, C) is for OLTP. DSS analytic workloads use **column store** (Hybrid Columnar Compression / HCC). Eliminate A and C.

**Step 2 — Good query performance → FOR QUERY, not FOR ARCHIVE**
`FOR ARCHIVE` options maximise compression at the cost of query performance. Since the question requires *good query performance*, use `FOR QUERY`. Eliminate B, E.

**Step 3 — Least CPU → LOW**
Between `FOR QUERY LOW` and `FOR QUERY HIGH`: HIGH gives more compression but uses more CPU. The requirement is **least CPU**, so LOW wins.

- D → `COLUMN STORE COMPRESS FOR QUERY LOW` → **Correct**
- F → `COLUMN STORE COMPRESS FOR QUERY HIGH` → More CPU, eliminated

> ✅ **Correct answer: D**

---

## 11. Answer Key

| Question | Topic | Correct Answers |
|----------|-------|----------------|
| 1 | Server architecture: sessions, UGA, PGA | C, D |
| 2 | SHUTDOWN IMMEDIATE behaviour | A, B |
| 3 | Row migration vs chaining | A, D, E |
| 4 | Transaction commits: DDL, shutdown modes | B, E, F |
| 5 | Index types: invisible vs unusable | A, D |
| 6 | Sequences: gaps, cycles, independence | B, D, F |
| 7 | CREATE TABLE AS SELECT constraints | A, C |
| 8 | Non-standard block size tablespace requirements | C, E |
| 9 | Table compression: DSS, column store, CPU | D |

---

## 12. Wrap-Up (You Now Know What You Are Walking Into)

Nine questions, nine topics, and a clear view of how Oracle constructs the exam. The recurring themes:

- **Know your pairs.** E/F pairs are often a tell that neither is true; A/D pairs often test a single subtle distinction.
- **Read the qualifiers.** Words like "always," "never," "any," and "same session vs different session" are doing a lot of work.
- **Scenario questions filter by requirement.** Work through each constraint in order and eliminate as you go — the Q9 compression approach is the template.
- **Invisible ≠ Unusable.** One is maintained, one is not. Know this cold.
- **DDL commits in the same session. DML does not.**

Next module: the exam content proper begins.
