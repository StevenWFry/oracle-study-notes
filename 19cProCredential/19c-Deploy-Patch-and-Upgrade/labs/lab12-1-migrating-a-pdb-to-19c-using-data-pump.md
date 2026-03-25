# Lab 12-1 - Migrating a PDB to 19c Using Data Pump

This lab uses `Oracle Data Pump` to move a source `PDB` from the `12.2` environment into a new `19c` `PDB`. The exercise follows the course demo pattern: export the source `PDB`, copy the transportable datafile into `ASM`, create an empty target `PDB`, and import the metadata into the new home. It is the "let's move house instead of renovating during occupancy" version of an upgrade.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Open the source `12.2` `PDB`
- Set its user tablespace to `READ ONLY`
- Run a full transportable export
- Copy the transported datafile into target `ASM` storage
- Create an empty target `19c` `PDB`
- Import the source data into the new `PDB`
- Validate the migrated objects and row counts

---

## 2. Assumptions

This lab assumes:

- `CDB122` exists in the `12.2` environment
- the source `PDB` is named `PDBEXPORT`
- `ORCL` has already been upgraded to `19c`
- the target system uses `ASM`
- you have a target `19c` home and a working source `12.2` home
- sample schemas exist in the source `PDB`

If your classroom image uses different paths or aliases, substitute the values from your own environment instead of copy-pasting your way into a support call.

---

## 3. Prepare the Source PDB

Open a terminal and become `oracle`:

```bash
su - oracle
```

Set the environment for the source `12.2` `CDB`:

```bash
export ORACLE_SID=CDB122
. oraenv
```

Connect and open the `PDB`s:

```sql
sqlplus / as sysdba
alter pluggable database all open;
show pdbs
```

Switch to the source `PDB`:

```sql
alter session set container=PDBEXPORT;
```

Make the user tablespace read only. In the demo environment, the only user tablespace is `USERS`:

```sql
alter tablespace users read only;
```

Capture a quick validation count you can compare after import:

```sql
select count(*)
from hr.employees;
```

Capture the datafile location for the transported tablespace:

```sql
select file_name
from dba_data_files
where tablespace_name = 'USERS';
```

Make a note of that datafile path. You will need it when copying the transported file to the target.

---

## 4. Create or Confirm a Data Pump Directory

If your environment already has a usable directory object, you can reuse it. Otherwise create one inside the source `PDB`:

```sql
create or replace directory dp_mig as '/home/oracle/dpump/pdbexport';
grant read, write on directory dp_mig to system;
```

Exit SQL*Plus:

```sql
exit
```

Create the operating system directory if needed:

```bash
mkdir -p /home/oracle/dpump/pdbexport
```

---

## 5. Export the Source PDB with Full Transportable Mode

Run the export from the source `PDB` service:

```bash
expdp system@PDBEXPORT \
  full=y \
  transportable=always \
  directory=dp_mig \
  dumpfile=pdbexport_ft.dmp \
  logfile=pdbexport_ft.log
```

Allow the export to finish.

Review the output and note the transported datafile names. In the class demo, the `USERS` datafile is the one that matters.

---

## 6. Copy the Transported Datafile into ASM

Open a second terminal and become `grid`:

```bash
su - grid
```

Set the `ASM` environment:

```bash
export ORACLE_SID=+ASM
. oraenv
```

Start `asmcmd`:

```bash
asmcmd
```

Copy the transported source datafile into a staging location in `ASM`, such as `+FRA`:

```bash
cp <source_users_datafile> +FRA
```

Use the actual path you captured earlier from `DBA_DATA_FILES`.

Exit `asmcmd` for now:

```bash
exit
```

---

## 7. Return the Source Tablespace to Read/Write

Go back to the first terminal as `oracle`:

```sql
sqlplus / as sysdba
alter session set container=PDBEXPORT;
alter tablespace users read write;
exit
```

The source `PDB` is no longer frozen. Civilization may resume.

---

## 8. Create the Target PDB in 19c

Still as `oracle`, set the environment for the target `19c` database:

```bash
export ORACLE_SID=ORCL
. oraenv
```

Connect as `SYSDBA`:

```sql
sqlplus / as sysdba
```

Create an empty target `PDB`:

```sql
create pluggable database pdbimport
  admin user pdbadmin identified by "cloud_4u"
  file_name_convert = ('pdbseed', 'pdbimport');
```

Open it:

```sql
alter pluggable database pdbimport open;
alter session set container=PDBIMPORT;
```

Create a directory object for the import dump file:

```sql
create or replace directory dp_mig as '/home/oracle/dpump/pdbimport';
grant read, write on directory dp_mig to system;
```

Query the datafile layout so you know the target `ASM` directory for this `PDB`:

```sql
select file_name
from dba_data_files
order by file_name;
```

Make a note of the `PDBIMPORT` path in `ASM`. You will use it to place the transported `USERS` datafile where the target `PDB` expects it.

Exit SQL*Plus:

```sql
exit
```

Create the target operating system directory if needed:

```bash
mkdir -p /home/oracle/dpump/pdbimport
```

---

## 9. Copy the Dump File to the Target Import Directory

Copy the dump file from the source export directory to the target import directory:

```bash
cp /home/oracle/dpump/pdbexport/pdbexport_ft.dmp /home/oracle/dpump/pdbimport/
```

If your environment uses a different source path, adjust accordingly.

---

## 10. Move the Transported Datafile into the Target PDB Location

Return to the `grid` terminal, or switch back to `grid` if needed:

```bash
su - grid
export ORACLE_SID=+ASM
. oraenv
asmcmd
```

Copy the staged file from `+FRA` to the target `PDBIMPORT` datafile directory, using the target path you discovered earlier:

```bash
cp +FRA/<copied_users_file> +DATA/ORCL/<pdbimport_guid>/DATAFILE/users01.dbf
```

Use your actual:

- copied file name in `+FRA`
- target `PDBIMPORT` `ASM` directory

Exit `asmcmd`:

```bash
exit
```

---

## 11. Import into the Target PDB

Back as `oracle`, run the import while connected to the target `PDB` service:

```bash
impdp system@PDBIMPORT \
  directory=dp_mig \
  dumpfile=pdbexport_ft.dmp \
  transport_datafiles='+DATA/ORCL/<pdbimport_guid>/DATAFILE/users01.dbf' \
  logfile=pdbimport_ft.log
```

For file-based full transportable import, Oracle infers the transportable full import behavior from `TRANSPORT_DATAFILES`.

Allow the import to complete.

Review the log for warnings or failed object creations. In the course demo, a few application-level incompatibilities appeared, which is exactly why migrations need validation instead of applause.

---

## 12. Validate the Imported PDB

Connect to the target database:

```sql
sqlplus / as sysdba
alter session set container=PDBIMPORT;
```

Check that the `USERS` tablespace exists:

```sql
select tablespace_name, status
from dba_tablespaces
where tablespace_name = 'USERS';
```

Check the datafile location:

```sql
select file_name
from dba_data_files
where tablespace_name = 'USERS';
```

Validate the same row count you captured earlier:

```sql
select count(*)
from hr.employees;
```

If the count matches and the objects are present, the migration landed correctly.

Exit SQL*Plus:

```sql
exit
```

---

## 13. What You Just Finished

By the end of this lab you:

- exported a source `12.2` `PDB`
- moved its transportable datafile into `ASM`
- created an empty target `19c` `PDB`
- imported the source application metadata and data into that target
- validated the migrated tablespace, datafile, and sample data

Which means the course ends the way Oracle courses often do: with you holding a working database, several log files, and just enough confidence to become dangerous in a real environment.
