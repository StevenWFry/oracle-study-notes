# Practice 2-1 - Exploring CDB Architecture and Structures

This practice is fully runnable on your personal lab VM (`ORCLCDB`). All queries below can be executed from SQL*Plus as SYSDBA connected to the CDB root. Run them in sequence to build a complete picture of how the CDB organises its structures.

---

## Setup — Connect and Format SQL*Plus Output

```bash
. oraenv    # ORACLE_SID=ORCLCDB
sqlplus / as sysdba
```

```sql
-- Set output formatting for readability
SET LINESIZE 150
SET PAGESIZE 50
COLUMN name         FORMAT A30
COLUMN member       FORMAT A60
COLUMN tablespace_name FORMAT A25
COLUMN file_name    FORMAT A60
COLUMN username     FORMAT A25
COLUMN role         FORMAT A30
COLUMN granted_role FORMAT A30
```

---

## 1. Verify OS Processes (Before Connecting)

From the OS shell, verify Oracle processes are running:

```bash
ps -ef | grep -i orcl | grep -v grep
```

Expected: background processes (`pmon`, `smon`, `lgwr`, `dbwr`, `ckpt`, `mman`, `pman`, etc.) all showing `ORCLCDB` in their name.

---

## 2. Database-Level Identity

```sql
-- Database name, CDB status, and CON_ID
-- CON_ID = 0 means this is a database-level structure
-- (control files and redo logs belong to the database, not to any container)
SELECT name, cdb, con_id FROM v$database;
```

Expected:
```
NAME      CDB CON_ID
--------- --- ------
ORCLCDB   YES      0
```

```sql
-- Instance name, status, and the database it belongs to
SELECT instance_name, status, con_id FROM v$instance;
```

Expected:
```
INSTANCE_NAME STATUS  CON_ID
------------- ------- ------
ORCLCDB       OPEN         0
```

> `CON_ID = 0` in `V$DATABASE` and `V$INSTANCE` means these objects describe the whole database, not any specific container. The instance itself is `CON_ID = 1` (`CDB$ROOT`) for session and object purposes.

---

## 3. Listener and Services

From the OS shell:

```bash
lsnrctl status
```

Look for:
- Listener port (usually `1521`)
- EM Express port (usually `5500`)
- Services registered: `ORCLCDB`, `ORCLPDB1`, `ORCLCDBXDB`

From SQL*Plus:

```sql
-- Services currently running in this instance
SELECT name, con_id, network_name
FROM   v$services
ORDER BY con_id, name;
```

| Service | What It Is |
|---------|-----------|
| `ORCLCDB` | Main CDB service — used to connect to `CDB$ROOT` |
| `ORCLPDB1` | PDB service — used to connect to `ORCLPDB1` |
| `ORCLCDBXDB` | XDB (XML DB / EM Express) service |
| `SYS$BACKGROUND` | Internal background process service |
| `SYS$USERS` | Internal user session service |

---

## 4. Container Overview

```sql
-- Where are we connected right now?
SHOW CON_NAME
SHOW CON_ID

-- All PDBs and their current state
SHOW PDBS

-- Equivalent query with more detail
SELECT con_id, name, open_mode, restricted
FROM   v$pdbs
ORDER BY con_id;
```

Expected:
```
CON_ID NAME       OPEN_MODE  RESTRICTED
------ ---------- ---------- ----------
     2 PDB$SEED   READ ONLY  NO
     3 ORCLPDB1   READ WRITE NO
```

```sql
-- PDB DBID and name (each PDB has its own database ID)
SELECT con_id, name, dbid FROM v$pdbs ORDER BY con_id;
```

---

## 5. Database-Level Files (CON_ID = 0)

Redo logs and control files belong to `CON_ID = 0` — they are database-level structures, not owned by any container.

```sql
-- Redo log files
COLUMN member FORMAT A70
SELECT group#, member, con_id FROM v$logfile ORDER BY group#;
```

```sql
-- Control files
COLUMN name FORMAT A70
SELECT name, con_id FROM v$controlfile;
```

Both should show `CON_ID = 0`.

---

## 6. Data Files — CDB View vs DBA View

```sql
-- CDB_DATAFILES: all data files across ALL containers (from CDB root)
-- PDB$SEED (CON_ID=2) is not shown — Oracle hides it
COLUMN file_name FORMAT A65
SELECT con_id, tablespace_name, file_name
FROM   cdb_data_files
ORDER BY con_id, tablespace_name;
```

```sql
-- DBA_DATAFILES: only the CURRENT container (CDB root here)
-- No CON_ID column — it is implied by your connection
SELECT tablespace_name, file_name
FROM   dba_data_files
ORDER BY tablespace_name;
```

```sql
-- V$DATAFILE joined with V$TABLESPACE: all files including PDB$SEED
SELECT t.name AS tablespace_name, f.file#, f.con_id
FROM   v$datafile f
JOIN   v$tablespace t ON f.ts# = t.ts# AND f.con_id = t.con_id
ORDER BY f.con_id, t.name;
```

> Notice: `V$DATAFILE` shows `PDB$SEED` (CON_ID=2) but `CDB_DATA_FILES` does not. Oracle deliberately hides PDB$SEED from the CDB_ data dictionary views.

---

## 7. Temp Files

```sql
-- Temp files across all containers
COLUMN file_name FORMAT A65
SELECT con_id, tablespace_name, file_name
FROM   cdb_temp_files
ORDER BY con_id;
```

Expected containers:
- `CON_ID = 1` — CDB root TEMP
- `CON_ID = 2` — PDB$SEED TEMP (may or may not appear)
- `CON_ID = 3` — ORCLPDB1 TEMP

---

## 8. Users — Common vs Local

```sql
-- All users across all containers
SELECT con_id, username, common, account_status
FROM   cdb_users
WHERE  con_id != 2          -- exclude PDB$SEED
ORDER BY con_id, common DESC, username;
```

```sql
-- Common users only (visible in all containers, con_id=1 row is authoritative)
SELECT username, account_status, default_tablespace
FROM   cdb_users
WHERE  common = 'YES'
AND    con_id = 1
ORDER BY username;
-- Expected: ~39 users (SYS, SYSTEM, DBSNMP, GSMADMIN_INTERNAL, etc.)
```

```sql
-- Local users only (exist only within their specific PDB)
SELECT con_id, username, account_status
FROM   cdb_users
WHERE  common = 'NO'
ORDER BY con_id, username;
-- Expected: HR and PDBADMIN in CON_ID=3 (ORCLPDB1)
```

```sql
-- Confirm: no local users exist in CDB root (they cannot)
SELECT username FROM dba_users WHERE common = 'NO';
-- Expected: no rows
```

---

## 9. Roles — Common vs Local

```sql
-- All roles across containers, showing common status
COLUMN role FORMAT A35
SELECT con_id, role, common
FROM   cdb_roles
WHERE  con_id != 2
ORDER BY con_id, role;
```

```sql
-- Local roles in any PDB (none by default on a fresh install)
SELECT con_id, role
FROM   cdb_roles
WHERE  common = 'NO'
ORDER BY con_id, role;
```

---

## 10. Privileges — CDB-Wide View

```sql
-- System privilege map (reference: all possible system privileges)
DESCRIBE sys.system_privilege_map
SELECT privilege, name FROM sys.system_privilege_map ORDER BY name;
```

```sql
-- System privileges granted across all containers
SELECT grantee, privilege, admin_option, common, con_id
FROM   cdb_sys_privs
WHERE  con_id != 2
AND    grantee NOT IN ('SYS','SYSTEM')   -- filter noise
ORDER BY con_id, grantee, privilege
FETCH FIRST 30 ROWS ONLY;
```

```sql
-- Table privileges across all containers
SELECT grantee, owner, table_name, privilege, common, con_id
FROM   cdb_tab_privs
WHERE  con_id != 2
ORDER BY con_id, grantee
FETCH FIRST 20 ROWS ONLY;
```

```sql
-- Role grants across all containers
COLUMN granted_role FORMAT A35
SELECT grantee, granted_role, admin_option, common, con_id
FROM   cdb_role_privs
WHERE  con_id != 2
ORDER BY con_id, grantee, granted_role
FETCH FIRST 20 ROWS ONLY;
```

---

## 11. Buffer Cache — Which Container Owns What

```sql
-- Buffer cache contents by container (useful for tracking PDB memory usage)
SELECT con_id, COUNT(*) AS buffers_cached
FROM   v$bh
GROUP BY con_id
ORDER BY con_id;
```

---

## 12. Summary of CON_ID Values Observed

| CON_ID | Owner | Notes |
|--------|-------|-------|
| `0` | Database (whole CDB) | Control files, redo logs, `V$DATABASE`, `V$INSTANCE` |
| `1` | `CDB$ROOT` | Instance, CDB-level tablespaces, common users and roles |
| `2` | `PDB$SEED` | Hidden from most CDB_ views; visible in `V$` views |
| `3` | `ORCLPDB1` | Local tablespaces, local users (HR, PDBADMIN) |

---

## 13. Wrap-Up (Everything You Just Queried, In One Rule)

The pattern for every query in this practice:

- **`V$` views from CDB root** → show everything including PDB$SEED
- **`CDB_` views from CDB root** → show everything except PDB$SEED
- **`DBA_` views from CDB root** → show CDB root objects only, no CON_ID column
- **`CDB_` or `DBA_` views from within a PDB** → show that PDB's objects only

When in doubt: `SHOW CON_NAME` before querying. Know where you are before interpreting results.
