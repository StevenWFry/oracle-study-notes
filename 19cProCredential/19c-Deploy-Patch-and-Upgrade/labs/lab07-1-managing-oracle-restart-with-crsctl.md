# Lab 7-1 - Managing Oracle Restart with CRSCTL

This lab uses `crsctl` to inspect, stop, and start Oracle Restart itself. The database, listener, and `ASM` resources may look important, but in this lab the star of the show is the local high-availability stack that supervises all of them.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Open a terminal as the `grid` user
- Set the Grid Infrastructure environment
- Check whether Oracle Restart is running
- View the Oracle Restart release and configuration
- Stop Oracle Restart
- Start Oracle Restart again
- Verify that the stack is back online

---

## 2. Assumptions

This lab assumes:

- Oracle Grid Infrastructure for a standalone server is already installed
- Oracle Database software is already installed
- The `ORCL` database already exists
- The `grid` and `oracle` users already exist

The transcript suggests keeping one terminal as `grid` and another as `oracle`. For this first lab, the `grid` terminal is the only one you actually need.

---

## 3. Switch to the `grid` User

Open a terminal and become the `grid` user:

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

If your lab image uses a different Grid home path, adjust the value accordingly.

---

## 4. Check Oracle Restart

Check whether Oracle Restart is running:

```bash
crsctl check has
```

Display the local release version:

```bash
crsctl query has releaseversion
```

Show whether Oracle Restart is enabled for automatic startup:

```bash
crsctl config has
```

The demo also does a quick process-level check before and after the stop/start cycle. That is optional. The real signal is the `crsctl` output, not your ability to win a `grep` contest against Oracle daemons.

---

## 5. Stop Oracle Restart

Stop Oracle Restart:

```bash
crsctl stop has
```

This may take a few minutes. Unlike the start operation, the stop operation usually shows resources being stopped in dependency order. Oracle is much more chatty when it is taking things down than when it is bringing them back.

After the command returns, confirm that the stack is down:

```bash
crsctl check has
```

---

## 6. Start Oracle Restart Again

Start Oracle Restart:

```bash
crsctl start has
```

The start command typically does not list every component as it comes online. It just tells you the service started and then leaves you to trust the machinery like some kind of optimist.

Give the stack a little time, then check again:

```bash
crsctl check has
```

If the first check runs too quickly, wait a moment and run it again.

---

## 7. Finish the Lab

When Oracle Restart reports as online again:

- leave the terminal open if you are continuing to the next lab
- otherwise exit the shell

---

## 8. What You Just Finished

By the end of this lab you:

- checked the status of Oracle Restart
- viewed the local release version
- confirmed whether automatic startup was enabled
- stopped Oracle Restart
- started Oracle Restart again

Which is useful because you do not really manage Oracle Restart until you have been willing to turn it off and then make sure it comes back without drama.
