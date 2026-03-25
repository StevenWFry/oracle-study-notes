# Lab 6-2 - Applying a Patch Manually with OPatch

This lab applies the same staged patch manually instead of using `OPatchAuto`. That means you handle the sequencing yourself: stop the managed resources, unlock the Grid home, patch Grid Infrastructure, relock it, patch the database home, restart the database, and then run `datapatch`. In other words, Oracle removes the guardrails and quietly watches what you do with your life.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `grid` user and stage the patch
- Review the patch README
- Stop the database home resources with `srvctl`
- Unlock the Grid home with `rootcrs.sh -prepatch`
- Apply the patch to the Grid home with `opatch`
- Relock the Grid home with `rootcrs.sh -postpatch`
- Apply the patch to the database home with `opatch`
- Restart the database home resources
- Run `datapatch`

---

## 2. Assumptions

This lab assumes:

- Oracle Grid Infrastructure is already installed
- Oracle Database software is already installed
- Oracle Restart is managing the single-instance database
- The database already exists and uses pluggable databases
- The staged patch zip file is already available in the lab image

The source demo includes one environment-specific copy command and several wrapped shell lines that do not survive transcription cleanly. This lab preserves the reliable steps and uses placeholders where the exact classroom filename is not recoverable.

---

## 3. Stage the Patch as `grid`

Open a terminal and become the `grid` user:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

Go to the staged patch area:

```bash
cd /stage/patches
ls
```

Unzip the patch bundle:

```bash
unzip -q <patch_bundle>.zip
```

Inspect the extracted patch directory and review the README:

```bash
cd <patch_dir>
ls
cd <patch_dir>
ls
less README.txt
```

Do not skip the README. That file is where Oracle explains prerequisites, conflicts, and post-patch actions before you learn them the expensive way.

---

## 4. Stop the Database Home and Unlock the Grid Home

Become `root`:

```bash
su -
```

Use the lab password:

```text
Oracle
```

Set the Grid Infrastructure environment:

```bash
export ORACLE_HOME=/u01/app/12.2.0/grid
export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$PATH
```

Stop the Oracle Restart-managed resources that run from the database home and save their state:

```bash
srvctl stop home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome_patch.state
```

Unlock the Grid home for patching:

```bash
cd /u01/app/12.2.0/grid/crs/install
./rootcrs.sh -prepatch
```

This stops the Grid stack and unlocks the Grid home so the binaries can be modified. Which is Oracle's way of saying, "You may now touch the sacred files."

---

## 5. Apply the Patch to the Grid Home

Exit back to the `grid` user:

```bash
exit
```

Set the Grid home environment again just to be explicit:

```bash
export ORACLE_HOME=/u01/app/12.2.0/grid
export PATH=$ORACLE_HOME/OPatch:$PATH
```

Change to the patch directory:

```bash
cd /stage/patches/<patch_dir>/<patch_dir>
```

Apply the patch:

```bash
opatch apply
```

When prompted, answer `y` to continue.

The transcript notes two confirmation prompts during the manual apply. That is Oracle's way of ensuring you are still committed to your choices.

---

## 6. Relock the Grid Home

Switch back to `root`:

```bash
su -
```

Run the post-patch step for Grid Infrastructure:

```bash
cd /u01/app/12.2.0/grid/crs/install
./rootcrs.sh -postpatch
```

This relocks the Grid home and restarts the Grid stack.

---

## 7. Apply the Patch to the Database Home

Switch to the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the database environment:

```bash
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$PATH
```

Change to the staged patch directory:

```bash
cd /stage/patches/<patch_dir>/<patch_dir>
```

Apply the patch to the database home:

```bash
opatch apply
```

When prompted, answer `y` to continue.

---

## 8. Restart the Database Home and Open the PDBs

Start the managed resources from the saved state file:

```bash
srvctl start home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome_patch.state
```

Then connect to the database:

```bash
sqlplus / as sysdba
```

Verify that the pluggable databases are open. If needed, open them:

```sql
show pdbs;
alter pluggable database all open;
show pdbs;
exit
```

For `datapatch`, the PDBs that need SQL patching should be open read/write. Oracle enjoys making this your problem if you forget.

---

## 9. Run `datapatch`

From the `oracle` shell, run:

```bash
datapatch -verbose
```

After it finishes, reconnect and verify the PDBs again if your lab guide wants an explicit check:

```bash
sqlplus / as sysdba
```

```sql
show pdbs;
exit
```

---

## 10. Finish the Lab

When everything completes successfully:

- confirm the database is open
- confirm the PDBs are read/write as expected
- exit all terminal windows

---

## 11. What You Just Finished

By the end of this lab you:

- manually unlocked the Grid home
- patched the Grid home with `opatch`
- relocked the Grid home
- patched the database home with `opatch`
- restarted the database resources
- ran `datapatch`

Which is the full manual patching path, for the days when Oracle gives you the wrench and trusts you not to hit the wrong pipe.
