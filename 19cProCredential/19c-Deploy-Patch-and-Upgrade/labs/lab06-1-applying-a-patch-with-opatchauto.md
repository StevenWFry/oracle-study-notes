# Lab 6-1 - Applying a Patch with OPatchAuto

This lab applies the staged pre-upgrade patch bundle to both the Grid Infrastructure home and the database home by using `OPatchAuto`. Oracle built this tool so you would not have to manually choreograph every stop, unlock, patch, relock, and restart step like a stressed-out theater stage manager.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `grid` user
- Unzip the staged patch bundle
- Review the patch README
- Switch to `root`
- Run `OPatchAuto` against the staged patch directory
- Review the generated log location

---

## 2. Assumptions

This lab assumes:

- Oracle Grid Infrastructure is already installed
- Oracle Database software is already installed
- The classroom database already exists
- The `grid`, `oracle`, and `root` accounts are available
- The patch zip file is already staged in the lab image

The transcript does not preserve the exact staged patch filename cleanly, so this lab uses placeholders where the classroom image may vary. Replace items such as `<patch_bundle>.zip` and `<patch_dir>` with the actual names in your environment.

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

Change to the staged patch directory and list the contents:

```bash
cd /stage/patches
ls
```

If your classroom image uses a different staging path, use that path instead. Oracle does not care what the folder is called. It cares very deeply whether you are in the right one.

---

## 4. Unzip the Patch Bundle and Inspect It

Unzip the staged patch zip file:

```bash
unzip -q <patch_bundle>.zip
```

After extraction, inspect the new directory:

```bash
ls
cd <patch_dir>
ls
cd <patch_dir>
ls
```

The demo calls out the classic Oracle double-directory pattern, where the zip extracts to a top-level patch number and then another directory with the same name underneath. Because one layer of unnecessary nesting is apparently part of the support contract.

Review the README before continuing:

```bash
less README.txt
```

Read the prerequisites and warnings carefully. The patch README is where Oracle explains what you should have done before touching anything valuable.

---

## 5. Switch to `root` and Run `OPatchAuto`

Become `root`:

```bash
su -
```

Use the lab password:

```text
Oracle
```

Set the Grid Infrastructure environment and `OPatch` path:

```bash
export ORACLE_HOME=/u01/app/12.2.0/grid
export PATH=$ORACLE_HOME/OPatch:$PATH
```

Apply the patch bundle with `OPatchAuto`:

```bash
opatchauto apply /stage/patches/<patch_dir>/<patch_dir>
```

`OPatchAuto` performs the orchestration that you would otherwise have to do manually:

- prerequisite checks
- service shutdowns
- binary patching
- required restart work

The command may run for several minutes. That is normal. Oracle patching is never in a hurry, but it is extremely willing to consume yours.

---

## 6. Review the Log Location

When `OPatchAuto` starts, it reports the log file location. The exact path and timestamp will vary by system.

If you want to inspect the log output after completion, use the path displayed in your terminal. Typical log areas include:

```text
/u01/app/12.2.0/grid/cfgtoollogs/opatchauto
```

You can view a specific log with a command such as:

```bash
cat <log_file_path>
```

---

## 7. Finish the Lab

When the patch completes successfully:

- confirm you have the shell prompt back
- review the log if needed
- exit the terminal

---

## 8. What You Just Finished

By the end of this lab you:

- unpacked the staged patch bundle
- reviewed the patch README
- used `OPatchAuto` to patch the Grid Infrastructure and database homes
- confirmed where the patch logs were written

Which is the pleasant version of patching, where Oracle does most of the heavy lifting and you mostly supervise the blast radius.
