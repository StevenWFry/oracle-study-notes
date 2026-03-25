# Lab 5-1 - Creating an Oracle Database Using DBCA GUI

This lab creates the main classroom database using the DBCA graphical interface. By the end, you will have a multitenant database with one PDB, data files in `+DATA`, recovery files in `+FRA`, and just enough configuration choices to remind you why people either love or fear wizards.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `oracle` user
- Set the Oracle environment for the database home
- Start `DBCA`
- Create a single-instance container database
- Create one pluggable database
- Store data files in `+DATA`
- Store recovery files in `+FRA`
- Enable sample schemas

---

## 2. Switch to `oracle` and Set the Environment

Open a terminal and become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

Set the database software environment:

```bash
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Then start DBCA:

```bash
dbca
```

---

## 3. Choose the Creation Mode

In DBCA:

1. Select **Create a database**
2. Click **Next**
3. Choose **Advanced configuration**

The advanced path gives you more control over the storage, `CDB`, `PDB`, memory, and recovery settings. Which is precisely why the lab wants it.

---

## 4. Choose Deployment Type and Template

Select:

- **Oracle Single Instance Database**
- Template: **General Purpose or Transaction Processing**

Click **Next**.

---

## 5. Set Database Identity and Multitenant Options

Use these values:

- Global database name:

```text
orclus.oracle.com
```

- SID:

```text
ORCL
```

Create a container database and enable:

- **One PDB**
- **Local undo for PDBs**

Use this PDB name:

```text
PDB1
```

Then click **Next**.

---

## 6. Configure Database Storage

Choose the option to specify storage attributes directly rather than relying on a template-only layout.

For the database file location:

- Use `ASM`
- Browse and choose:

```text
+DATA
```

Keep **Oracle Managed Files** enabled.

Click **Next**.

---

## 7. Configure the Fast Recovery Area

Enable **Specify Fast Recovery Area**.

Set:

- Recovery area location:

```text
+FRA
```

- Recovery area size:

```text
40 GB
```

The lab intentionally increases the `FRA` size because the default value is not enough for later activities. Oracle's defaults are many things. Generous is not always one of them.

Click **Next**.

---

## 8. Verify Listener and Leave Security Options at Default

On the network page:

- Verify the default listener is selected

The listener should already be running from the Grid Infrastructure home.

On the security page:

- Leave Oracle Database Vault unselected
- Leave Oracle Label Security unselected

Click **Next**.

---

## 9. Set Memory, Character Set, and Sample Schemas

On the configuration options pages:

- Reduce the memory target to around:

```text
3000 MB
```

- Leave sizing defaults unless the lab image instructs otherwise
- Set the character set to **Unicode**
- Leave the connection mode at the default
- Select **Sample Schemas**

Then click **Next**.

---

## 10. Enable Database Express and Set Administrative Passwords

On the management page:

- Leave **Enterprise Manager Database Express** enabled
- Use port:

```text
5500
```

For the administrative users, use the same password:

```text
cloud_4u
```

This applies to users such as `SYS`, `SYSTEM`, and `PDBADMIN`.

Click **Next**.

---

## 11. Create the Database

On the database creation options page:

- Leave **Create Database** selected

Optional alternatives you may notice:

- Save as a template
- Generate database creation scripts

Those are useful, but for this lab the goal is to create the database now, not start a side quest.

Click **Next**, review the summary page, and then click **Finish**.

---

## 12. Monitor the Progress and Finish

Watch the Database Creation Progress page.

You can open:

- Activity log
- Alert log

if you want more detail while the database is being created.

When creation completes:

- Use **Password Management** only if you need to change any user passwords
- Click **Close**

---

## 13. What You Just Created

By the end of this lab you:

- Created a single-instance container database
- Set the global name to `orclus.oracle.com`
- Used SID `ORCL`
- Created one PDB named `PDB1`
- Placed database files in `+DATA`
- Placed recovery files in `+FRA`
- Set the `FRA` size to `40 GB`
- Enabled sample schemas

In other words, the lab now has an actual database instead of a lonely Oracle home waiting for meaning.
