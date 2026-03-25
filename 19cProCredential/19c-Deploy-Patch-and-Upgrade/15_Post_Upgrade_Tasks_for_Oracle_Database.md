## Lesson 15 - Post-Upgrade Tasks for Oracle Database (where "upgrade complete" turns out to be a dangerously optimistic phrase)

Finishing the upgrade is not the same thing as finishing the job. After the database is upgraded, Oracle expects you to validate the result, clean up what the upgrade left behind, recompile what broke, gather the right statistics, review the new home's configuration, and decide whether you are finally brave enough to drop the guaranteed restore point. This lesson focuses on the post-upgrade work that matters, especially the tasks that **AutoUpgrade** may automate for you and the tasks that **manual** upgrades leave entirely in your lap like a bag of live raccoons.

By the end of this lesson, you should be able to:

- Distinguish post-upgrade tasks that AutoUpgrade handles from those you still must verify
- Identify the mandatory cleanup tasks after a manual upgrade
- Use `postupgrade_fixups.sql`, `utlrp.sql`, and `utlusts.sql` appropriately
- Validate Oracle Restart, `oratab`, password-file, and `SPFILE` configuration after upgrade
- Recognize important recommended post-upgrade actions such as statistics gathering and Unified Auditing planning

---

## 1. AutoUpgrade vs Manual Post-Upgrade Work

What you must do after an upgrade depends heavily on:

- which upgrade method you used

### If you used AutoUpgrade

Then Oracle can automatically handle many post-upgrade tasks, including:

- running several post-upgrade checks and fixups
- updating `oratab`
- copying or merging network files, depending on configuration
- restarting the upgraded database in the new home

But it does **not** necessarily do everything you may want.

For example:

- the guaranteed restore point is **not** dropped by default

Oracle documents the parameter:

- `drop_grp_after_upgrade`

and the default is:

- `no`

That is sensible. Oracle is basically saying, "Please validate the database before you throw away the parachute."

### If you upgraded manually

Then the post-upgrade work is much more manual, including:

- running `postupgrade_fixups.sql`
- recompiling invalid objects
- checking environment variables
- updating `oratab`
- moving or recreating home-local files
- validating listeners, password files, and other network assets

So yes, the manual path gives you control. It also gives you a to-do list with opinions.

---

## 2. `postupgrade_fixups.sql` (manual-upgrade cleanup that Oracle already warned you about)

When you run the **Pre-Upgrade Information Tool**:

- `preupgrade.jar`

it generates scripts including:

- `preupgrade_fixups.sql`
- `postupgrade_fixups.sql`

After a **manual** upgrade completes and the database is opened normally, Oracle expects you to run:

- `postupgrade_fixups.sql`

This script addresses warnings, informational items, and cleanup actions identified by the earlier preupgrade analysis.

If you used:

- `DBUA`
- or `AutoUpgrade`

then Oracle already automates much of this work for you.

So the short version is:

- manual upgrade: you run it
- DBUA: Oracle largely runs it
- AutoUpgrade: Oracle largely absorbs it into the workflow

---

## 3. Recompile Invalid Objects

After upgrade, recompilation matters because many stored objects become invalid when:

- Oracle-owned packages change
- internal metadata changes
- compiled code references old definitions

The standard tool is:

- `utlrp.sql`

Oracle explicitly recommends running it after:

- install
- patch
- upgrade

Manual upgrades require this step directly.

With AutoUpgrade:

- Oracle recompiles Oracle-maintained invalid objects automatically

But it is still wise to **verify** the outcome instead of assuming all invalids vanished through moral effort.

Useful checks include:

```sql
select count(*)
from dba_objects
where status = 'INVALID';
```

and, if needed:

```sql
select owner, object_name, object_type
from dba_objects
where status = 'INVALID'
order by owner, object_type, object_name;
```

Because "no invalid objects" is better discovered with a query than with interpretive hope.

---

## 4. Check the Upgrade Result Properly

Oracle provides a dedicated post-upgrade status tool:

- `utlusts.sql`

This tool summarizes:

- component versions
- elapsed upgrade time
- status information from `DBA_REGISTRY`

It is one of the cleanest ways to answer the question:

- "Did the components actually finish upgrading?"

Manual path example:

```sql
@?/rdbms/admin/utlusts.sql TEXT
```

You should also review:

- the upgrade spool log
- the alert log
- the AutoUpgrade or DBUA logs

The transcript's practical point is correct: a lot of the real detail is sitting in the logs, not in the single "success" message Oracle shows when it is done trying to keep you calm.

---

## 5. Environment and Home Cleanup After Manual Upgrades

If the upgrade was manual, then you must verify that the operating system and database tooling now point to the **new** home.

That includes checking:

- `ORACLE_HOME`
- `PATH`
- `ORACLE_BASE`
- shell profiles for the Oracle software owner
- `/etc/oratab`

For AutoUpgrade:

- `oratab` is updated automatically by default

For manual upgrades:

- you must update it yourself

If the shell is still pointing to the old home after the upgrade, then congratulations, your database may be upgraded while your admin session is living in the past.

---

## 6. Password File and SPFILE Validation

After upgrade, validate where the database is actually getting its critical files from.

Important items:

- password file location
- `SPFILE` location
- Oracle Restart / `srvctl` configuration

In `ASM` environments, it is often preferable for the password file to live in:

- an `ASM` disk group

not only in:

- `$ORACLE_HOME/dbs`

Oracle documents that password files in disk groups can be managed with:

- `orapwd`
- `srvctl`
- `ASMCMD`

and `srvctl config database` should reflect the correct location.

Likewise, verify the `SPFILE` location with:

- `srvctl config database -db <db_unique_name> -all`
- `show parameter spfile`

The goal is to make sure Oracle Restart and the database agree about reality. When they do not, the next restart becomes a debugging exercise with religious overtones.

---

## 7. If You Use an RMAN Recovery Catalog

If your environment uses an `RMAN` recovery catalog, then after the database upgrade you must ensure:

- the catalog database is on a compatible release
- the recovery catalog schema itself is upgraded if needed

Oracle's command for that is:

- `UPGRADE CATALOG`

And yes, Oracle makes you type it twice unless you use `NOPROMPT`, because apparently one confirmation is not enough when metadata is being rewritten.

So if backup and recovery matter to you, and one assumes they do, do not leave the catalog lagging behind the upgraded environment and then act surprised later.

---

## 8. Statistics After Upgrade

Oracle recommends gathering statistics after upgrade, especially:

- dictionary statistics
- fixed object statistics

Dictionary stats matter because:

- the data dictionary changed during the upgrade

Fixed object stats matter because:

- the optimizer uses them for internal performance decisions
- bad or stale fixed-object stats can create ugly execution plans

Typical examples:

```sql
exec dbms_stats.gather_dictionary_stats;
exec dbms_stats.gather_fixed_objects_stats;
```

For fixed object stats, Oracle recommends doing this after representative workload has run, so the stats reflect something real instead of a system that has only just awakened from surgery.

---

## 9. Other Recommended Post-Upgrade Checks

Depending on your environment, Oracle recommends or supports additional checks after upgrade.

These can include:

- upgrading user tables dependent on Oracle-maintained types with `utluptabdata.sql` if needed
- upgrading statistics tables created with `DBMS_STATS.UPGRADE_STAT_TABLE`
- validating or upgrading `APEX` if it is in use
- identifying Oracle Text indexes that should be rebuilt
- validating network ACL, listener, and client compatibility settings if older clients are involved
- reviewing XML DB / HTTP / application-integration ports if they matter in your environment

Not every database needs every one of these tasks. But pretending your environment is generic when it has Text, APEX, ACLs, and older clients attached to it is how post-upgrade issues get promoted into production folklore.

---

## 10. Security and Feature Follow-Up

The transcript correctly calls out two security-related items that deserve attention after upgrade.

### Re-enable Database Vault / Oracle Label Security if you disabled them

If these were disabled for the upgrade:

- turn them back on
- validate that they are functioning as expected

Because "we meant to re-enable it later" is not a control framework.

### Plan the move to Unified Auditing

Oracle recommends planning migration to:

- **Unified Auditing**

After upgrade, you should review whether:

- legacy auditing is still in use
- Unified Auditing should be enabled or expanded

Oracle calls this out because Unified Auditing is the long-term direction, and the old auditing model is not where Oracle wants you camping forever.

---

## 11. The Guaranteed Restore Point Problem

If AutoUpgrade created a guaranteed restore point:

- keep it until the upgraded database is validated

Then:

- drop it deliberately

Why:

- restore points consume `FRA` space
- leaving them around too long can cause avoidable pressure in recovery storage

So yes, keeping it forever is also not a strategy. The point is:

1. keep it long enough to trust the upgrade
2. remove it before it starts eating recovery space like a raccoon in your attic

---

## 12. Practical Post-Upgrade Checklist

After upgrading Oracle Database, validate at least the following:

- `DBA_REGISTRY` shows the expected component versions and `VALID` status
- invalid object count is acceptable or zero after recompilation
- `oratab` points to the new home
- `ORACLE_HOME` and shell profiles point to the new home
- Oracle Restart / `srvctl` shows the correct `SPFILE` and password-file locations
- the guaranteed restore point has been retained only until validation is complete
- dictionary and fixed object statistics have been addressed
- RMAN recovery catalog is upgraded if your environment uses one
- security options like Database Vault or OLS are re-enabled if you disabled them
- any APEX, Text, ACL, XML DB, or client-compatibility issues are reviewed where relevant

This is the part where you decide whether the upgrade is actually finished, or whether it merely stopped emitting errors for a while.

---

## 13. Wrap-Up (the upgrade is over, the responsibility is not)

This lesson covered the practical post-upgrade tasks for Oracle Database: what AutoUpgrade may handle automatically, what manual upgrades still require, how to use `postupgrade_fixups.sql`, `utlrp.sql`, and `utlusts.sql`, how to validate environment settings, `oratab`, `SPFILE`, and password-file configuration, why statistics and recovery catalog updates matter, and why the guaranteed restore point should be dropped only after validation. That completes the upgrade-and-post-upgrade part of the course, which means the remaining work is mostly cleanup, confirmation, and resisting the urge to declare victory too early.
