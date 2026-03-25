# Lab 10-1 - Installing the 19c Database Home for Upgrade

This lab installs the patched `19c` database software home that will be used for the Oracle Database upgrade. The gold image already exists from the patching exercises, so the goal here is not to build a home from scratch. The goal is to unzip it into the target location, run `runInstaller`, and finish with a usable software-only `19c` home instead of a directory full of potential.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Validate the `oracle` user's group memberships
- Create the target `19c` database home directory
- Unzip the patched gold image into that home
- Run `runInstaller`
- Choose a software-only single-instance Enterprise Edition install
- Let the installer run the required root scripts automatically

---

## 2. Assumptions

This lab assumes:

- The patched `19c` database gold image already exists from the patching chapter
- The `oracle` and `root` users already exist
- The old `12.2` database home is still present
- The future `19c` database home directory does not yet exist, or is empty and ready to use

The exact gold-image zip filename is not preserved clearly in the transcript, so this lab uses a placeholder where the classroom image may vary.

---

## 3. Check the Oracle User's Group Memberships

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Verify the user and groups:

```bash
id
```

Make sure the expected database administrative groups are present. If they are not, stop there and fix the ownership model before pretending the installer will become emotionally supportive.

---

## 4. Create the Target 19c Home as `root`

Switch to `root`:

```bash
su -
```

Use the lab password:

```text
Oracle
```

Create the target `19c` database home directory:

```bash
mkdir -p /u01/app/oracle/product/19c/dbhome_1
```

Set ownership correctly:

```bash
chown -R oracle:oinstall /u01/app/oracle/product/19c/dbhome_1
```

Return to the `oracle` user:

```bash
exit
```

---

## 5. Unzip the Gold Image into the New Home

Change to the new home:

```bash
cd /u01/app/oracle/product/19c/dbhome_1
```

Unzip the staged database gold image:

```bash
unzip -q /u01/stage/gold-images/<db_gold_image_zip>
```

Wait for extraction to finish, then verify the contents:

```bash
ls
```

If the directory still looks empty, the unzip did not happen. Oracle will not reward your confidence for skipping that check.

---

## 6. Run the Installer

Start the installer from the new home:

```bash
./runInstaller
```

In the installer:

1. Select **Set Up Software Only**
2. Choose **Single instance database installation**
3. Choose **Enterprise Edition**
4. Confirm Oracle base:

```text
/u01/app/oracle
```

5. Confirm the Oracle home is the new `19c` home you launched from
6. Verify the OS group mappings
7. Choose automatic root script execution
8. Provide the `root` password:

```text
Oracle
```

9. Let the prerequisite checks complete
10. Review the summary
11. Click **Install**

When the installer prompts to run the root scripts, click **Yes** so it can execute them automatically.

---

## 7. Finish the Lab

When the installer reports success:

- click **Close**

Optional quick environment check:

```bash
export ORACLE_HOME=/u01/app/oracle/product/19c/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
sqlplus -v
```

---

## 8. What You Just Finished

By the end of this lab you:

- validated the `oracle` user's group memberships
- created the target `19c` database home
- unpacked the patched gold image into that home
- completed a software-only database-home install

Which means the future upgrade target now exists as an actual Oracle home instead of a plan written in optimistic tense.
