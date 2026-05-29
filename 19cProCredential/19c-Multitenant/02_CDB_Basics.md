# Lesson 2 - CDB Basics (In Which Oracle Puts a Database Inside a Database and Insists This Makes Things Simpler)

Before multitenant existed, the solution to "we have many applications with different security and tuning requirements" was "run many separate databases." This worked, in the same way that having a separate house for each of your houseplants technically works. It is not wrong, it is just a lot to manage.

The multitenant architecture solves this by introducing the **Container Database (CDB)** — a single database that hosts multiple **Pluggable Databases (PDBs)**, each with its own isolated metadata, its own users, its own tablespaces, and its own configuration. One instance. One set of files. Many virtual databases inside it. Think of it as VMs for databases — a single physical host divided into isolated environments that each believe they have the place to themselves.

By the end of this lesson, you should be able to:

- Explain the problem multitenant solves and why it was introduced
- Describe the components of a CDB and their roles
- Distinguish between the CDB root, CDB seed, and PDBs
- Understand how data dictionary views behave differently at the CDB and PDB level
- Explain the common vs local distinction for users, roles, and objects
- Identify key differences in how features like TDE, auditing, and Resource Manager operate across CDB and PDB scope

---

## 1. The Problem Multitenant Solves

In a traditional (non-CDB) Oracle database:

- One instance + one database = one application environment
- All Oracle metadata and all application metadata share the **same system tablespace**
- Multiple applications in one database = shared data dictionary = security and isolation nightmare
- Multiple applications in separate databases = separate instances, separate patching, separate backups, separate everything

The result: enterprise environments ended up managing dozens or hundreds of databases, each requiring independent maintenance cycles. Change management — upgrades, patching, migration — had to be planned and executed separately for each one.

Multitenant changes the model:

- One CDB hosts many PDBs
- Each PDB has **its own metadata tables** — its own virtual data dictionary
- Oracle provides isolation, security, and configuration separation between PDBs
- Management can be **centralised at the CDB** or **delegated to the PDB level**

> Non-CDB architecture is **desupported in Oracle 20c and later**. If you are still deploying standalone databases in 19c "for simplicity," you are accruing a migration debt that will come due.

---

## 2. CDB Architecture Components

A CDB has the following structural layers:

### The Instance (Shared by Everyone)
- SGA, PGA, background processes — the same as always
- All PDBs share the single instance
- Connection management, SQL execution, and maintenance processes operate here

### CDB-Level Files (Belong to the Container)
- **Control files** — structure of the whole database, RMAN schema, database events
- **Redo log files** — transaction consistency, crash recovery for all PDBs
- These files have `CON_ID = 0` — they belong to the database as a whole, not to any specific container

### CDB Root (`CDB$ROOT`, CON_ID = 1)
- Oracle-supplied metadata, utilities, and services for managing the whole container
- Has its own SYSTEM, SYSAUX tablespaces
- Has its own UNDO and TEMP (shared CDB undo, or per-PDB if local undo is enabled)
- **You do not put user data here.** The root is Oracle's management layer.

### CDB Seed (`PDB$SEED`, CON_ID = 2)
- A read-only template PDB maintained by Oracle
- Contains the base Oracle structure of a pluggable database (SYSTEM, SYSAUX, UNDO if local, TEMP)
- **Always read-only** — you cannot modify it directly
- Used as the source when creating new empty PDBs

### Pluggable Databases (CON_ID = 3+)
- Each PDB is a virtual database for one (or more) applications
- Has its own SYSTEM, SYSAUX tablespaces — its own data dictionary
- Has its own UNDO (local undo, default in 19c) and TEMP
- Users connect to PDB services — they never connect directly to the root
- PDBs are **isolated from each other by default** — they cannot see each other's data

```
CDB (ORCL)
├── Instance (shared SGA + background processes)
├── CDB$ROOT      (CON_ID=1) ← Oracle management layer
├── PDB$SEED      (CON_ID=2) ← Read-only template
├── PDB1          (CON_ID=3) ← Application A
├── PDB2          (CON_ID=4) ← Application B
└── PDB3          (CON_ID=5) ← Application C
```

---

## 3. Local Undo — Default in 19c

From 12.2 onwards, Oracle introduced **local undo mode** — each PDB manages its own undo tablespace rather than sharing the CDB undo. In 19c, **local undo is the default**.

Why it matters:
- Enables **hot cloning** and **PDB relocation** while the PDB is open
- Required for certain Flashback operations at the PDB level
- Better isolation between PDBs — one PDB's heavy transaction load doesn't eat another's undo space

If you are building a new CDB in 19c, local undo is already on. Do not turn it off unless you have a specific reason.

---

## 4. Building a CDB — The Sequence

The construction order matters:

1. **Create the CDB** — this creates `CDB$ROOT` and `PDB$SEED` automatically
   - You cannot convert a non-CDB into a CDB root; it must be created as one
2. **Create PDBs from the seed** — empty PDBs with the Oracle structure, ready to customise
3. **Add tablespaces and schema objects** to each PDB for its application
4. **Plug in existing non-CDBs** — convert a non-container database into a PDB and plug it into the CDB

---

## 5. Tools for Multitenant Management

All standard Oracle tools have been updated for multitenant:

| Tool | What You Can Do With It |
|------|------------------------|
| **SQL*Plus** | Everything — create CDBs, PDBs, manage parameters, run upgrade scripts |
| **DBCA** | Create and configure CDBs and PDBs; in 19c also supports duplication |
| **Oracle Universal Installer (OUI)** | Install software and create the initial CDB; uses DBCA underneath |
| **EM Cloud Control** | Full administration of CDBs and PDBs; cannot install fresh software but can duplicate |
| **EM Express** | Local database-level tool; performance overview, PDB management |
| **SQL Developer** | Create and manage PDBs; anything you can do in SQL*Plus, you can do here |
| **AutoUpgrade** | Upgrade CDBs and PDBs between releases |

---

## 6. Data Dictionary Views in a CDB

This is where behaviour changes depending on where you are connected. The same view name returns different data depending on your connection level.

### The Four View Families

| View Prefix | Shows | Where Available |
|-------------|-------|----------------|
| `USER_` | Objects owned by the current user | Anywhere |
| `ALL_` | Objects the current user has access to | Anywhere |
| `DBA_` | All objects within the **current container** | Anywhere |
| `CDB_` | All objects across **all containers** | CDB root: all containers. PDB: same as DBA_ (plus CON_ID column) |

**Key rule:** `DBA_` views only show what is in the container you are connected to. If you connect to PDB1, `DBA_TABLESPACES` shows PDB1's tablespaces only. Connect to `CDB$ROOT` and query `CDB_TABLESPACES` to see everything.

**The CDB_ view in a PDB:** Oracle could have thrown an error when you query `CDB_TABLESPACES` from inside a PDB (since PDBs cannot see out). Instead, it returns the same data as `DBA_TABLESPACES` plus a `CON_ID` column. Convenient, but don't mistake it for a cross-container view — it isn't when you are in a PDB.

### V$ Performance Views

- Connected to `CDB$ROOT` with appropriate privilege: `V$SESSION` shows sessions across all PDBs
- Connected to a PDB: `V$SESSION` shows only that PDB's sessions

Oracle adjusts the scope behind the scenes based on your connection.

---

## 7. Common vs Local

This distinction applies to users, roles, privileges, and objects. Know it cold.

| | Common | Local |
|--|--------|-------|
| **Where created** | `CDB$ROOT` | Inside a specific PDB |
| **Where visible** | Across all PDBs in the CDB | Only within the PDB it was created in |
| **Naming convention** | Must start with `C##` (by default) | Any valid name |
| **Example** | `C##ADMIN`, `SYS`, `SYSTEM` | `HR`, `APP_USER`, `PDBADMIN` |
| **Can exist in CDB root?** | Yes | No — local objects cannot exist in root |

> `SYS` and `SYSTEM` are common users. The `HR` schema you created in the lab is a local user — it only exists in `ORCLPDB1`.

---

## 8. CDB DBA vs PDB DBA

The traditional DBA role splits into two levels in a multitenant environment:

- **CDB DBA** — manages the whole container: all PDBs, common users, common auditing, resource plans, TDE at the CDB level, backups, upgrades, patching
- **PDB DBA** — manages a single PDB: local users, local roles, local tablespaces, local parameters, local auditing policies, application objects

In practice, one person may wear both hats in a small environment. In a cloud or multi-tenant hosting scenario, a CDB DBA might provision PDBs for teams who then manage their own PDB DBA responsibilities independently.

---

## 9. Feature Scope: CDB Level vs PDB Level

| Feature | CDB Level | PDB Level |
|---------|-----------|-----------|
| **Parameter file** | Single SPFILE for the CDB | PDB-specific parameters stored in PDB data dictionary |
| **Character set** | CDB defines the master character set | PDBs can use the CDB character set or a subset of it |
| **Resource Manager** | CDB plan divides resources between PDBs | PDB plans manage resources within each PDB |
| **Unified Auditing** | Enabled at CDB level (binary) | Policies configured at PDB level |
| **Transparent Data Encryption (TDE)** | Master key enabled at CDB level | Each PDB has its own master key; shared or dedicated wallet |
| **Database Vault** | Enabled at CDB level | Policies built and enforced at PDB level |
| **Data Guard / Standby** | Operates at CDB level | Standby can include a subset of primary PDBs |
| **Heat Maps / ADO** | Enabled at CDB level | Policies defined per-object within each PDB |
| **LogMiner** | Accesses all redo for the CDB | Used at CDB level across all containers |
| **XStream / GoldenGate** | Replication configurable at CDB or PDB scope | |
| **Backup (RMAN)** | Centralised backup of whole CDB | Can also back up individual PDBs |

---

## 10. Key Queries for CDB Exploration

These are the first queries you run when connecting to any CDB:

```sql
-- What database am I connected to? Is it a CDB?
SELECT name, cdb, con_id FROM v$database;
-- CON_ID = 0 here (belongs to the database, not a container)

-- What container am I currently in?
SHOW CON_ID
SHOW CON_NAME

-- What PDBs exist in this CDB?
SHOW PDBS

-- All tablespaces across all containers (from CDB root)
SELECT tablespace_name, con_id FROM cdb_tablespaces ORDER BY con_id;

-- Tablespaces in the current container only
SELECT tablespace_name FROM dba_tablespaces;

-- All users, their container, and whether they are common or local
SELECT username, common, con_id FROM cdb_users ORDER BY con_id, username;
```

### What the Lab Demo Showed

Connected to CDB root (`CDB$ROOT`, CON_ID = 1) on a database called `ORCL`:

- `v$database` → `CON_ID = 0`, `CDB = YES`
- `SHOW CON_ID` → `1` (connected to root)
- `SHOW CON_NAME` → `CDB$ROOT`
- `SHOW PDBS` → `PDB$SEED` (READ ONLY) and `PDB1` (READ WRITE)
- `CDB_TABLESPACES` → tablespaces for CON_ID 1 (root) and CON_ID 4 (PDB1)
- `DBA_TABLESPACES` → only root tablespaces (current container only)
- `CDB_USERS` → root users all show `COMMON = YES`; `HR` in CON_ID 4 shows `COMMON = NO` (local user)

---

## 11. Wrap-Up (One Database to Contain Them All)

The multitenant architecture is not a feature you configure on top of Oracle Database — in 19c, it *is* Oracle Database. The key mental model to carry forward:

- **CDB root** = Oracle's management layer. You run commands here, you don't store data here.
- **PDB$SEED** = read-only template. Oracle owns it; you use it.
- **PDBs** = where applications and users live. Isolated, portable, independently manageable.
- **Common** = defined in root, visible everywhere. **Local** = defined in a PDB, stays in that PDB.
- **CDB_ views** from root = cross-container visibility. **DBA_ views** = current container only.

Next up: creating and provisioning CDBs and PDBs.
