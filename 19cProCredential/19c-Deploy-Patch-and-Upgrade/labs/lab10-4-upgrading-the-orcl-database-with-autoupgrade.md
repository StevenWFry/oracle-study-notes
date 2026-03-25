# Lab 10-4 - Upgrading the ORCL Database with AutoUpgrade

This lab performs the actual database upgrade of `ORCL` from `12.2` to `19c` using `AutoUpgrade` in `deploy` mode. The analyze and fixups are already done. The backup exists. The `ARCHIVELOG` setting is corrected. Now Oracle gets to do the part where it changes the dictionary, upgrades the `CDB` and `PDB`s, and leaves behind enough logs to keep a storage team employed.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Return to the `AutoUpgrade` config directory
- Run the same config file in `deploy` mode
- Use the AutoUpgrade console to monitor progress
- Watch the database move from prechecks to real upgrade work
- Note the guaranteed restore point behavior
- Confirm successful completion

---

## 2. Assumptions

This lab assumes:

- Lab 10-1 is complete
- Lab 10-2 is complete
- Lab 10-3 is complete
- The `ORCL` database is in a healthy pre-upgrade state
- `ARCHIVELOG` is enabled
- The `FRA` has enough free space for the guaranteed restore point and flashback logging

---

## 3. Switch to `oracle` and Go to the Config Directory

Open a terminal and become the `oracle` user if needed:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the shell for the target `19c` home used to run AutoUpgrade:

```bash
export ORACLE_HOME=/u01/app/oracle/product/19c/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Go to the config directory:

```bash
cd /home/oracle/autoupgrade/orcl
```

Verify the config file exists:

```bash
ls
```

You should see:

```text
config_orcl_122_2_19.cfg
```

---

## 4. Run AutoUpgrade in Deploy Mode

Start the actual upgrade:

```bash
$ORACLE_HOME/jdk/bin/java -jar $ORACLE_HOME/rdbms/admin/autoupgrade.jar -config config_orcl_122_2_19.cfg -mode deploy
```

This is the same command structure used in the earlier lab, except the mode is now:

```text
deploy
```

The deploy path performs:

- prechecks
- validation of pending fixups
- creation of the restore point
- the actual upgrade
- post-upgrade processing

This is where the training wheels come off and Oracle starts rewriting metadata for real.

---

## 5. Monitor the Job in the AutoUpgrade Console

While the job runs, use the console command:

```text
lsj -r
```

This shows currently running jobs and their stage.

Early on, you may see it validating:

- prechecks
- fixups

Later, when the real upgrade is underway, you should see stages that indicate:

- database upgrade execution
- `CDB$ROOT` activity
- `PDB` upgrade work

The transcript explicitly calls out that the active component can move between:

- `CDB$ROOT`
- the `PDB`
- back to `CDB$ROOT`

Which is normal. Oracle upgrades containers in the sequence it thinks is sensible, and then leaves you to read the job status like weather radar.

---

## 6. Let the Deploy Complete

Allow the deploy job to run to completion.

When it finishes successfully, the console output should report:

- one finished job
- no failed jobs
- no pending jobs

It will also report that a restore point exists and should eventually be dropped after post-upgrade validation is complete.

That restore point is not a decorative souvenir. It exists so the upgrade can be rolled back if the environment fails later validation.

---

## 7. Review the Output Files

After the job completes, review the generated status files in the AutoUpgrade log area:

```bash
cd /home/oracle/autoupgrade
ls
```

The exact directory numbering varies by run, but the deploy job creates its own log and status tree.

Review the summary files and status HTML if desired.

If you need to troubleshoot, the log tree is where Oracle leaves the evidence instead of just the attitude.

---

## 8. Do Not Drop the Restore Point Yet

The transcript makes an important operational note:

- AutoUpgrade leaves the guaranteed restore point in place by default

Do **not** drop it immediately.

Wait until:

- the upgrade is validated
- post-upgrade tasks are complete
- the environment is confirmed healthy

Only then should the restore point be removed. Getting rid of your rollback point before validation is complete is the kind of thing people later describe as "a learning experience."

---

## 9. What You Just Finished

By the end of this lab you:

- ran `AutoUpgrade` in `deploy` mode
- monitored the running job with `lsj -r`
- watched the `CDB` and `PDB` upgrade phases execute
- completed the `ORCL` upgrade from `12.2` to `19c`
- preserved the guaranteed restore point for later validation

Which means the actual upgrade is now done, and Oracle is ready to move on to the equally important chapter where you prove the upgraded database still deserves your trust.
