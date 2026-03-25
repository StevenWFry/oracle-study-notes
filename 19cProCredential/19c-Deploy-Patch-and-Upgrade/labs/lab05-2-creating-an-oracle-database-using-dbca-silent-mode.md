# Lab 5-2 - Creating an Oracle Database Using DBCA Silent Mode

This lab creates another database using `DBCA` in silent mode. The point is not to admire the command line for its own sake. The point is to create a repeatable database build that can be used later in upgrade exercises without re-clicking the entire GUI like a medieval clerk.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `oracle` user
- Review the provided `DBCA` shell script
- Run the script
- Watch the silent-mode database creation progress
- Finish with a second database ready for later upgrade practice

---

## 2. Switch to `oracle`

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

If the environment variables for the database home are not already set by the profile, set them now:

```bash
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

---

## 3. Review the Provided Script

The practice environment provides a script named:

```text
create_CDB122.sh
```

Review it before running it:

```bash
cat create_CDB122.sh
```

From the course transcript, the script does two main things:

- Deletes the target database if it already exists
- Runs `DBCA` in silent mode to create a fresh replacement database

That second database is used later for upgrade practice, so this is not random duplication. It is future suffering with better planning.

---

## 4. Run the Script

Execute the script:

```bash
./create_CDB122.sh
```

This starts the silent `DBCA` workflow.

The process takes a few minutes, so do not panic just because the terminal is busy and Oracle has briefly chosen introspection over speed.

---

## 5. Monitor the Progress

As the script runs, you should see progress output from `DBCA`, including:

- Percentage progress
- Registration of the database with Oracle Restart
- Confirmation that database creation completed successfully

If the script is written defensively, it may first remove a pre-existing copy of the target database and only then build the new one.

---

## 6. What You Just Finished

By the end of this lab you:

- Reviewed the `create_CDB122.sh` script
- Ran `DBCA` in silent mode
- Recreated the target database automatically
- Finished with a second database ready for later upgrade-related exercises

This is the part where database creation stops being a wizard performance and starts becoming something you can repeat on purpose, which is usually a sign that the adults have entered the room.
