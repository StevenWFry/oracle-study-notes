# Practice 1-1 - Discovering the Practice Environment (In Which the Lab Has More Databases Than Your Entire Career Up to This Point)

This practice session covers the Oracle University lab environment setup for the Multitenant Architecture course. The official lab uses **two VMs and four databases across multiple releases**, which is considerably more complex than the single-VM 19c setup in the [lab SOP](../19cAdminWorkshop/sops/lab-vm-oracle-install.md). This document maps the official lab to your personal setup so you know what translates and what to skip.

---

## 1. Official Lab Environment Overview

### VM 1 — Database Host

Four databases are pre-installed:

| Database | Release | Type | Notes |
|----------|---------|------|-------|
| *(name not specified)* | 12.2 | Non-CDB | Used for plug-in migration exercises |
| *(name not specified)* | 12.2 | CDB | Used for cross-version exercises |
| *(name not specified)* | 18c | CDB | Used for upgrade path exercises |
| **ORCL** | **19c** | **CDB** | **Primary lab database** |

Additional databases created during the course:
- **CDB19** — a second 19c CDB created during later exercises
- A temporary **duplicate CDB** — created and then removed during duplication exercises

### VM 2 — Cloud Control Host

| Database | Release | Purpose |
|----------|---------|---------|
| `rcatcdb` | 19c | CDB hosting the Cloud Control repository |
| `emccpdb` | 19c | PDB within rcatcdb — Oracle Enterprise Manager Cloud Control management repository (`sysman`) |

---

## 2. Lab Passwords

| Account | Password | Notes |
|---------|----------|-------|
| All databases (SYS, SYSTEM, etc.) | `Welcome1_` | Capital W, underscore, digit 1 |
| Cloud Control `sysman` | `cloud_4U` | Lowercase `cloud`, uppercase `U` |
| `PDBEM` sysman | `cloud_4U` | Same as Cloud Control sysman |

> **For your personal lab** (Oracle 19c on OL8): you set your own passwords during the `oracledb_ORCLCDB-19c configure` step. Use whatever you set then, or reset with `sqlplus / as sysdba` → `ALTER USER sys IDENTIFIED BY newpassword;`

---

## 3. Storage Layout (Oracle Flex Architecture)

The lab uses **Oracle Flex Disk Groups** with the standard Oracle directory structure. Key locations:

| Location Type | Path Pattern |
|---------------|-------------|
| Data files | Under the DB_CREATE_FILE_DEST (Oracle-managed) |
| Redo log files | Oracle-managed via OMF |
| Recovery files | Fast Recovery Area (FRA) |
| Scripts | Provided per-exercise for setup and cleanup |

> **For your personal lab:** The 19c RPM install uses `/opt/oracle/oradata/` for data files and `/opt/oracle/fast_recovery_area/` for the FRA by default. These are set in `v$parameter` — query `SHOW PARAMETER db_create_file_dest` and `SHOW PARAMETER db_recovery_file_dest` to confirm.

---

## 4. Emergency Recovery Scripts

The official lab provides reset scripts for each database in case of catastrophic lab failure — not for routine cleanup, only when a database becomes unrecoverable and exercises cannot continue.

> **Rule:** Do not run these scripts unless a database cannot start and is throwing unrecoverable errors. They recreate databases from scratch, which means any work done in prior exercises is lost.

This is good practice for your personal lab too — **take a VM snapshot before any exercise that involves destructive operations** (dropping tablespaces, testing incomplete recovery, duplication). VirtualBox snapshots are the personal lab equivalent of the course's reset scripts.

---

## 5. Adapting the Official Labs to Your Personal 19c Setup

The official lab environment uses multiple database versions for exercises that demonstrate cross-version migration and upgrade paths. Your personal lab has a single 19c CDB. Here is how to map the key scenarios:

| Official Lab Scenario | Your Lab Equivalent |
|-----------------------|-------------------|
| Non-CDB (12.2) → plug into ORCL | Create a non-CDB-style schema in a PDB, practice the plug-in process conceptually |
| ORCL (19c) as primary CDB | `ORCLCDB` — your single database |
| CDB19 (second 19c CDB) | Create a second CDB using DBCA — gives you the same cross-CDB migration practice |
| Cloud Control (VM2) | Not available in personal lab — use EM Express at `https://localhost:5500/em` instead |
| Duplicate CDB | Practise RMAN `DUPLICATE` to a different `ORACLE_SID` on the same host |

---

## 6. Connecting to EM Express on Your Lab VM

Since you do not have Cloud Control, EM Express is your GUI management tool. It is already configured by the 19c RPM install:

```bash
# From a browser on the VM (or with port forwarding to the host)
https://localhost:5500/em

# Or if connecting from your Windows host to the VM:
https://<VM_IP>:5500/em
```

Login with `SYS` and your SYS password. Select your container (`ORCLCDB` for CDB-level view, or `ORCLPDB1` for PDB-level view).

To find your VM's IP from the VM terminal:

```bash
ip addr show | grep "inet " | grep -v 127
```

---

## 7. Wrap-Up (Your Lab Is Smaller, But the Concepts Are Identical)

The multi-database, multi-VM official lab is designed for an instructor-led course where each scenario has a pre-built database to work with. Your single-VM setup covers everything you need for 1Z0-082 and the multitenant concepts in this course. Where the labs call for a 12.2 non-CDB, understand the concept and note the difference — the exam tests your knowledge of what the process does, not which specific VM it runs on.

Next up: the actual content of configuring and creating CDBs and PDBs.
