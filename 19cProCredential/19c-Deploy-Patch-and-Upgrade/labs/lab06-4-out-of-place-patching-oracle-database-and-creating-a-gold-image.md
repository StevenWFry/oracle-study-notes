# Lab 6-4 - Out-of-Place Patching Oracle Database and Creating a Gold Image

This lab builds a new out-of-place `19c` database home, applies the `19.20` `RU`, creates a database gold image, and then optionally removes the temporary working homes to reclaim space. Because once the gold images exist, there is no point keeping extra unpacked software around unless your hobby is feeding disk alerts.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Prepare a new `19c` database home
- Unzip the `19.20` patch into a clean directory
- Extract the base `19.3` database image
- Replace the bundled `OPatch`
- Use `runInstaller` to save a response file
- Cancel the GUI install and rerun it silently with `-applyRU`
- Create a database gold image
- Optionally remove the temporary working homes after the gold images exist

---

## 2. Assumptions

This lab assumes:

- The `oracle` and `root` users already exist
- The `19.3` database image zip is already staged
- The `19.20` patch bundle is already staged
- The newer `OPatch` zip is already staged
- The destination directories from the previous lab already exist, or you can create them now

The exact staged filenames are not preserved cleanly in the transcript, so this lab uses placeholders such as `<db_19c_zip>`, `<ru_patch_zip>`, and `<opatch_zip>`.

---

## 3. Prepare the Target Directories

If the `19c` database home and patch-unzip directory were not already created in the previous lab, create them now as `root`:

```bash
su -
mkdir -p /u01/app/oracle/product/19c/dbhome_1
mkdir -p /u01/stage/db_ru_1920
chown -R oracle:oinstall /u01/app/oracle/product/19c/dbhome_1 /u01/stage/db_ru_1920
```

Then switch to the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

---

## 4. Unzip the Patch and the Base `19.3` Database Image

Unzip the `19.20` patch into its clean directory:

```bash
cd /u01/stage/db_ru_1920
unzip -q /stage/19c_patches/<ru_patch_zip>
```

Extract the base `19.3` database image into the new home:

```bash
cd /u01/app/oracle/product/19c/dbhome_1
unzip -q /stage/19c_software/<db_19c_zip>
```

---

## 5. Replace the Bundled `OPatch`

Remove the bundled `OPatch` directory:

```bash
rm -rf /u01/app/oracle/product/19c/dbhome_1/OPatch
```

Unzip the staged newer `OPatch`:

```bash
unzip -q /stage/19c_patches/<opatch_zip> -d /u01/app/oracle/product/19c/dbhome_1
```

---

## 6. Use the GUI Once to Save a Response File

Launch the installer:

```bash
cd /u01/app/oracle/product/19c/dbhome_1
./runInstaller
```

In the wizard:

1. Select **Set Up Software Only**
2. Choose **Single instance database installation**
3. Choose **Enterprise Edition**
4. Verify Oracle base:

```text
/u01/app/oracle
```

5. Verify the OS group mappings
6. Choose automatic root script execution
7. Let the prerequisite checks complete
8. On the summary page, click **Save Response File**
9. Save the file with a memorable name such as:

```text
db_1920.rsp
```

10. Return to the summary page and click **Cancel**
11. Confirm the exit

Just like the Grid lab, the GUI pass is only there to capture a clean response file. Installers apparently enjoy being used as response-file vending machines.

---

## 7. Apply the `RU` Silently to the New Database Home

From the `oracle` shell, run the silent build of the patched home:

```bash
cd /u01/app/oracle/product/19c/dbhome_1
./runInstaller -silent -responseFile /home/oracle/db_1920.rsp -applyRU /u01/stage/db_ru_1920/<ru_patch_dir>
```

If your response file has a different name or location, use that full path.

Because the installer was configured for automatic root script execution, it will eventually prompt for the `root` password. Enter:

```text
Oracle
```

The demo notes that the prompt appears after roughly 10 minutes in the classroom environment.

---

## 8. Create the Database Gold Image

After the patched `19c` database home completes successfully, create a gold image:

```bash
cd /u01/app/oracle/product/19c/dbhome_1
./runInstaller -createGoldImage -destinationLocation /u01/stage/gold-images -silent
```

This creates a reusable zip image of the patched database home for later upgrade exercises.

---

## 9. Optional Cleanup to Reclaim Space

Once both the Grid and database gold images exist, the classroom demo removes the temporary working homes to save disk space.

If your lab guide tells you to do the same, switch to `root` and remove only the disposable working homes, not the gold image zip files:

```bash
su -
rm -rf /u01/app/19c/grid
rm -rf /u01/app/oracle/product/19c/dbhome_1
```

Do not perform this cleanup unless the gold images were created successfully and you no longer need the unpacked working homes. Deleting the only copy of the patched home is a bold lifestyle choice, not a best practice.

---

## 10. What You Just Finished

By the end of this lab you:

- built a new out-of-place `19c` database home
- patched it from `19.3` to `19.20`
- captured the installer choices in a response file
- created a reusable database gold image
- optionally cleaned up the temporary working homes

Which means the upgrade labs can now use gold images instead of redoing the same patching work from scratch like a machine designed by regret.
