# Lab 10-3 - Running AutoUpgrade Analyze and Fixups

This lab performs the `AutoUpgrade` prechecks and fixups for the `ORCL` database. The goal is to prove the database is ready, create the upgrade configuration file, and let Oracle clean up the obvious problems before the actual deploy begins. Basically, this is the chapter where the jar file gets to judge your environment.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Validate the source database state
- Check that Database Vault is not blocking the upgrade
- Confirm the `PDB`s are open `READ WRITE`
- Create an `AutoUpgrade` working directory
- Generate a sample config file
- Create the real `ORCL` upgrade config file
- Run `AutoUpgrade` in `analyze` mode
- Enable `ARCHIVELOG` if needed
- Run `AutoUpgrade` in `fixups` mode

---

## 2. Assumptions

This lab assumes:

- The new `19c` home is installed
- The source database is `ORCL`
- The source database is currently still on `12.2`
- The `oracle` user owns both the source and target database homes

The transcript preserves the config filename clearly:

```text
config_orcl_122_2_19.cfg
```

It does not preserve every command perfectly, so this lab keeps the exact values that are clear and uses standard Oracle syntax for the rest.

---

## 3. Switch to `oracle` and Set the Source Environment

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the source environment:

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=ORCL
```

Or use:

```bash
. oraenv
```

with:

```text
ORCL
```

Restart the database cleanly if the lab guide wants you to normalize its state:

```bash
srvctl stop database -db orcl
srvctl start database -db orcl
```

---

## 4. Check the Database Before Running AutoUpgrade

Connect to `SQL*Plus`:

```bash
sqlplus / as sysdba
```

Check whether Database Vault is present:

```sql
column comp_name format a40
select comp_name, status
from dba_registry
where comp_name like '%Vault%';
```

If Database Vault is installed and enabled, handle it before continuing.

Check the pluggable databases:

```sql
show pdbs;
```

Make sure the `PDB`s that you want upgraded are open `READ WRITE`.

Exit:

```sql
exit
```

---

## 5. Create the AutoUpgrade Working Area

Create a working directory for config and log files:

```bash
mkdir -p /home/oracle/autoupgrade/orcl
cd /home/oracle/autoupgrade/orcl
```

This keeps the config files and the log sprawl in one predictable place instead of scattering it across the filesystem like Oracle confetti.

---

## 6. Generate a Sample Config File

Switch the shell to the target `19c` home for running AutoUpgrade:

```bash
export ORACLE_HOME=/u01/app/oracle/product/19c/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Create a sample config file:

```bash
$ORACLE_HOME/jdk/bin/java -jar $ORACLE_HOME/rdbms/admin/autoupgrade.jar -create_sample_file config_sample.cfg
```

Verify it exists:

```bash
ls
cat config_sample.cfg
```

The transcript shows the sample file being used as the starter template rather than building the real config from scratch, which is exactly the sane thing to do.

---

## 7. Create the Real ORCL Config File

Create the upgrade config file:

```bash
gedit config_orcl_122_2_19.cfg
```

Use content in this pattern:

```text
global.autoupg_log_dir=/home/oracle/autoupgrade
upg1.start_time=NOW
upg1.dbname=ORCL
upg1.source_home=/u01/app/oracle/product/12.2.0/dbhome_1
upg1.target_home=/u01/app/oracle/product/19c/dbhome_1
upg1.sid=ORCL
upg1.log_dir=/home/oracle/autoupgrade/orcl
upg1.upgrade_node=localhost
upg1.target_version=19
upg1.run_utlrp=yes
upg1.timezone_upg=yes
```

Save the file and verify it:

```bash
cat config_orcl_122_2_19.cfg
```

If your classroom image uses a slightly different target-home path, use that actual path instead.

---

## 8. Run Analyze Mode

Run AutoUpgrade in analyze mode:

```bash
$ORACLE_HOME/jdk/bin/java -jar $ORACLE_HOME/rdbms/admin/autoupgrade.jar -config config_orcl_122_2_19.cfg -mode analyze
```

The demo notes that a warning about AutoUpgrade not being fully tested can appear in this environment. Treat that as a known classroom artifact unless the actual job output reports a real failure.

When analyze completes, note the job number. In the transcript, the first job became:

```text
100
```

Review the generated files:

```bash
cd /home/oracle/autoupgrade/orcl/100/prechecks
ls
```

Open the preupgrade log:

```bash
cat orcl_preupgrade.log
```

Review the sections that report required actions.

---

## 9. Enable ARCHIVELOG If Needed

The transcript shows that `ORCL` was not yet in `ARCHIVELOG` mode, which matters because the later deploy step wants flashback-based fallback.

Return to `SQL*Plus` as `SYSDBA`:

```bash
sqlplus / as sysdba
```

Enable archiving:

```sql
shutdown immediate;
startup mount;
alter database archivelog;
alter database open;
alter pluggable database all open;
exit
```

Make sure the `FRA` is configured and sized appropriately before the actual deploy step.

---

## 10. Run Fixups Mode

Return to the config directory:

```bash
cd /home/oracle/autoupgrade/orcl
```

Run fixups:

```bash
$ORACLE_HOME/jdk/bin/java -jar $ORACLE_HOME/rdbms/admin/autoupgrade.jar -config config_orcl_122_2_19.cfg -mode fixups
```

By default, AutoUpgrade opens its interactive console. While the job is running, you can check active jobs with:

```text
lsj -r
```

When fixups complete, note the new job number. In the transcript, the fixups run became:

```text
101
```

Review the generated fixup logs, for example:

```bash
cd /home/oracle/autoupgrade/orcl/101/prechecks
ls
cat preupgrade_fixups_cdbroot.log
```

The exact filename can vary a bit by environment, but the point is the same: inspect what AutoUpgrade actually changed instead of assuming it solved your problems through moral clarity.

---

## 11. What You Just Finished

By the end of this lab you:

- validated the source database and `PDB` open state
- built an `AutoUpgrade` working directory
- created a real upgrade config file for `ORCL`
- ran `analyze`
- enabled `ARCHIVELOG`
- ran `fixups`

Which means the database has now been judged, corrected where possible, and prepared for the actual deploy instead of being thrown into the upgrade cold.
