# Lesson 7 - CDB and Regular PDBs (In Which We Actually Build the Thing Instead of Just Describing It)

Chapter 2 moves from architecture to construction. You now know what a CDB is. This chapter covers how to create one, how to create PDBs inside it, what to think about before you start, and a few ways to accidentally shut down the entire database when you meant to close a single PDB. That last one is important.

By the end of this lesson, you should be able to:

- Create a CDB using a `CREATE DATABASE` statement with `ENABLE PLUGGABLE DATABASE`
- Create PDBs from the seed using `FILE_NAME_CONVERT` or Oracle Managed Files
- Switch between containers using `ALTER SESSION SET CONTAINER` or direct connections
- Open and close PDBs safely using the correct commands
- Describe the ADR structure for a CDB
- List the post-creation steps required after a CDB is built

---

## 1. Creation Methods

A CDB can be created three ways:

| Method | When to Use |
|--------|------------|
| **SQL (`CREATE DATABASE`)** | Maximum control; scripted automation; exam knowledge |
| **DBCA (Database Configuration Assistant)** | GUI-driven; also supports PDB creation, duplication, relocation in 19c |
| **OUI (Oracle Universal Installer)** | Installs software and optionally invokes DBCA to create the DB |

DBCA in 19c is significantly more capable than earlier versions — it can create CDBs, create PDBs, duplicate PDBs, duplicate CDBs, and relocate PDBs. For most production work, DBCA is the right choice. For the exam, understand the SQL approach.

---

## 2. Planning Before You Build

A CDB can host up to **10,000 services**. Before creating one, decide:

| Question | Why It Matters |
|----------|---------------|
| How many PDBs? | Each PDB needs at least one service |
| How many services per PDB? | Multiple services allow connection routing (e.g. read/write vs read-only) |
| Centralised backup from CDB or per-PDB? | Affects RMAN configuration and backup window planning |
| Which users are common, which are local? | Determines privilege management strategy |
| Which roles/privileges are granted across all PDBs? | Common grants vs PDB-specific grants |
| PDB startup strategy? | By default PDBs do not auto-open when the CDB starts |

Getting these decisions right before creation is much easier than retrofitting them later.

---

## 3. Creating a CDB with SQL

### Step 1 — Create the Init Parameter File

Minimum required parameters:

```ini
DB_NAME=cdb1
ENABLE_PLUGGABLE_DATABASE=TRUE
DB_CREATE_FILE_DEST=/opt/oracle/oradata    # If using OMF
```

### Step 2 — Start Instance to NOMOUNT

```bash
sqlplus / as sysdba
STARTUP NOMOUNT PFILE='/path/to/initcdb1.ora'
```

### Step 3 — CREATE DATABASE

**With explicit file paths:**

```sql
CREATE DATABASE cdb1
  USER SYS IDENTIFIED BY YourPassword
  USER SYSTEM IDENTIFIED BY YourPassword
  LOGFILE
    GROUP 1 ('/opt/oracle/oradata/cdb1/redo01a.log',
             '/opt/oracle/oradata/cdb1/redo01b.log') SIZE 100M,
    GROUP 2 ('/opt/oracle/oradata/cdb1/redo02a.log',
             '/opt/oracle/oradata/cdb1/redo02b.log') SIZE 100M
  CHARACTER SET AL32UTF8
  NATIONAL CHARACTER SET AL16UTF16
  EXTENT MANAGEMENT LOCAL
  DATAFILE '/opt/oracle/oradata/cdb1/system01.dbf'
    SIZE 700M AUTOEXTEND ON
  SYSAUX DATAFILE '/opt/oracle/oradata/cdb1/sysaux01.dbf'
    SIZE 550M AUTOEXTEND ON
  DEFAULT TEMPORARY TABLESPACE temp
    TEMPFILE '/opt/oracle/oradata/cdb1/temp01.dbf' SIZE 100M
  UNDO TABLESPACE undotbs1
    DATAFILE '/opt/oracle/oradata/cdb1/undotbs1.dbf'
    SIZE 200M AUTOEXTEND ON
  ENABLE PLUGGABLE DATABASE
  SEED FILE_NAME_CONVERT = ('/opt/oracle/oradata/cdb1/',
                             '/opt/oracle/oradata/cdb1/pdbseed/');
```

**With Oracle Managed Files (simpler):**

```sql
-- DB_CREATE_FILE_DEST set in parameter file
CREATE DATABASE cdb1
  USER SYS IDENTIFIED BY YourPassword
  USER SYSTEM IDENTIFIED BY YourPassword
  CHARACTER SET AL32UTF8
  EXTENT MANAGEMENT LOCAL
  ENABLE PLUGGABLE DATABASE;
-- Oracle creates all files under DB_CREATE_FILE_DEST automatically
```

### Step 4 — Run the Catalog Scripts

```sql
-- Build the data dictionary and PL/SQL packages
@$ORACLE_HOME/rdbms/admin/catcdb.sql
-- This runs catalog.sql, catproc.sql, and related scripts
-- Takes several minutes
```

### Step 5 — Post-Creation

```sql
-- Create SPFILE from PFILE for persistent parameter management
CREATE SPFILE FROM PFILE;
SHUTDOWN IMMEDIATE;
STARTUP;

-- Compile any invalid objects
@$ORACLE_HOME/rdbms/admin/utlrp.sql
```

---

## 4. Creating a PDB from the Seed

Once the CDB exists, create PDBs from `PDB$SEED`:

**With FILE_NAME_CONVERT (explicit path mapping):**

```sql
CREATE PLUGGABLE DATABASE pdb1
  ADMIN USER pdbadmin IDENTIFIED BY YourPassword
  FILE_NAME_CONVERT = ('/opt/oracle/oradata/ORCLCDB/pdbseed/',
                       '/opt/oracle/oradata/ORCLCDB/pdb1/');
```

**With Oracle Managed Files:**

```sql
CREATE PLUGGABLE DATABASE pdb1
  ADMIN USER pdbadmin IDENTIFIED BY YourPassword
  CREATE_FILE_DEST = '/opt/oracle/oradata/ORCLCDB/pdb1/';
```

**With session-level parameter (set before the CREATE):**

```sql
ALTER SESSION SET DB_CREATE_FILE_DEST = '/opt/oracle/oradata/ORCLCDB/pdb1/';

CREATE PLUGGABLE DATABASE pdb1
  ADMIN USER pdbadmin IDENTIFIED BY YourPassword;
```

### Open the PDB After Creation

New PDBs are created in `MOUNTED` state. Open and save the state so it survives restarts:

```sql
ALTER PLUGGABLE DATABASE pdb1 OPEN;
ALTER PLUGGABLE DATABASE pdb1 SAVE STATE;
```

---

## 5. FILE_NAME_CONVERT Parameters — Which One to Use

| Parameter | Scope | Use Case |
|-----------|-------|---------|
| `FILE_NAME_CONVERT` | In the CREATE statement | One-time path mapping for a specific PDB creation |
| `PDB_FILE_NAME_CONVERT` | Session or instance parameter | Default path conversion for PDB files being created |
| `DB_CREATE_FILE_DEST` | Session or instance parameter | Oracle Managed Files destination — Oracle names the files |
| `CREATE_FILE_DEST` | In the CREATE PLUGGABLE DATABASE statement | OMF destination specific to this PDB creation |

All of these can be combined. The priority order when multiple are set: parameter in the statement → session-level parameter → instance-level parameter.

---

## 6. Switching Between Containers

**Connect directly to a PDB:**

```sql
CONNECT sys/YourPassword@localhost:1521/ORCLPDB1 AS SYSDBA
-- or
CONNECT system/YourPassword@localhost:1521/ORCLPDB1
```

**Switch containers within an existing session (common user with privilege required):**

```sql
ALTER SESSION SET CONTAINER = ORCLPDB1;
SHOW CON_NAME;   -- confirms: ORCLPDB1
```

**Switch back to root:**

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;
```

**Check where you are (always run this when you're not sure):**

```sql
SHOW CON_NAME    -- container name
SHOW CON_ID      -- container ID number
```

---

## 7. Opening and Closing PDBs — The Safe Way

This is the section where people lose CDBs they did not intend to shut down.

### The Correct Commands

```sql
-- From CDB root — manage a named PDB:
ALTER PLUGGABLE DATABASE pdb1 OPEN;
ALTER PLUGGABLE DATABASE pdb1 CLOSE IMMEDIATE;

-- Open all PDBs at once:
ALTER PLUGGABLE DATABASE ALL OPEN;

-- From within the PDB — manage the current PDB (no name needed):
ALTER PLUGGABLE DATABASE OPEN;
ALTER PLUGGABLE DATABASE CLOSE IMMEDIATE;
```

### The Dangerous Alias

When you are connected to a PDB and run `SHUTDOWN IMMEDIATE`, Oracle translates it to `ALTER PLUGGABLE DATABASE CLOSE IMMEDIATE`. Convenient.

**But if you are connected to the CDB root** and run `SHUTDOWN IMMEDIATE`, Oracle shuts down the **entire CDB** — taking every PDB down with it.

> The recommendation: **always use the explicit PDB commands** (`ALTER PLUGGABLE DATABASE ...`) and never rely on `STARTUP`/`SHUTDOWN` when working in a PDB context. The translation alias exists for compatibility, not safety.

### Always Save PDB State

```sql
-- After opening a PDB, save its state so it reopens after CDB restart:
ALTER PLUGGABLE DATABASE pdb1 SAVE STATE;

-- Or open all and save all:
ALTER PLUGGABLE DATABASE ALL OPEN;
ALTER PLUGGABLE DATABASE ALL SAVE STATE;
```

Without `SAVE STATE`, PDBs return to `MOUNTED` state every time the CDB restarts.

### Startup Trigger (Alternative to SAVE STATE)

```sql
-- Create a trigger that opens all PDBs whenever the CDB starts
CREATE OR REPLACE TRIGGER open_all_pdbs
  AFTER STARTUP ON DATABASE
BEGIN
  EXECUTE IMMEDIATE 'ALTER PLUGGABLE DATABASE ALL OPEN';
END;
/
```

---

## 8. Viewing PDB Status

```sql
-- From CDB root:
SHOW PDBS

-- Equivalent query:
SELECT con_id, name, open_mode, restricted
FROM   v$pdbs
ORDER BY con_id;
```

Expected states: `READ WRITE`, `READ ONLY`, `MOUNTED`, `MIGRATE`.

---

## 9. CON_ID in Performance and Metadata Views

The `CON_ID` column in `V$` and `CDB_` views identifies which container an object or session belongs to:

```sql
-- Buffer cache contents by container
SELECT con_id, COUNT(*) AS blocks_cached
FROM   v$bh
GROUP BY con_id
ORDER BY con_id;

-- Locked objects by container
SELECT con_id, object_id, session_id, oracle_username
FROM   v$locked_object
ORDER BY con_id;

-- Active sessions by container
SELECT con_id, COUNT(*) AS sessions
FROM   v$session
WHERE  type = 'USER'
GROUP BY con_id
ORDER BY con_id;
```

---

## 10. Automatic Diagnostic Repository (ADR) Structure

There is **one ADR location for the entire CDB**. All CDB and PDB events are written to the same alert log.

### Directory Structure

```
$ORACLE_BASE/diag/rdbms/<db_name>/<instance_name>/
├── alert/        ← log.xml (XML alert log — used by EM tools)
├── cdump/        ← core dump files
├── incident/     ← one subdirectory per incident (ORA- errors)
├── incpkg/       ← incident packages for Oracle Support (zip files)
├── hm/           ← Health Monitor data
├── trace/        ← trace files + text alert log (alert_<SID>.log)
└── log/          ← DDL log, debug logging
```

For your lab VM: `$ORACLE_BASE/diag/rdbms/orclcdb/ORCLCDB/`

### Finding the ADR Path

```sql
SELECT name, value FROM v$diag_info ORDER BY name;
-- Shows all ADR paths including the text alert log location
```

### Key Files

| File | Location | Format | Used By |
|------|----------|--------|---------|
| Alert log (text) | `trace/alert_ORCLCDB.log` | Plain text | DBAs, `tail -f` |
| Alert log (XML) | `alert/log.xml` | XML | EM Express, Cloud Control |
| Trace files | `trace/*.trc` | Text | Oracle Support, ADRCI |
| Incident packages | `incpkg/` | ZIP | Oracle Support |

### Managing ADR with ADRCI

```bash
adrci

# Inside ADRCI:
SHOW HOMES                          -- list all ADR homes
SET HOMEPATH diag/rdbms/orclcdb/ORCLCDB
SHOW ALERT -TAIL 50                 -- last 50 lines of alert log
SHOW INCIDENT                       -- list all incidents
SHOW PROBLEM                        -- list all problems (groups of incidents)
```

---

## 11. Tool Capabilities Summary (Updated for 19c)

| Tool | CDB Create | PDB Create | PDB Duplicate | PDB Relocate | Performance | Backup/Recovery |
|------|-----------|-----------|--------------|--------------|-------------|----------------|
| **SQL*Plus** | ✅ | ✅ | ✅ | ✅ | Limited | Via RMAN |
| **DBCA** | ✅ | ✅ | ✅ (new in 19c) | ✅ (new in 19c) | No | No |
| **OUI** | ✅ (via DBCA) | ✅ | No | No | No | No |
| **EM Cloud Control** | No¹ | ✅ | ✅ | ✅ | ✅ Full | ✅ Full |
| **EM Express** | No | Limited | No | No | ✅ Basic | No |
| **SQL Developer** | ✅ (via SQL) | ✅ | No | No | No | No |

> ¹ EM Cloud Control can duplicate an existing installation but cannot install Oracle software from scratch.

---

## 12. Wrap-Up (The Database Has Been Built. Now We Manage It.)

Key rules to carry forward:

1. **`ENABLE PLUGGABLE DATABASE`** in the `CREATE DATABASE` statement is what makes a CDB
2. **`catcdb.sql`** must be run after creation to build the catalog
3. **`FILE_NAME_CONVERT`** or **`DB_CREATE_FILE_DEST`** are required when creating PDBs — Oracle needs to know where to put the files
4. **`SAVE STATE`** or a startup trigger is required for PDBs to auto-open after a CDB restart
5. **Never use `SHUTDOWN`/`STARTUP` when you think you are in a PDB** unless you have confirmed `SHOW CON_NAME` first
6. **One ADR location, one alert log** — both CDB and PDB events appear in the same file

Next up: the hands-on lab exercises for CDB and PDB creation.
