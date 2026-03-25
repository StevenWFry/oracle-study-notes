# Lab 7-2 - Testing Oracle Restart Component Recovery

This lab uses `srvctl` and a few highly educational acts of violence against background processes to prove that Oracle Restart really will restart registered components after a failure. This is a lab exercise only. Do not turn "kill the process and see what happens" into a production management philosophy.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Keep one terminal as `oracle`
- Keep one terminal as `grid`
- Check the status of the `ORCL` database
- Kill the database `LGWR` process and watch Oracle Restart recover it
- Check the status of `ASM`
- Kill the `ASM` `LGWR` process and watch Oracle Restart recover it
- Check the status of the `DATA` disk group and the listener
- Kill the listener process and watch Oracle Restart recover it

---

## 2. Assumptions

This lab assumes:

- Oracle Restart is running
- The `ORCL` database is registered with Oracle Restart
- `ASM` is running
- The `DATA` disk group exists
- The default listener is registered with Oracle Restart

Keep two terminals open:

- one as `oracle`
- one as `grid`

That mirrors the course demo and saves you from repeatedly re-authenticating just because Oracle enjoys role separation.

---

## 3. Oracle User Terminal: Check the Database

In the first terminal, become the `oracle` user:

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

Check the database status:

```bash
srvctl status database -db orcl
```

You should see that the database is running.

---

## 4. Crash the Database and Watch It Recover

Find the database `LGWR` process:

```bash
ps -ef | grep -i lgwr | grep -i orcl
```

Kill that process using its `PID`:

```bash
kill -9 <lgwr_pid>
```

Check the database status again:

```bash
srvctl status database -db orcl
```

It may briefly report as down. Wait a little and rerun the status command until Oracle Restart brings it back:

```bash
srvctl status database -db orcl
```

The demo explicitly repeats the status check until the resource returns. Oracle Restart is automatic, not instantaneous.

---

## 5. Grid User Terminal: Check ASM

In the second terminal, become the `grid` user:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

Set the Grid Infrastructure environment:

```bash
export ORACLE_HOME=/u01/app/12.2.0/grid
export PATH=$ORACLE_HOME/bin:$PATH
```

Check the `ASM` status:

```bash
srvctl status asm
```

You should see that the `ASM` instance is running.

---

## 6. Crash ASM and Watch It Recover

Find the `ASM` `LGWR` process:

```bash
ps -ef | grep -i lgwr | grep -i asm
```

Kill the process:

```bash
kill -9 <asm_lgwr_pid>
```

Check `ASM` status again and repeat as needed until it comes back:

```bash
srvctl status asm
```

Again, the recovery is automatic but not immediate. Oracle Restart still takes a few moments to realize you have sabotaged its day.

---

## 7. Check the Disk Group and Listener

Still in the `grid` terminal, check the `DATA` disk group:

```bash
srvctl status diskgroup -diskgroup DATA
```

Check the listener:

```bash
srvctl status listener -listener LISTENER
```

You should see both resources online.

---

## 8. Crash the Listener and Watch It Recover

Find the listener process:

```bash
ps -ef | grep tnslsnr | grep LISTENER
```

Kill the listener process:

```bash
kill -9 <listener_pid>
```

Check its status until Oracle Restart starts it again:

```bash
srvctl status listener -listener LISTENER
```

If the first check says it is still down, wait a bit and run it again.

---

## 9. Finish the Lab

When the database, `ASM`, and listener are all back online:

- leave the terminals open if you are continuing to the next lab
- otherwise exit both shells

---

## 10. What You Just Finished

By the end of this lab you:

- verified the `ORCL` database was managed by Oracle Restart
- forced the database to crash and watched it recover
- forced `ASM` to crash and watched it recover
- checked the `DATA` disk group and listener
- forced the listener to crash and watched it recover

Which is a very Oracle way to build confidence: break three important things and feel reassured when the stack quietly puts them back.
