# Practice 2-2 - Creating a New CDB (In Which We Prove We Can Build One, Not Just Talk About It)

This practice creates a second CDB called `cdb19` using **DBCA in silent mode** — the command-line, scriptable version of DBCA that runs without a GUI and is suitable for automation. The official lab uses pre-built scripts. This document shows both the script approach and how to adapt it for your personal lab to create a second CDB alongside `ORCLCDB`.

---

## 1. Check Existing Databases (oratab)

Before creating anything, see what already exists:

```bash
cat /etc/oratab
```

The official lab environment shows:
```
noncdb:/opt/oracle/product/12.2/dbhome_1:N
cdb12:/opt/oracle/product/12.2/dbhome_1:N
cdb18:/opt/oracle/product/18.1/dbhome_1:N
ORCL:/opt/oracle/product/19c/dbhome_1:Y
```

Your personal lab after RPM install:
```
ORCLCDB:/opt/oracle/product/19c/dbhome_1:Y
```

After creating a second CDB, it will be added here automatically by DBCA.

---

## 2. The DBCA Silent Mode Command

The official lab script uses DBCA in silent mode to create `cdb19`. The key parameters:

```bash
dbca -silent \
  -createDatabase \
  -gdbName cdb19 \
  -sid cdb19 \
  -createAsContainerDatabase true \
  -numberOfPDBs 0 \
  -templateName General_Purpose.dbc \
  -emExpressPort 5502 \
  -recoveryAreaSize 3000 \
  -recoveryAreaDestination /opt/oracle/fast_recovery_area \
  -datafileDestination /opt/oracle/oradata \
  -enableArchive false \
  -sysPassword YourPassword \
  -systemPassword YourPassword
```

Key parameters explained:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `-createAsContainerDatabase true` | `true` | Makes this a CDB, not a non-CDB |
| `-numberOfPDBs 0` | `0` | Create an empty CDB — no PDBs at creation time |
| `-emExpressPort` | `5502` | Assigns a different EM Express port (5500 is taken by ORCLCDB) |
| `-datafileDestination` | path | Where Oracle puts data files (OMF-style) |
| `-recoveryAreaDestination` | path | Fast Recovery Area location |

> DBCA silent mode writes progress to the terminal and a log file. It takes 20–30 minutes. There is no GUI — if it hangs with no output, check the DBCA log at `$ORACLE_BASE/cfgtoollogs/dbca/`.

---

## 3. The Redo Log Adjustment Script

After DBCA creates the CDB, the lab runs a second script to replace the default redo log groups with properly sized ones. This mirrors what you would do in any fresh CDB:

```sql
-- Connect to the new CDB
sqlplus / as sysdba

-- Clear the LOCAL_LISTENER parameter (use default listener, not a hardcoded one)
ALTER SYSTEM SET local_listener='' SCOPE=BOTH;

-- Create three properly-sized redo log groups (multiplexed)
ALTER DATABASE ADD LOGFILE GROUP 4
  ('/opt/oracle/oradata/cdb19/redo04a.log',
   '/opt/oracle/oradata/cdb19/redo04b.log') SIZE 200M;

ALTER DATABASE ADD LOGFILE GROUP 5
  ('/opt/oracle/oradata/cdb19/redo05a.log',
   '/opt/oracle/oradata/cdb19/redo05b.log') SIZE 200M;

ALTER DATABASE ADD LOGFILE GROUP 6
  ('/opt/oracle/oradata/cdb19/redo06a.log',
   '/opt/oracle/oradata/cdb19/redo06b.log') SIZE 200M;

-- Switch to one of the new groups so the old ones become INACTIVE
ALTER SYSTEM SWITCH LOGFILE;
ALTER SYSTEM SWITCH LOGFILE;
ALTER SYSTEM SWITCH LOGFILE;

-- Checkpoint to ensure all redo is written
ALTER SYSTEM CHECKPOINT;

-- Drop the original DBCA-generated groups (check they are INACTIVE first)
SELECT group#, status FROM v$log ORDER BY group#;
-- Drop only INACTIVE groups
ALTER DATABASE DROP LOGFILE GROUP 1;
ALTER DATABASE DROP LOGFILE GROUP 2;
ALTER DATABASE DROP LOGFILE GROUP 3;
```

---

## 4. Personal Lab — Creating a Second CDB

Your `ORCLCDB` uses port `5500` for EM Express. A second CDB needs a different port. Here is the full sequence for your OL8 lab:

### Step 1 — Run DBCA

```bash
# Switch to oracle user if not already
sudo su - oracle
source ~/.bash_profile   # ensure ORACLE_HOME is set

dbca -silent \
  -createDatabase \
  -gdbName CDB2 \
  -sid CDB2 \
  -createAsContainerDatabase true \
  -numberOfPDBs 0 \
  -templateName General_Purpose.dbc \
  -emExpressPort 5502 \
  -recoveryAreaSize 3000 \
  -recoveryAreaDestination /opt/oracle/fast_recovery_area \
  -datafileDestination /opt/oracle/oradata \
  -enableArchive false \
  -sysPassword YourPassword \
  -systemPassword YourPassword \
  -useOMF true
```

Watch for the completion message:
```
100% complete
Look at the log file "/opt/oracle/cfgtoollogs/dbca/CDB2/..." for further details.
```

### Step 2 — Set Environment and Connect

```bash
export ORACLE_SID=CDB2
sqlplus / as sysdba
```

### Step 3 — Verify

```sql
-- Confirm it's a CDB
SELECT name, cdb, open_mode FROM v$database;

-- Should show only PDB$SEED (empty CDB, no user PDBs)
SELECT con_id, name, open_mode FROM v$pdbs;

-- Data files — only CDB root and PDB$SEED
SELECT con_id, tablespace_name FROM cdb_data_files ORDER BY con_id;

-- Tablespaces in CDB root
SELECT tablespace_name FROM dba_tablespaces ORDER BY tablespace_name;
```

### Step 4 — Configure EM Express

```sql
-- Verify EM Express port (should be 5502 as set in DBCA)
SELECT dbms_xdb_config.gethttpsport() FROM dual;

-- Enable global port so all future PDBs can also use EM Express
EXEC dbms_xdb_config.setglobaltxnport(5502);

-- Verify LOCAL_LISTENER is clear (not hardcoded to a specific listener)
SHOW PARAMETER local_listener;
-- VALUE should be empty
-- If not: ALTER SYSTEM SET local_listener='' SCOPE=BOTH;
```

### Step 5 — Access the New CDB in EM Express

```
https://localhost:5502/em
```

Log in as `SYS` with your password, container `CDB2`.

---

## 5. Verify oratab After Creation

```bash
cat /etc/oratab
```

Expected — both CDBs should now appear:
```
ORCLCDB:/opt/oracle/product/19c/dbhome_1:Y
CDB2:/opt/oracle/product/19c/dbhome_1:Y
```

DBCA registers the new database automatically. The `:Y` means `oraenv` will auto-start this database.

---

## 6. Switching Between CDBs

With two CDBs on the same host, use `oraenv` to switch context:

```bash
# Switch to ORCLCDB
. oraenv    # enter: ORCLCDB
sqlplus / as sysdba

# Switch to CDB2
. oraenv    # enter: CDB2
sqlplus / as sysdba
```

Or set `ORACLE_SID` directly:

```bash
export ORACLE_SID=CDB2
sqlplus / as sysdba
```

The listener serves both CDBs on port 1521. Check which services are registered:

```bash
lsnrctl status | grep -i service
```

---

## 7. Starting the Second CDB on Boot

The RPM-based `ORCLCDB` has a systemd service. A DBCA-created CDB does not — you need to start it manually or add it to `/etc/oratab` with `:Y` and configure `dbstart`/`dbshut`, or create your own systemd unit.

For a lab, manual startup is fine:

```bash
export ORACLE_SID=CDB2
sqlplus / as sysdba
STARTUP
```

---

## 8. Wrap-Up (You Now Have Two CDBs)

Creating a second CDB demonstrates that the CDB is not a special singleton — it is a database instance like any other, just with `ENABLE PLUGGABLE DATABASE` turned on. With two CDBs on one host you can now practice:

- **Cross-CDB PDB migration** (plug a PDB out of CDB2, plug into ORCLCDB)
- **RMAN duplication** from one CDB to another
- **Listener service routing** to different CDBs on the same port
- **EM Express on different ports** for each CDB

Next up: creating PDBs inside a CDB.
