## Lesson 9 - Creating the Oracle Database (where the binaries finally get a personality)

Installing the software home is only half the story. The real moment of truth is creating the actual database, which is where storage decisions, memory settings, listener choices, recovery layout, and pluggable-database design stop being abstract architecture talk and start becoming a thing users can break in production.

By the end of this lesson, you should be able to:

- Explain the planning work that should happen before creating an Oracle database
- Identify the major Database Configuration Assistant (`DBCA`) operation modes
- Walk through the main `DBCA` pages for creating a container database
- Distinguish GUI-driven database creation from silent-mode database creation
- Recognize how templates, pluggable databases, and recovery settings fit into the creation workflow

---

## 1. Plan the Database Before You Click Anything

Before creating a database, think through the logical and physical design.

Important planning questions include:

- How many disks and disk devices are available?
- What storage type is being used?
- How many data files should each tablespace use?
- What kind of data will live in each tablespace?
- Where should recovery files be stored?
- What backup strategy will the design need to support?

This matters because the logical storage design affects:

- Performance
- Backup efficiency
- Recovery behavior
- Ongoing management effort

In distributed or busy environments, the physical placement of frequently accessed data can have a direct effect on application performance. This is Oracle's polite way of saying storage layout can absolutely ruin your day if you treat it like clerical work.

---

## 2. What DBCA Can Actually Do

When you start **Database Configuration Assistant (`DBCA`)**, the first page is the **database operation selection** page.

Common choices include:

- **Create a database**
- Configure or manage an existing database
- Delete a database
- Manage templates
- Manage pluggable databases
- Oracle RAC database instance management

For this lesson, the focus is the default and most important choice:

- **Create a database**

But `DBCA` is more than just a one-time database-creation wizard. It is also Oracle's way of centralizing several "please don't do this blind by hand" operations into one tool.

---

## 3. Typical vs Advanced Configuration

After choosing to create a database, `DBCA` asks how much control you want.

The two main modes are:

- **Typical configuration**
  - Minimal input
  - Faster setup
  - Good if you want Oracle to make more decisions for you

- **Advanced configuration**
  - More pages
  - More control
  - Better when you want to customize the database layout and options

In this course, the emphasis is on:

- **Advanced configuration**

Because that is the mode where you can actually see what Oracle is doing instead of just receiving a database as if it emerged from fog.

---

## 4. Deployment Type and Template Selection

Next, `DBCA` asks about the deployment type.

In a standalone environment, the normal choice is:

- **Oracle single-instance database**

In clustered environments, other options can appear, such as:

- **Oracle RAC**
- **Oracle RAC One Node**

After that, you choose a template.

Common template choices include:

- **General Purpose / Transaction Processing**
- **Data Warehouse**
- Custom templates

The default is typically:

- **General Purpose / Transaction Processing**

This template determines a starting set of defaults. It is not magic. It is just Oracle's opinionated starter kit.

---

## 5. Database Identification and Multitenant Choices

The database identification page asks for:

- The **Global Database Name**
- The **SID**

In the course practice flow, example values include:

- Global database name: `orclus.oracle.com`
- SID: `ORCL`

`DBCA` also lets you choose whether to create:

- A **container database (`CDB`)**
- Zero, one, or more **pluggable databases (`PDBs`)**

You can also enable:

- **Local undo** for PDBs

This is one of the most important modern design choices in Oracle. If you are building multitenant, say so up front. Do not create a whole database and then act surprised that the containers matter.

---

## 6. Storage and Recovery Configuration

`DBCA` then moves into database storage options.

You can choose to:

- Use template-based storage attributes
- Or specify the storage attributes directly

Storage can use:

- File system
- `Oracle ASM`

In the practice environment, the database files are placed in:

- `+DATA`

The **Fast Recovery Area (`FRA`)** is configured separately.

Typical `FRA` choices include:

- Enable **Specify Fast Recovery Area**
- Set the storage type
- Set the `FRA` location
- Set the `FRA` size
- Enable or disable archiving

In the practice flow:

- `FRA` location: `+FRA`
- `FRA` size: `40 GB`

The course intentionally raises the `FRA` size because the default was not enough for the later labs. Which is one of the more honest moments in Oracle training.

---

## 7. Listener and Security Options

The network configuration page shows the listeners available for the new database.

In the standalone Grid Infrastructure path used here:

- The default listener is already running from the **Grid home**
- It commonly listens on port `1521`

If no listener exists, you can choose:

- **Create a new listener**

`DBCA` may also expose optional security features such as:

- **Oracle Database Vault**
- **Oracle Label Security**

If you do not need them, leave them disabled. Security features are excellent when intentionally designed and somewhat less charming when enabled accidentally during a wizard binge.

---

## 8. Configuration Options: Memory, Character Set, Connection Mode

The configuration options pages let you define memory and instance behavior.

Common memory choices include:

- **Automatic Shared Memory Management (`ASMM`)**
- **Manual Shared Memory Management**
- **Automatic Memory Management (`AMM`)**

The course notes that:

- `ASMM` is often the default
- Manual sizing lets you control things such as `shared pool`, `buffer cache`, `large pool`, `java pool`, and `PGA`

Other options include:

- Block size
- Character set
- Dedicated server vs shared server connection mode
- Sample schema installation

In the practice flow:

- Character set is set to **Unicode**
- Sample schemas are installed

Installing the sample schemas is useful in labs, demos, and documentation examples. In production, people usually prefer that their HR schema not exist as a gift from the installer.

---

## 9. Management Options and Administrative Passwords

For management, `DBCA` can configure:

- **Enterprise Manager Database Express**
- **Enterprise Manager Cloud Control**

In the course example, the database is configured for:

- **Enterprise Manager Database Express**

Administrative passwords must also be set for users such as:

- `SYS`
- `SYSTEM`
- `PDBADMIN`

You can choose:

- A different password for each account
- The same password for all administrative users

In the practice environment, the shared administrative password used is:

- `cloud_4u`

Which is acceptable for a classroom and slightly horrifying anywhere with customers.

---

## 10. Database Creation Options and Summary

Near the end of the wizard, `DBCA` asks what output you want from the work you just defined.

Common choices include:

- **Create database**
- **Save as database template**
- **Generate database creation scripts**

These options are useful because they let you:

- Create the database immediately
- Reuse the design later
- Capture the build logic in script form for repeatability

Then the summary page appears.

This is your last chance to notice that the database is about to be created in the wrong disk group with the wrong memory size and a name that would embarrass you in front of other adults.

---

## 11. Database Creation Progress and Completion

Once you click **Finish**, `DBCA` shows the **Database Creation Progress** page.

This page displays:

- Overall progress
- Step-by-step status
- Optional access to activity and alert logs

When creation completes, the final page offers:

- **Password Management**

If needed, you can change user passwords there before closing the tool.

Then the database exists, the listener knows about it, and your calm weekend has officially become conditional.

---

## 12. Silent Mode DBCA (for people who prefer shells to wizard pages)

`DBCA` can also run in **silent mode** from the command line.

This is useful when you want:

- Repeatable database creation
- Less GUI dependence
- Easier automation
- Scripted rebuilds for labs or standard environments

In the course practice flow, a shell script such as:

- `create_CDB122.sh`

is reviewed and then executed. The script:

- Deletes a target database if it already exists
- Invokes `DBCA` in silent mode
- Creates a replacement database for later upgrade practice

This is the adult version of database creation: fewer buttons, more consequences, and much better reproducibility.

---

## 13. Practical Takeaways

Before creating the database, confirm:

- The database software is already installed
- The listener is available
- `ASM` disk groups such as `+DATA` and `+FRA` exist if you plan to use them
- You know whether you want a `CDB` and how many `PDBs`
- You have decided on recovery area size and archiving behavior
- You know whether you want to use `DBCA` interactively or in silent mode

If these choices are fuzzy, the database can still be created. It just may not be the database you meant to create, which is a very Oracle-shaped problem.

---

## 14. Wrap-Up (the database is finally real)

This lesson covered how `DBCA` is used to create an Oracle database, choose templates, define storage and recovery settings, configure multitenant options, and manage database creation through either the GUI or silent mode. Next, the course moves from creation into patching and upgrading, where the thing you just built begins its long career of being maintained under pressure.
