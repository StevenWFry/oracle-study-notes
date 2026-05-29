# Practice 1-3 - Exploring CDB and PDB Using Enterprise Manager Cloud Control (In Which We Click Through Dashboards That You Do Not Have, But the SQL Underneath Them Is Yours)

This practice uses **Oracle Enterprise Manager Cloud Control** to explore CDB and PDB structure, performance, users, and tablespaces through a GUI. The personal lab does not have Cloud Control — but every screen shown in this practice has a direct SQL*Plus equivalent, and several are also available in **EM Express**. This document covers both.

---

## 1. What This Practice Covers

| EMCC Navigation | What It Shows |
|-----------------|--------------|
| Database Homepage (ORCL) | CDB overview, performance summary, services, containers graph |
| Administration → Storage → Tablespaces | All tablespaces across CDB root and PDBs (PDB$SEED excluded) |
| Performance → Performance Hub → Performance Home | Load, ASH, SQL monitoring, IO, parallel, services, PDB resource consumption |
| Performance → Top Activities | Top SQLs and top sessions |
| Security → Users | Users by container — common users and local PDB users |
| PDB Homepage (PDB1) | PDB-specific view: only that PDB's data |
| PDB → Performance Hub → ASH Analysis | Active session history scoped to a single PDB |
| PDB → Tablespaces | Only that PDB's tablespaces |

**Key observation from the practice:** When connected at the CDB level, EMCC shows resources and objects across all containers. When you navigate into a PDB, every view scopes down to that PDB only. This mirrors exactly how `CDB_` vs `DBA_` views and `V$` views behave depending on your connection point — the GUI is just a wrapper around the same logic.

**PDB$SEED is never shown in EMCC** — it is Oracle-managed and read-only. EMCC hides it the same way `CDB_TABLESPACES` queries typically filter it out.

---

## 2. Personal Lab Equivalents — SQL*Plus

### CDB Overview

```sql
-- Connect to CDB root
sqlplus / as sysdba

-- Database identity and CDB status
SELECT name, db_unique_name, cdb, open_mode, con_id FROM v$database;

-- All containers and their status
SELECT con_id, name, open_mode, restricted FROM v$pdbs ORDER BY con_id;
```

### Tablespaces — All Containers (EMCC: Administration → Storage → Tablespaces)

```sql
-- All tablespaces across all containers (from CDB root)
-- PDB$SEED (CON_ID=2) typically filtered out in tools
SELECT con_id, tablespace_name, status, contents
FROM   cdb_tablespaces
WHERE  con_id != 2          -- exclude PDB$SEED
ORDER BY con_id, tablespace_name;
```

```sql
-- Tablespace sizes with data files
SELECT c.con_id,
       c.tablespace_name,
       ROUND(SUM(d.bytes)/1048576, 1) AS size_mb,
       c.status
FROM   cdb_tablespaces c
JOIN   cdb_data_files d USING (tablespace_name, con_id)
WHERE  c.con_id != 2
GROUP BY c.con_id, c.tablespace_name, c.status
ORDER BY c.con_id, c.tablespace_name;
```

### Users — By Container (EMCC: Security → Users)

```sql
-- All users across all containers — shows common vs local
SELECT con_id,
       username,
       common,
       account_status,
       default_tablespace
FROM   cdb_users
WHERE  con_id != 2                          -- exclude PDB$SEED
ORDER BY con_id, common DESC, username;
```

```sql
-- Common users only (visible in all containers)
SELECT username, account_status, default_tablespace
FROM   cdb_users
WHERE  common = 'YES' AND con_id = 1
ORDER BY username;

-- Local users in a specific PDB (e.g. HR in PDB1)
SELECT username, account_status
FROM   cdb_users
WHERE  common = 'NO'
ORDER BY con_id, username;
```

### Active Sessions (EMCC: Top Activities → Top Sessions)

```sql
-- Current active user sessions across all containers
SELECT sid, serial#, username, status, con_id, machine, program
FROM   v$session
WHERE  type = 'USER'
AND    status = 'ACTIVE'
ORDER BY con_id, username;

-- All user sessions (active and inactive)
SELECT sid, serial#, username, status, con_id
FROM   v$session
WHERE  type = 'USER'
ORDER BY con_id;
```

### Top SQL (EMCC: Top Activities → Top SQLs)

```sql
-- Top 10 SQL statements by elapsed time
SELECT sql_id,
       con_id,
       ROUND(elapsed_time/1000000, 2)  AS elapsed_sec,
       ROUND(cpu_time/1000000, 2)      AS cpu_sec,
       executions,
       ROUND(elapsed_time/GREATEST(executions,1)/1000000, 4) AS elapsed_per_exec,
       SUBSTR(sql_text, 1, 80)         AS sql_text
FROM   v$sql
WHERE  executions > 0
ORDER BY elapsed_time DESC
FETCH FIRST 10 ROWS ONLY;
```

### ASH — Active Session History (EMCC: Performance Hub → ASH Analysis)

```sql
-- Last hour of active session history, summarised
SELECT TRUNC(sample_time, 'MI') AS sample_minute,
       con_id,
       session_state,
       COUNT(*) AS active_samples
FROM   v$active_session_history
WHERE  sample_time > SYSDATE - INTERVAL '1' HOUR
GROUP BY TRUNC(sample_time, 'MI'), con_id, session_state
ORDER BY sample_minute, con_id;

-- Top wait events from ASH
SELECT event,
       con_id,
       COUNT(*) AS waits
FROM   v$active_session_history
WHERE  sample_time > SYSDATE - INTERVAL '1' HOUR
AND    session_state = 'WAITING'
GROUP BY event, con_id
ORDER BY waits DESC
FETCH FIRST 10 ROWS ONLY;
```

### PDB-Scoped View — Connect to a PDB and Repeat

```sql
-- Connect directly to the PDB
CONNECT sys/YourPassword@localhost:1521/ORCLPDB1 AS SYSDBA

-- Now DBA_ and V$ views are scoped to ORCLPDB1 only
SELECT tablespace_name, status, contents FROM dba_tablespaces;
SELECT username, common, account_status FROM dba_users ORDER BY username;
SELECT sid, serial#, username, status FROM v$session WHERE type = 'USER';
```

This is exactly what EMCC does when you navigate from the CDB homepage into a PDB homepage — it changes the connection context and all the views narrow accordingly.

---

## 3. Personal Lab — EM Express

EM Express provides a subset of what EMCC shows. Access it at:

```
https://localhost:5500/em
```

Available views relevant to this practice:

| EM Express Section | Equivalent to EMCC |
|-------------------|-------------------|
| **Configuration → Initialization Parameters** | Instance-level parameters |
| **Storage → Tablespaces** | Tablespaces (scoped to connected container) |
| **Security → Users** | Users (select container from dropdown) |
| **Performance → Performance Hub** | ASH, SQL monitoring, load chart |
| **Oracle Database → PDBs** | List of PDBs, open/close controls |

To switch between CDB-level and PDB-level views in EM Express, use the container selector at the top of the page.

---

## 4. Key Observations from This Practice

1. **PDB$SEED is always hidden in management tools** — it is read-only and Oracle-owned; no need to show it in tablespace or user lists.

2. **EMCC container-aware navigation** mirrors the SQL model exactly — at the CDB level you see everything via `CDB_` views and global `V$`; inside a PDB you see only that container's data via `DBA_` views and scoped `V$`.

3. **Performance data is container-aware** — ASH, Top SQL, and session data all include a `CON_ID` column when viewed from the CDB root, allowing you to identify which PDB is driving load.

4. **Common users appear in every container's user list** — this is expected. `SYS`, `SYSTEM`, and any `C##` users you created will show up in both the CDB root view and the PDB view.

---

## 5. Wrap-Up (The GUI Is a Window. SQL*Plus Is the Room.)

Everything EMCC shows in these screens is queryable directly from SQL*Plus. The GUI adds graphs, drill-down navigation, and point-and-click administration — but when Cloud Control is unavailable or you need precise control, the SQL layer is always there.

For your personal lab, the SQL equivalents in this document cover all the same ground as the EMCC walkthrough. Run them from `CDB$ROOT` for the cross-container view and from within `ORCLPDB1` for the PDB-scoped view.

Next up: creating and configuring CDBs and PDBs.
