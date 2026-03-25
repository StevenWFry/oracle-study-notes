# Lab 11-1 - Performing Post-Upgrade Validation and Cleanup

This lab performs the classroom post-upgrade checks after the `ORCL` upgrade to `19c`. Some of the work is basic validation. Some of it is cleanup. And some of it is the Oracle equivalent of checking whether the parachute opened before you pack it away and pretend the landing was always going to be graceful.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Verify that `oratab` points to the new `19c` home
- Validate the Oracle Restart configuration for `ORCL`
- Check component versions and invalid objects
- Optionally move the database password file into `ASM`
- Verify the `SPFILE` location
- Recreate a local `init.ora` stub if needed
- Drop the AutoUpgrade guaranteed restore point
- Review the alert log for upgrade evidence

---

## 2. Assumptions

This lab assumes:

- The `ORCL` upgrade to `19c` completed successfully
- Oracle Restart is managing the database
- The database uses `ASM`
- The AutoUpgrade guaranteed restore point still exists

The transcript uses the classroom passwords and assumes the `ORCL` database password remains:

```text
cloud_4u
```

---

## 3. Switch to `oracle` and Check `oratab`

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Verify your group memberships if you want the same reassurance the demo asked for:

```bash
id
```

Now check `oratab`:

```bash
grep '^ORCL:' /etc/oratab
```

You should see that `ORCL` now points to the `19c` home, showing that AutoUpgrade updated `oratab` as expected.

---

## 4. Set the 19c Environment and Check Oracle Restart Configuration

Set the upgraded environment:

```bash
export ORACLE_HOME=/u01/app/oracle/product/19c/dbhome_1
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

Check the Oracle Restart configuration:

```bash
srvctl config database -db orcl -all
```

Review the output for:

- Oracle home
- `SPFILE`
- password-file location

In the classroom flow, the `SPFILE` already points to `ASM`, but the password file is still only in the default local location. That is functional, but not ideal.

---

## 5. Check Component Versions and Invalid Objects

Connect to the database:

```bash
sqlplus / as sysdba
```

Check the component registry:

```sql
column comp_name format a40
select comp_name, version, status
from dba_registry
order by comp_name;
```

Check for invalid objects:

```sql
select count(*)
from dba_objects
where status = 'INVALID';
```

In the classroom transcript, the result is zero invalid objects, which is exactly the sort of number you want to see after Oracle has spent hours rewriting metadata.

Exit `SQL*Plus`:

```sql
exit
```

---

## 6. Validate the Current Password File Location

Check for a local password file in the upgraded home:

```bash
ls $ORACLE_HOME/dbs/orapw*
```

Test a password-file-based login if desired:

```bash
sqlplus sys/cloud_4u@ORCL as sysdba
```

Then deliberately test a wrong password to prove authentication is actually happening through the password file rather than through wishful thinking:

```bash
sqlplus sys/not_the_right_password@ORCL as sysdba
```

Exit if the valid login succeeds.

---

## 7. Optionally Move the Database Password File into ASM

If `srvctl config database -db orcl -all` does not already show a password file in `ASM`, create one there.

Oracle documents that `orapwd` with `DBUNIQUENAME` can create a database password file in an `ASM` disk group and update the database resource metadata.

Example:

```bash
orapwd file='+DATA/ORCL/orapworcl' dbuniquename='orcl' password='cloud_4u' format=12.2 force=y
```

If the password file is already registered and Oracle objects, keep the same command pattern but use the overwrite behavior your classroom guide expects.

Now remove the old local password file:

```bash
rm -f $ORACLE_HOME/dbs/orapwORCL
```

Recheck the Oracle Restart configuration:

```bash
srvctl config database -db orcl -all
```

It should now show the password file in `ASM`.

Test the valid login again:

```bash
sqlplus sys/cloud_4u@ORCL as sysdba
```

If that works after the local file is removed, then the `ASM` password file is doing the actual job instead of just sitting there decoratively.

Exit `SQL*Plus` if needed.

---

## 8. Verify the Password File in ASM as `grid`

Switch to the `grid` user:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

Set the `ASM` environment:

```bash
export ORACLE_HOME=/u01/app/19c/grid
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=+ASM
```

Start `asmcmd`:

```bash
asmcmd
```

List the password file location:

```text
ls +DATA/ORCL
```

You should see the password file in the disk group path.

Exit:

```text
exit
```

Then return to the `oracle` user if needed:

```bash
exit
```

---

## 9. Verify the SPFILE and Recreate a Local init File if Needed

Back as `oracle`, connect to `SQL*Plus`:

```bash
sqlplus / as sysdba
```

Check the active `SPFILE`:

```sql
show parameter spfile
```

You should see it pointing into `ASM`.

Exit:

```sql
exit
```

Now check whether a local `initORCL.ora` stub exists:

```bash
ls $ORACLE_HOME/dbs/initORCL.ora
```

If it does not exist, recreate it from the active `SPFILE`:

```bash
sqlplus / as sysdba
```

```sql
create pfile='$ORACLE_HOME/dbs/initORCL.ora' from spfile;
exit
```

This is not the active `SPFILE`; it is just a useful local stub so the home is not missing all signs of civilization.

---

## 10. Drop the AutoUpgrade Restore Point

Connect to `SQL*Plus` as `SYSDBA`:

```bash
sqlplus / as sysdba
```

List the restore points:

```sql
select name, guarantee_flashback_database
from v$restore_point;
```

Identify the AutoUpgrade restore point and drop it:

```sql
drop restore point <autoupgrade_restore_point_name>;
exit
```

Do this only after you are satisfied with the upgraded database. The transcript is right about that. Keeping the restore point forever is bad for the `FRA`, but dropping it too early is bad for your blood pressure.

---

## 11. Review the Alert Log

Find the diagnostic trace location:

```bash
sqlplus / as sysdba
```

```sql
select value
from v$diag_info
where name = 'Diag Trace';
exit
```

Then inspect the alert log in that directory. Typical example:

```bash
tail -n 200 <diag_trace_dir>/alert_orcl.log
```

Review the upgrade-related messages, component status lines, and startup transitions. The classroom transcript scrolls through the alert log manually, which is exactly the sort of archaeology Oracle administrators call Tuesday.

---

## 12. Optional Old-Home Cleanup

In the classroom flow, the old `12.2` home is **not** removed yet because later exercises still use it.

In a real environment, once validation is complete and no later work depends on the old home, you can plan to remove it. Just do not confuse:

- "the upgrade worked"

with:

- "it is now safe to delete everything old immediately"

Those are different statements, and Oracle is very interested in whether you can tell them apart.

---

## 13. What You Just Finished

By the end of this lab you:

- verified `oratab` and Oracle Restart now point to the `19c` home
- confirmed the database components upgraded successfully
- checked for invalid objects
- optionally moved the password file into `ASM`
- verified the `SPFILE` location
- recreated a local `init.ora` stub if needed
- dropped the AutoUpgrade restore point
- reviewed the alert log for upgrade history

Which means the database is no longer merely upgraded. It is actually validated, cleaned up, and much less likely to surprise you on the next restart.
