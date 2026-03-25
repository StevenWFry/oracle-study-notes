# Lab 10-2 - Backing Up the Database Before Upgrade

This lab creates a full pre-upgrade backup of the `ORCL` database. The point is not to admire `RMAN` output like modern art. The point is to make sure that if the upgrade becomes a story nobody wants to retell, you still have a clean way back.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `oracle` user
- Set the environment for `ORCL`
- Confirm the database state
- Start the database in mount mode
- Run a full `RMAN` backup before the upgrade

---

## 2. Assumptions

This lab assumes:

- The source database is `ORCL`
- Oracle Restart is managing the database
- The source home is still the `12.2` database home
- Enough backup storage is available

The transcript uses a straightforward full backup workflow. It does not preserve every `RMAN` formatting detail perfectly, so this lab keeps the operationally important steps and uses a clean representative command set.

---

## 3. Switch to `oracle` and Set the Environment

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the environment for the source database home:

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=ORCL
```

If you prefer the classroom style:

```bash
. oraenv
```

Then enter:

```text
ORCL
```

---

## 4. Start the Database in Mount Mode

If the database is still open, stop it first:

```bash
srvctl stop database -db orcl
```

Start it in mount mode:

```bash
srvctl start database -db orcl -startoption mount
```

You can verify the mounted state in `SQL*Plus` if desired:

```bash
sqlplus / as sysdba
```

```sql
select open_mode from v$database;
exit
```

The transcript notes that the database was already down from earlier work in the classroom flow. That is fine. The important point is that the backup is taken from a consistent mounted state.

---

## 5. Run the RMAN Backup

Start `RMAN`:

```bash
rman target /
```

Run a full backup. A representative pattern is:

```rman
run {
  allocate channel c1 device type disk;
  backup database tag 'PRE_UPGRADE_ORCL';
  backup current controlfile tag 'PRE_UPGRADE_CTL';
  sql "create pfile=''?/dbs/initORCL_preupgrade.ora'' from spfile";
  release channel c1;
}
```

If your classroom guide specifies a different backup destination, format string, or multiple channels, use those values instead.

The key point is:

- full database backup
- current control file backup
- enough metadata preserved to rebuild the instance if the upgrade becomes theatrical

Exit `RMAN` when the backup completes:

```rman
exit
```

---

## 6. What You Just Finished

By the end of this lab you:

- mounted the `ORCL` database consistently
- created a full pre-upgrade `RMAN` backup
- backed up the current control file
- captured a text initialization file from the `SPFILE`

Which means the upgrade now has an adult supervision plan instead of just hope and a maintenance window.
