# Lab 7-3 - Managing Oracle Restart Resource Configuration

This lab inspects, removes, re-adds, disables, and re-enables the `ORCL` database resource in Oracle Restart. It is essentially a guided lesson in what happens when Oracle Restart knows about your database, forgets about your database, and then is forced to care again.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Review the Oracle Restart configuration for `ORCL`
- Record the `SPFILE` location
- Create a local `initORCL.ora` stub that points to the `ASM`-based `SPFILE`
- Stop and remove `ORCL` from Oracle Restart
- Start the database manually in `SQL*Plus`
- Prove that Oracle Restart no longer restarts it automatically
- Add `ORCL` back to Oracle Restart
- Disable and re-enable the database resource
- Verify the effect on automatic restart behavior

---

## 2. Assumptions

This lab assumes:

- The `ORCL` database exists and is currently registered with Oracle Restart
- The database uses an `SPFILE` stored in `ASM`
- The `oracle` and `grid` users already exist

The course demo includes a long `srvctl add database` command whose full line does not survive transcription cleanly. This lab preserves the important values that were clearly called out:

- the recorded `SPFILE` path
- the Oracle home
- the database unique name
- the database name
- the domain

If your classroom guide includes extra options such as disk-group dependencies, use them too.

---

## 3. Oracle User Terminal: Inspect the Current Configuration

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the database environment:

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
export ORACLE_SID=ORCL
```

List the configured databases:

```bash
srvctl config database
```

Display the detailed configuration for `ORCL`:

```bash
srvctl config database -db orcl -all
```

Record the `SPFILE` location shown in the output. You will need that exact value later.

---

## 4. Create a Local `initORCL.ora` Stub

Change to the database parameter file directory:

```bash
cd /u01/app/oracle/product/12.2.0/dbhome_1/dbs
```

Create a text initialization file that points to the recorded `SPFILE` in `ASM`:

```bash
vi initORCL.ora
```

Add a single line in this format:

```text
SPFILE='<recorded_spfile_path>'
```

Save the file and verify it:

```bash
cat initORCL.ora
```

The demo catches a typo here and fixes it before continuing, which is exactly why checking the file is a better habit than trusting your keyboard on faith.

---

## 5. Stop and Remove `ORCL` from Oracle Restart

Stop the database:

```bash
srvctl stop database -db orcl
```

Check until it is no longer running:

```bash
srvctl status database -db orcl
```

Now remove it from Oracle Restart:

```bash
srvctl remove database -db orcl
```

Confirm the prompt when asked.

Verify that it is no longer configured:

```bash
srvctl config database -db orcl
```

At this point Oracle Restart no longer manages `ORCL`. It has been officially uninvited.

---

## 6. Start the Database Manually and Prove It Is No Longer Recovered

Start `SQL*Plus`:

```bash
sqlplus / as sysdba
```

Start the database manually:

```sql
startup;
exit
```

Find the database `LGWR` process:

```bash
ps -ef | grep -i lgwr | grep -i orcl
```

Kill it:

```bash
kill -9 <lgwr_pid>
```

Wait and check for the process again:

```bash
ps -ef | grep -i lgwr | grep -i orcl
```

Unlike the earlier recovery lab, it should **not** come back automatically, because Oracle Restart is no longer managing the database resource.

---

## 7. Add `ORCL` Back to Oracle Restart

Re-add the database to Oracle Restart using the recorded `SPFILE` path:

```bash
srvctl add database -db orcl -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -dbname orcl -domain oracle.com -spfile '<recorded_spfile_path>'
```

If your classroom image requires additional options such as `-diskgroup`, include them here.

Check the configuration again:

```bash
srvctl config database -db orcl -all
```

Because the database was started manually, Oracle Restart may still not regard it as properly managed yet. Start it with `srvctl`:

```bash
srvctl start database -db orcl
srvctl status database -db orcl
```

Now test recovery again:

```bash
ps -ef | grep -i lgwr | grep -i orcl
kill -9 <lgwr_pid>
srvctl status database -db orcl
```

Wait and rerun the status or process check until the database comes back.

---

## 8. Disable the Database Resource

Disable the database resource:

```bash
srvctl disable database -db orcl
```

Check its status:

```bash
srvctl status database -db orcl
```

The database may still be running, but the resource is now disabled. That means Oracle Restart will not automatically restart it and `srvctl start` will refuse to manage it while the resource remains disabled.

Crash the database again:

```bash
ps -ef | grep -i lgwr | grep -i orcl
kill -9 <lgwr_pid>
```

Wait and verify that it does **not** restart automatically.

Try to start it with `srvctl`:

```bash
srvctl start database -db orcl
```

This should fail while the resource is disabled.

---

## 9. Re-Enable the Resource and Start It Cleanly

Re-enable the resource:

```bash
srvctl enable database -db orcl
```

Start the database:

```bash
srvctl start database -db orcl
srvctl status database -db orcl
```

If you want one last confidence check, verify that the listener is still online:

```bash
srvctl status listener -listener LISTENER
```

---

## 10. Finish the Lab

When `ORCL` is running again and managed by Oracle Restart:

- close any editors you opened
- exit the terminal windows

---

## 11. What You Just Finished

By the end of this lab you:

- inspected the current Oracle Restart configuration for `ORCL`
- created a local `initORCL.ora` stub for the `ASM` `SPFILE`
- removed `ORCL` from Oracle Restart
- proved that an unmanaged database is not automatically restarted
- added `ORCL` back to Oracle Restart
- disabled and re-enabled the database resource

Which is the cleanest possible demonstration that Oracle Restart is not magic. It only protects the resources it knows about and is currently allowed to manage.
