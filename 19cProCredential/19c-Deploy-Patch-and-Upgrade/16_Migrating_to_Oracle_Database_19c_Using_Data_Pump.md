## Lesson 16 - Migrating to Oracle Database 19c Using Data Pump (when "just move the data" is actually the sane option)

Not every trip to `19c` should be a direct upgrade. Sometimes the cleaner answer is to build a fresh target database, create an empty `PDB`, and move the data with `Oracle Data Pump`. This lesson covers the migration path Oracle documents for that job, including full transportable export/import, network-link imports, and the multitenant details that love to punish sloppy assumptions.

By the end of this lesson, you should be able to:

- Explain when a `Data Pump` migration is preferable to a direct upgrade
- Describe the file-based and `NETWORK_LINK` migration paths
- Identify the roles and prerequisites required for full transportable migration
- Create or prepare an empty target `PDB` for import
- Recognize the major follow-up checks after the import finishes

---

## 1. Upgrade vs Migration (same destination, very different emotional damage)

A direct upgrade:

- keeps the same database and upgrades its Oracle metadata in place
- upgrades the data dictionary, compiled objects, and internal structures
- leaves user data in place while the software and metadata move forward

A migration with `Oracle Data Pump`:

- creates a new target database or `PDB`
- exports metadata and data from the source
- imports that content into the new target
- avoids doing an in-place metadata upgrade on the original database

This is useful when:

- you want a clean `19c` target
- you are moving into multitenant architecture
- you want to change storage layout
- you are changing more than one thing at once and would like your risk to be organized instead of theatrical

Oracle explicitly documents `Data Pump` migration as an upgrade alternative.

---

## 2. The Basic Data Pump Migration Flow

At a high level, the process is:

1. Export from the source database or source `PDB`
2. Create the target `19c` database and empty target `PDB`
3. Move the dump file and, if using transportable mode, the datafiles
4. Import into the target `PDB`
5. Validate the result and clean up incompatibilities

For multitenant migrations, the empty `PDB` is the landing pad. Oracle builds the application metadata there during import, and the imported objects become local to that `PDB`.

The important operational point is:

- connect to the actual target `PDB`, not `CDB$ROOT`

Oracle warns that Data Pump operations are not generally meant to be run from the root container. If you connect to the root or seed, Oracle responds with `ORA-39357`, which is its polite way of asking why you are doing this to both of you.

---

## 3. Two Common Migration Paths

### File-based migration

This is the classic path:

- export metadata to a dump file
- copy the dump file to the target
- if using full transportable mode, also copy the user tablespace datafiles
- run `impdp` on the target

This is straightforward and usually easier to reason about.

### Network-link migration

This avoids writing an export dump file for the metadata movement. Instead:

1. create a database link in the target database
2. run `impdp` from the target
3. pull metadata over the link from the source

Oracle documents this as a valid migration method, especially when moving to a different storage system.

The price of convenience is that:

- the network must be reliable
- the source and target connectivity must be correct
- you still need to satisfy the privilege and transportable-datafile requirements

Because "we'll just do it over the network" is only comforting until the link drops halfway through a maintenance window.

---

## 4. Required Roles and Privileges

For full-database migration over a network link, Oracle documents these roles:

- `DATAPUMP_EXP_FULL_DATABASE` on the source
- `DATAPUMP_IMP_FULL_DATABASE` on the target

In multitenant environments, Oracle also documents that to administer the environment you need:

- `CDB_DBA`

Practical implication:

- the exporting account on the source needs the export role
- the importing account on the target needs the import role
- the administrator preparing `PDB`s and container-level plumbing needs the multitenant privileges to do it

If the privilege model is wrong, Data Pump does not reward effort points.

---

## 5. Preparing the Target `19c` CDB and Empty `PDB`

Before importing anything, you need:

- the `19c` software installed
- a `19c` container database already created
- an empty target `PDB`

You can create the target `PDB` with:

- `DBCA`
- or `CREATE PLUGGABLE DATABASE`

Example:

```sql
create pluggable database pdbimport
  admin user pdbadmin identified by "password"
  file_name_convert = ('pdbseed', 'pdbimport');

alter pluggable database pdbimport open;
```

What this gives you:

- the Oracle-supplied metadata copied from `PDB$SEED`
- a local administrator account for the new `PDB`
- a service that can be used to connect to the `PDB`

Listener registration happens when the `PDB` opens, but client-side aliases may still need attention. In real life, that means checking `tnsnames.ora`, application connect strings, and anything else that has been quietly hardcoded since 2014.

---

## 6. Full Transportable Export/Import (the fast path when user tablespaces can travel)

Oracle documents full transportable export/import as the efficient path for moving a database or moving data into a `CDB`/`PDB`.

The core idea:

- metadata moves through Data Pump
- user tablespace datafiles are transported instead of unloaded and reloaded
- objects in administrative tablespaces such as `SYSTEM` and `SYSAUX` still move conventionally

This is why full transportable migration is often much faster than a plain full export/import.

### Source-side requirements

Oracle documents these requirements:

- put all user-defined tablespaces into `READ ONLY`
- export with `FULL=Y`
- export with `TRANSPORTABLE=ALWAYS`

Example:

```bash
expdp system@pdbexport \
  full=y \
  transportable=always \
  directory=dp_mig \
  dumpfile=pdbexport_ft.dmp \
  logfile=pdbexport_ft.log
```

If the source is `11.2.0.3` or later but earlier than `12.1`, Oracle documents that you must also use:

- `VERSION=12`

### Import-side requirements

For file-based full transportable import into a `PDB`, Oracle documents that `impdp` only requires:

- `TRANSPORT_DATAFILES`

Data Pump import infers the full transportable nature of the operation.

Example:

```bash
impdp system@pdbimport \
  directory=dp_mig \
  dumpfile=pdbexport_ft.dmp \
  transport_datafiles='+DATA/ORCL/<pdb_guid>/DATAFILE/users01.dbf' \
  logfile=pdbimport_ft.log
```

For network-based full transportable import, Oracle documents that you must supply:

- `FULL=YES`
- `TRANSPORTABLE=ALWAYS`
- `TRANSPORT_DATAFILES=...`

Example:

```bash
impdp system@pdbimport \
  network_link=src_pdb \
  full=yes \
  transportable=always \
  transport_datafiles='+DATA/ORCL/<pdb_guid>/DATAFILE/users01.dbf' \
  logfile=pdbimport_net.log
```

---

## 7. Moving into a `PDB`

Oracle documents that using Data Pump with `PDB`s is broadly the same as using it with a non-`CDB`, except for multitenant-specific details.

Supported scenarios include:

- from a non-`CDB` into a `PDB`
- between `PDB`s in the same or different `CDB`s
- from a `PDB` into a non-`CDB`

The course demo follows the "move a source `PDB` into a target `19c` `PDB`" pattern:

1. open the source `PDB`
2. make user tablespaces read only
3. export with full transportable mode
4. copy the transported datafile(s) into the target storage
5. create and open the empty target `PDB`
6. import into that target `PDB`
7. validate that the expected tablespaces, datafiles, and row counts are present

This is a clean way to modernize the landing zone without forcing the original database through the full in-place upgrade carnival.

---

## 8. Alternative Path: Adopt a Non-CDB as a PDB

Data Pump is not the only way to bring older architecture into multitenant.

Oracle also documents the `DBMS_PDB.DESCRIBE` path:

- describe the non-`CDB` into an XML file
- plug it into a `CDB`
- run `noncdb_to_pdb.sql`

That is a different workflow from `Data Pump`, but it matters because it is another official route for moving a non-`CDB` into a `PDB`.

So when you are planning a migration, you should know there are at least two broad official answers:

- `Data Pump`
- plug-in / `DBMS_PDB`

Choose the one that fits the source version, compatibility situation, storage plan, and outage appetite.

---

## 9. Operational Gotchas That Matter

Some of the transcript's practical warnings are absolutely worth keeping:

- user tablespaces must be consistent before export
- if the source keeps changing after export, your imported target is already stale
- if you use full transportable migration, you must move the right datafiles to the right target location
- in `ASM` environments, target file placement matters
- after import, read the log and expect some objects to complain if the application was already skating on old or unsupported behavior

Also note Oracle's compatibility limits:

- source and target character sets must be compatible
- transportable operations have compatibility rules
- older sources may need intermediate handling or parameter changes

Migration is not magical. It is just much cleaner than direct upgrade when the circumstances cooperate.

---

## 10. Post-Migration Validation

After import, validate at least the following:

- expected `PDB` is open and usable
- required tablespaces exist
- transported datafiles are attached where expected
- key schemas and tables exist
- representative row counts match the source
- invalid objects and import warnings are reviewed
- application connectivity works through the correct service

A simple validation pattern is:

```sql
show pdbs

select tablespace_name, status
from dba_tablespaces
order by tablespace_name;

select file_name
from dba_data_files
where tablespace_name = 'USERS';

select count(*)
from hr.employees;
```

Do not stop at "import completed." That phrase has launched many unnecessary meetings.

---

## 11. Wrap-Up

`Data Pump` migration is the "build a clean target and move the payload" approach to getting into `19c`. It is especially useful when moving into multitenant, changing storage, or avoiding a messier direct upgrade path. The big ideas are simple: prepare the target `PDB`, export cleanly from the source, move what needs to be moved, import into the target, and then validate like you do not trust computers to grade their own homework.
