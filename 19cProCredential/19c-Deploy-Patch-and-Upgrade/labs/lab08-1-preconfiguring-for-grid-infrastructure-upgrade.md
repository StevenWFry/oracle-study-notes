# Lab 8-1 - Preconfiguring for Grid Infrastructure Upgrade

This lab performs the operating system and environment preparation needed before upgrading Oracle Grid Infrastructure to `19c`. It is the part where you do all the sensible setup work before asking Oracle to move an `ASM`-backed high-availability stack into a new home without developing opinions.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `root` user
- Install or validate the `19c` preinstall `RPM`
- Create a `grid` limits file from the `oracle` limits template
- Remove obsolete limits files from earlier releases
- Prepare the target `19c` Grid home directory
- Switch to the `oracle` user
- Stop the `ORCL` and `CDB122` databases before the Grid upgrade

---

## 2. Assumptions

This lab assumes:

- Oracle Grid Infrastructure `12.2` is already installed
- Oracle Database software is already installed
- The classroom databases `ORCL` and `CDB122` already exist
- Both databases use `ASM`
- The `19c` preinstall `RPM` is already staged locally
- The future `19c` Grid home or gold image location is known

The transcript does not preserve the exact staged `RPM` filename cleanly, so this lab uses a placeholder for the package name where needed.

---

## 3. Switch to `root`

Open a terminal and become `root`:

```bash
su -
```

Use the lab password:

```text
Oracle
```

---

## 4. Install or Validate the 19c Preinstall RPM

Go to the staged `RPM` location and install the `19c` preinstall package:

```bash
cd /stage/RPM
rpm -ivh <oracle-database-preinstall-19c-rpm>
```

If the package is already installed, `rpm` reports that fact. In the classroom demo, this was normal because the step had already been validated once. Oracle is perfectly happy to tell you something is already present with all the warmth of a parking ticket.

---

## 5. Refresh the Security Limits Files for `oracle` and `grid`

Go to the limits directory:

```bash
cd /etc/security/limits.d
```

You should already have a `19c` limits file for the `oracle` user created by the preinstall package. Create a matching file for the `grid` user by copying the Oracle version and replacing the username:

```bash
cp oracle-database-preinstall-19c.conf grid-database-preinstall-19c.conf
sed -i 's/^oracle/grid/' grid-database-preinstall-19c.conf
```

Verify that both files now exist:

```bash
ls *19c*.conf
```

If older limits files from earlier releases still exist and the lab guide tells you to remove them, do that now so the `19c` versions are the active ones:

```bash
rm -f <older_limit_files>
```

The transcript confirms the cleanup step but does not preserve the exact filenames cleanly, so use the actual names present in your classroom image.

---

## 6. Prepare the Target 19c Grid Home Directory

Create the target directory for the patched `19c` Grid home if it does not already exist:

```bash
mkdir -p /u01/app/19c/grid
```

Set ownership correctly:

```bash
chown -R grid:oinstall /u01/app/19c/grid
```

This is the location that will later receive the patched `19c` gold image contents before the upgrade begins.

---

## 7. Switch to the `oracle` User and Stop the Databases

Exit the `root` shell and become the `oracle` user:

```bash
exit
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the environment for the existing `12.2` database home:

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Stop the `ORCL` database:

```bash
srvctl stop database -db orcl
```

Stop the `CDB122` database:

```bash
srvctl stop database -db cdb122
```

If either database is already stopped, `srvctl` tells you. That is fine. The point is that nothing using `ASM` should still be running when the Grid Infrastructure upgrade begins.

You can verify status if desired:

```bash
srvctl status database -db orcl
srvctl status database -db cdb122
```

---

## 8. Finish the Lab

When both databases are confirmed down:

- leave the terminal ready for the next upgrade step
- or exit the `oracle` shell if you want a clean start for the next lab

---

## 9. What You Just Finished

By the end of this lab you:

- validated the `19c` preinstall package
- refreshed the limits configuration for both `oracle` and `grid`
- prepared the future `19c` Grid home directory
- shut down the databases that depend on `ASM`

Which means the environment is now finally behaving like it understands that upgrading the storage stack while active databases are still using it would be a terrible personality trait.
