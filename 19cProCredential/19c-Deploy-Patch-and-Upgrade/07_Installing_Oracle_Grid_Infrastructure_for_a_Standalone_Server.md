## Lesson 7 - Installing Oracle Grid Infrastructure for a Standalone Server and Managing Oracle Restart (where high availability moves in before the database does)

Oracle Grid Infrastructure for a standalone server installs two things the course cares deeply about: **Oracle Restart** for local high availability and **Oracle ASM** for storage management. They live together in the Grid Infrastructure home, they should be installed before the database software, and if you ignore that order Oracle will make you do the registration work manually later like a petty but technically correct bureaucrat.

By the end of this lesson, you should be able to:

- Explain what Oracle Grid Infrastructure for a standalone server installs
- Justify why Grid Infrastructure should be installed before the database software
- Explain what Oracle Restart does in a single-instance environment
- Identify the storage planning decisions required for `Oracle ASM`
- Compare ASM disk-group redundancy choices and supported storage sources
- Summarize the main 19c system requirements for a standalone Grid Infrastructure installation
- Describe the role of `Oracle ASM Configuration Assistant (ASMCA)` in creating a recovery disk group
- Describe how Oracle Restart tracks and manages resources
- Use `crsctl` and `srvctl` for Oracle Restart administration
- Stop and start Oracle Restart and managed homes in the correct order

---

## 1. What This Installation Actually Gives You

Oracle Grid Infrastructure is the software bundle that provides:

- **Oracle Restart** for standalone servers
- **Oracle Clusterware** in clustered environments
- **Oracle ASM** for storage management

For this lesson, the focus is **Oracle Grid Infrastructure for a standalone server**, which means:

- It is installed on the same server as the database software
- It provides local high-availability management through Oracle Restart
- It provides the ASM binaries used for Oracle-managed storage

Both the high-availability components and the `ASM` binaries are installed into a single **Oracle Grid Infrastructure home**.

This is convenient, but it also means mistakes in the Grid Infrastructure install do not stay politely contained. They follow you into storage, restart management, and later database registration.

---

## 2. Why You Install Grid Infrastructure First

If you plan to use Oracle Grid Infrastructure for a standalone server, install it **before** installing the Oracle Database software and creating the database.

Why:

- Oracle Restart is then in place before the database exists
- ASM storage is ready before database files need to live somewhere
- The database can be registered correctly as part of the expected flow

If you install the database first and Grid Infrastructure second:

- You can still make the environment work
- But you will need to manually register the database with Oracle Restart

In other words, Oracle lets you do it out of order in the same way a tax agency lets you fix paperwork after a mistake: technically possible, spiritually avoidable.

---

## 3. ASM Storage Planning (because "we'll figure out the disks later" is not a plan)

Before using `Oracle ASM`, determine the storage requirements for the disk groups you plan to create.

That means answering questions such as:

- How many devices are required?
- How much free space is needed?
- Will `ASM` store database files, recovery files, or both?
- How many disk groups are needed?

A common design is:

- One disk group for database files
- One disk group for recovery files

That separation is operationally useful because it keeps the recovery area from competing directly with primary data storage for the same space and failure blast radius.

The course specifically calls out creating an ASM disk group for **recovery** using `ASMCA`, which fits neatly with that two-disk-group strategy.

---

## 4. ASM Redundancy Choices (pick how much failure you want to survive)

For each ASM disk group, choose a redundancy level.

The transcript focuses on these options:

- **Normal redundancy**
  - Two-way mirroring

- **High redundancy**
  - Three-way mirroring

- **Flex redundancy**
  - More flexible redundancy behavior with flex disk groups

These choices determine:

- How Oracle ASM mirrors files
- How many disks are required
- How much usable capacity remains after protection overhead

And yes, this is where storage planning becomes arithmetic with consequences.

Practical note:

- If the storage layer already provides mirroring, Oracle environments may also use **external redundancy**

The course emphasis is on normal, high, and flex because that is where `ASM` itself is doing the protection work rather than politely trusting storage underneath it.

---

## 5. Storage Sources for ASM Disk Groups

An ASM disk group can be created from several storage resource types, including:

- Disk partitions
- Logical Unit Numbers (`LUNs`)
- Logical volumes
- `NFS` files
- On Exadata systems, grid disks

The course also notes an important recommendation:

- Oracle does **not** recommend using logical volume managers for mirroring when `ASM` is already providing mirroring

That is sensible because double-stacking mirroring technologies is a classic way to spend extra capacity and complexity without becoming nearly as clever as you think.

For `NFS`-backed ASM storage:

- Disk-group files can come from multiple `NFS` servers
- This can improve load balancing
- It can also support more flexible capacity planning

So yes, `ASM` can live on network storage too. Oracle has never met a storage topology it could not turn into homework.

---

## 6. Disk Path Persistence (the reboot should not rename your future)

ASM requires stable disk ownership, permissions, and naming.

The course lists three approaches:

- `ASMLib`
- `ASM Filter Driver`
- Rule files such as custom `udev` rules

This class focuses on **`udev` rules**, which is a good practical choice because they provide:

- Persistent device naming
- Persistent ownership
- Persistent permissions across reboot

Current Oracle documentation also treats `ASM Filter Driver` as **deprecated beginning with Oracle Database 19c**, which makes the course's `udev` emphasis look much less like a coincidence and much more like good judgment.

If you skip this planning and rely on whatever names Linux feels like producing during startup, the next reboot can leave your ASM disks looking like strangers at a bad family reunion.

---

## 7. Standalone Grid Infrastructure System Requirements

Oracle Grid Infrastructure for a standalone server has minimum system requirements that are very similar between `12.2` and `19c`, with a few important differences.

### Memory and swap

Minimum physical memory:

- **8 GB RAM**

Swap guidance:

- If physical RAM is between **8 GB and 16 GB**, configure swap equal to RAM
- If physical RAM is **greater than 16 GB**, configure **16 GB** of swap

This matches the familiar Oracle philosophy of "minimum requirements" that still somehow feel like the system is being judged personally.

### Grid home disk space

Approximate software space requirement:

- `12.2`: about **8.6 GB**
- `19c`: about **12 GB**

So yes, the software got larger. Shocking. Apparently Oracle did not spend the last several releases on aggressive minimalist sculpture.

### Temporary space

Minimum temporary directory requirement:

- **1 GB** in the temp directory

If `/tmp` is too small:

- Remove unnecessary files
- Or set `TMP` / `TMPDIR` to another directory

### Operating system requirements

For supported operating systems, kernel versions, and prerequisite packages, review the Linux installation guides.

The 19c preinstallation package is:

- `oracle-database-preinstall-19c`

Display requirements are effectively unchanged from `12.2` for the graphical installer.

---

## 8. High-Level Installation Flow

At a high level, the standalone Grid Infrastructure installation flow is:

1. Prepare the operating system and users
2. Prepare persistent ASM storage device naming
3. Launch the Grid Infrastructure installer
4. Select the standalone-server configuration path
5. Configure the initial ASM disk group
6. Map the privileged operating system groups
7. Confirm install and inventory locations
8. Decide how root scripts will run
9. Complete prerequisite checks and install
10. Create any additional ASM disk groups such as the recovery disk group
11. Only then proceed to Oracle Database software installation

This ordering matters because the database wants a stable home, stable storage, and a restart framework that already exists instead of one that shows up halfway through and asks to be introduced retroactively.

---

## 9. OUI Configuration Option Page (pick the right branch of the family tree)

When Oracle Universal Installer (`OUI`) starts, the first major decision is the **configuration option**.

The installer presents options such as:

- Configure Oracle Grid Infrastructure for a new cluster
- Configure Oracle Grid Infrastructure for a standalone server (`Oracle Restart`)
- Upgrade an existing Oracle Grid Infrastructure installation
- Install software only without configuration

For this course, the correct choice is:

- **Configure Oracle Grid Infrastructure for a standalone server**

The software-only option exists for people who want to configure everything manually later, which is absolutely an option if your hobbies include unnecessary self-inflicted complexity.

---

## 10. Create ASM Disk Group Page (the part where storage stops being abstract)

One of the next major pages is the ASM disk-group creation page.

Here you specify:

- The ASM disk-group name
- The redundancy level
- The allocation unit (`AU`) size
- The candidate disks to include

### Disk group name and redundancy

Oracle commonly defaults the disk-group name to `DATA`, though you can change it.

Redundancy choices include:

- **External**
  - Use this when redundancy is handled by the storage layer

- **Normal**
  - Two-way mirroring

- **High**
  - Three-way mirroring

- **Flex**
  - Flexible redundancy behavior with flex disk groups

For many installations, **normal redundancy** is the practical default when `ASM` itself is expected to protect the data.

Important warning:

- You cannot change the redundancy level after the disk group is created

So if you choose badly, the disk group will spend its entire life reminding you.

### Allocation unit size

The allocation unit size can be chosen from:

- `1 MB`
- `2 MB`
- `4 MB`
- `8 MB`
- `16 MB`
- `32 MB`
- `64 MB`

One transcript detail needed cleanup here:

- The default is **not** universally `4 MB`
- Oracle documentation for this flow shows **`1 MB`** as the default in the standard disk-group screen
- Flex disk groups commonly use `4 MB` as the default AU size

And just like redundancy:

- You cannot change the AU size after creating the disk group

This is why "we'll tune it later" is not a real plan at this stage.

### Selecting disks and discovery path

If the expected ASM disks do not appear, click **Change Discovery Path** and provide the appropriate search string for the device location.

The practical rules are:

- The disks must be visible to the installer
- The disks must be owned correctly for the install user
- The discovery path must match where the devices actually live

Optional note:

- `OUI` may offer **Configure Oracle ASM Filter Driver**
- For 19c-era study purposes, remember that `ASMFD` is already deprecated, so treat that checkbox as legacy-adjacent rather than aspirational

---

## 11. ASM Password Page (where `SYSASM` gets a password and Oracle judges your punctuation)

The **Specify ASM Password** page sets passwords for the administrative ASM accounts, including:

- `SYS`
- `ASMSNMP`

You can choose either:

- Different passwords for each account
- The same password for all ASM administrative users

Oracle generally recommends separate passwords, because reducing privilege overlap is good practice and also makes security people slightly less theatrical.

From `12.2` onward, password validation is stricter, so the value you enter must satisfy Oracle's complexity rules for the installer.

---

## 12. Management Options Page (Enterprise Manager, if your environment actually uses it)

The **Management Options** page lets you choose whether to register the installation with:

- **Enterprise Manager Cloud Control**

If your environment uses Enterprise Manager, provide the required management details there.

If not:

- Leave it unregistered
- Continue to the next page

This is a normal enterprise integration choice, not a moral referendum on whether you are a "serious" administrator.

---

## 13. Privileged Operating System Groups Page

The **Privileged Operating System Groups** page maps OS groups to Oracle privilege groups for ASM administration.

In a job-role-separation environment, the normal mappings are:

- `asmadmin` -> `OSASM`
- `asmdba` -> `OSDBA for ASM`
- `asmoper` -> `OSOPER for ASM`

If the environment uses fewer groups, Oracle can map multiple privilege classes to the same OS group. But in this course's role-separated setup, the point is to keep them distinct so responsibilities are clearer and mistakes are more traceable.

This is where all the earlier prep work with users and groups stops being theoretical and starts becoming installer dropdowns with consequences.

---

## 14. Specify Install Location Page

The **Specify Install Location** page is where you confirm:

- The Oracle base location
- The Grid Infrastructure software location

Check these paths carefully:

- They must be valid
- They must match your intended ownership model
- They must not be casual improvisations created by someone who was getting tired

If the defaults are wrong, use **Browse** and correct them before continuing.

---

## 15. Inventory Page

If this is the first Oracle installation on the host, `OUI` displays the inventory page.

Here you confirm:

- The inventory directory path, commonly something like `/u01/app/oraInventory`
- The inventory group, typically `oinstall`

The important operational detail:

- The inventory directory must be writable by the installation user

In the course's role-separated pattern, that means the Grid Infrastructure install owner must be able to write there. A perfect inventory path with the wrong ownership is still a broken setup, just with better typography.

---

## 16. Root Script Execution Configuration Page

The **Root Script Execution Configuration** page is where you decide how mandatory privileged scripts will run.

Your choices are:

- Run them manually by leaving automatic execution disabled
- Run them automatically using the `root` password
- Run them automatically using a `sudo`-enabled account

If you choose automatic execution:

- `OUI` still prompts for confirmation during the install
- You must supply the required credentials in advance

If you choose manual execution:

- `OUI` stops at the appropriate point
- You run the scripts yourself as `root`
- Then return to the installer and continue

This is not just a convenience setting. It is a security-model decision disguised as a wizard page.

---

## 17. Prerequisite Checks Page

The **Prerequisite Checks** page validates whether the system meets the minimum installation and configuration requirements.

Usually, this page requires no new input unless:

- A check fails
- A warning appears that requires manual review

If something fails:

- Fix the problem
- Rerun the check
- Do not try to out-argue the installer unless you enjoy proving it right later

---

## 18. Summary and Installation Progress

On the **Summary** page:

- Review all selections carefully
- Use **Edit** if any choice is wrong
- Optionally use **Save Response File** if you want to reuse the same configuration in another environment

When you proceed:

- `OUI` begins the actual installation
- If automatic root-script execution was chosen, a popup appears asking you to confirm that the root scripts should be executed
- You accept that prompt and let the install continue

When the installation completes successfully, the final page reports that Oracle Grid Infrastructure for a standalone server has been configured successfully.

This is the brief and beautiful phase where the wizard finally stops asking questions and starts producing software instead of paperwork.

---

## 19. Creating the Recovery Disk Group with ASMCA

After Grid Infrastructure is installed, you can use **Oracle ASM Configuration Assistant (`ASMCA`)** to create an ASM disk group for recovery files.

That recovery disk group typically holds items such as:

- Fast Recovery Area contents
- Backup-related files
- Recovery-related files

Using `ASMCA` for this step gives you a guided path to:

- Select the candidate disks
- Choose the disk-group name
- Set the redundancy level
- Choose the allocation unit size
- Set any additional disk-group attributes
- Finish the ASM disk-group configuration

When you open `ASMCA` and go to the **Disk Groups** view:

- Existing disk groups, such as `DATA`, are listed first
- You click **Create** to define a new disk group

Typical inputs on the create screen include:

- **Disk Group Name**
  - For example, `FRAASM`

- **Redundancy**
  - `External`
  - `Normal`
  - `High`
  - `Flex`

- **Allocation Unit Size**
  - Common values range from `1 MB` through `64 MB`

- **Candidate Disks**
  - Select the devices that will belong to the new disk group

- **Additional Attributes**
  - `COMPATIBLE.ASM`
  - `COMPATIBLE.RDBMS`
  - Sector-size settings where relevant

One subtle but important point:

- `4 MB` is a very common AU choice and frequently what course material recommends
- But the real default can vary depending on the disk-group creation path and type, so verify what `ASMCA` is actually showing rather than worshipping slide screenshots

And just like the disk group created in `OUI`:

- Redundancy is not something you casually change later
- AU size is not something you casually change later either

A common pattern is:

- One ASM disk group for primary database files
- One ASM disk group for recovery files

Because when a recovery area fills up or has a storage issue, it is much nicer if that problem is not sitting in exactly the same place as everything else you were hoping to recover.

---

## 20. Oracle Restart Overview (because single-instance databases still enjoy not falling over)

Oracle Restart is Oracle's local high-availability framework for:

- **Single-instance**
- **Non-clustered**
- **Standalone server** environments

It is not the clustered answer. In `RAC` environments, that role belongs to **Oracle Clusterware**.

Oracle Restart is designed to:

- Start components in the correct dependency order
- Stop components cleanly in the correct dependency order
- Restart failed components after hardware or software failures
- Keep track of the resources it manages and how they are configured

In practice, it is the thing standing between "one server had a hiccup" and "why is nothing back online yet?"

Oracle Restart runs out of the **Grid Infrastructure home**, which is why the standalone Grid install matters even when you are not building a cluster.

---

## 21. What Oracle Restart Can Manage

Oracle Restart can manage resources such as:

- Databases
- Database instances
- Listeners
- Database services
- Oracle `ASM`
- `ASM` disk groups
- `ONS` where it is configured

The point is not just to maintain a list of objects. The point is to understand the dependencies between them.

For example:

- A database may depend on `ASM` disk groups
- Services depend on the database
- The listener should be available before clients try to connect

That dependency ordering is why Oracle recommends using the management tools instead of manually starting random pieces and hoping the stack interprets that as leadership.

---

## 22. Oracle Restart Internals at a Useful Level (not the daemon zoo, just the guided tour)

At the center of Oracle Restart is:

- `ohasd`
  - The Oracle High Availability Services daemon

That local stack also uses supporting agents and daemons, including items such as:

- `oraagent.bin`
- `orarootagent.bin`
- `cssdagent`
- `diskmon`

At a practical level, what matters is:

- `ohasd` brings up the local high-availability stack
- The agents manage resources under the correct OS ownership
- Root-owned resources and user-owned resources are not managed the same way

If you stop Oracle Restart, the daemons stop. If Oracle Restart is enabled, it comes back with the host and resumes its local babysitting duties.

You do not need to memorize every process name to administer the environment, but you do need to know Oracle Restart is not a single binary with a motivational poster. It is a small stack.

---

## 23. Oracle Restart Configuration (the official list of things Oracle promises to care about)

Oracle Restart maintains a configuration repository of managed components and their settings.

That includes:

- Which resources exist
- Which home they belong to
- Startup and stop behavior
- Dependency information
- Resource-specific configuration such as `SPFILE` paths and service definitions

Some resources are added automatically when you use Oracle's tools.

Typical examples:

- Databases created with `DBCA`
- Disk groups created with `ASMCA`
- Listeners created with `NETCA`
- Services created with `srvctl`

Other things are **not** added or removed automatically when you improvise with SQL or OS commands.

Typical examples:

- Databases created manually with SQL
- Services created only with `DBMS_SERVICE` or `service_names`
- Files deleted at the OS level instead of being removed with Oracle tools

That is why the sane rule is:

- Use Oracle utilities when you want Oracle Restart to stay synchronized

If you build or destroy objects behind its back, Oracle Restart does not become wiser through intuition.

---

## 24. `crsctl` vs `srvctl` (choose the right wrench before hitting the machinery)

Oracle gives you two different command-line utilities here, and they are not interchangeable.

### Use `crsctl` for Oracle Restart itself

`crsctl` is used for the local high-availability stack, such as:

- Checking whether Oracle Restart is running
- Displaying whether auto-start is enabled
- Enabling or disabling Oracle Restart
- Stopping or starting Oracle Restart itself

Common examples:

```bash
crsctl check has
crsctl config has
crsctl disable has
crsctl enable has
crsctl stop has
crsctl start has
```

### Use `srvctl` for the resources Oracle Restart manages

`srvctl` is used for:

- Databases
- Database services
- Listeners
- `ASM`
- Disk groups
- `ONS`
- Entire Oracle homes managed by Oracle Restart

This split matters. `crsctl` deals with the framework. `srvctl` deals with the things living inside the framework.

---

## 25. Running `srvctl` from the Correct Home and Correct User

One of the easiest ways to waste time is to run `srvctl` from the wrong home or as the wrong user and then act surprised when Oracle objects.

General rule:

- Manage **database** resources from the **database home**
- Manage **Grid / ASM** resources from the **Grid Infrastructure home**

Practical mapping:

- Database and database services
  - Run `srvctl` from the **database home**
  - Usually as the **database software owner**

- `ASM`, disk groups, listeners, and `ONS`
  - Run `srvctl` from the **Grid Infrastructure home**
  - Usually as the **Grid Infrastructure owner**

Example environment setup for Grid-side work:

```bash
export ORACLE_HOME=/u01/app/12.2.0/grid
export PATH=$ORACLE_HOME/bin:$PATH
```

Example environment setup for database-side work:

```bash
export ORACLE_HOME=/u01/app/oracle/product/12.2.0/dbhome_1
export PATH=$ORACLE_HOME/bin:$PATH
```

Oracle does provide help output:

```bash
srvctl -help
srvctl status database -help
```

Which is useful because the command syntax is extensive and Oracle has never met an option list it could not make longer.

---

## 26. Common `srvctl` Tasks

The basic `srvctl` command shape is:

```bash
srvctl <command> <object> [options]
```

Common actions include:

- `add`
- `remove`
- `config`
- `modify`
- `status`
- `start`
- `stop`

### Configuration and status examples

Check database configuration:

```bash
srvctl config database -db orcl
```

Check detailed database configuration:

```bash
srvctl config database -db orcl -all
```

Check database status:

```bash
srvctl status database -db orcl
```

Check listener status:

```bash
srvctl status listener -listener LISTENER
```

### Start and stop examples

Start a database:

```bash
srvctl start database -db orcl
```

Start a database in mount mode:

```bash
srvctl start database -db orcl -startoption mount
```

Stop a database cleanly:

```bash
srvctl stop database -db orcl -stopoption immediate
```

### Home-level maintenance examples

Stop everything managed from a specific home and save the state:

```bash
srvctl stop home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome.state
```

Start everything in that home back to the saved state:

```bash
srvctl start home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome.state
```

That `statefile` capability is especially useful during maintenance because it records what was up, what was down, and what should be restored afterward, which is much better than relying on your memory after midnight patching.

### Manual registration examples

If a resource was created outside the tools that auto-register it, you may need to add or modify it manually.

For example, updating the `SPFILE` location for a database:

```bash
srvctl modify database -db orcl -spfile +DATA/ORCL/spfileorcl.ora
```

The broader lesson is simple:

- If you create or change resources outside the assistants, Oracle Restart may need manual help

---

## 27. Stopping and Starting Oracle Restart for Maintenance

Sometimes you need to stop only a database home. Sometimes you need to stop Grid Infrastructure resources. And sometimes, especially for Grid Infrastructure patching, you need to stop Oracle Restart itself.

### If you are only maintaining the database home

Usually:

1. Stop the database home resources
2. Leave Oracle Restart itself running
3. Perform the database-home maintenance
4. Start the database home resources again

Example:

```bash
srvctl stop home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome.state
srvctl start home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome.state
```

### If you are maintaining Grid Infrastructure

Then the sequencing matters more:

1. Stop database-home resources first
2. Stop Grid-home resources second
3. Disable Oracle Restart
4. Stop Oracle Restart
5. Perform the maintenance
6. Enable Oracle Restart
7. Start Oracle Restart
8. Start Grid-home resources
9. Start database-home resources

Typical commands:

```bash
srvctl stop home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome.state
srvctl stop home -oraclehome /u01/app/12.2.0/grid -statefile /tmp/gridhome.state
crsctl disable has
crsctl stop has
```

After maintenance:

```bash
crsctl enable has
crsctl start has
srvctl start home -oraclehome /u01/app/12.2.0/grid -statefile /tmp/gridhome.state
srvctl start home -oraclehome /u01/app/oracle/product/12.2.0/dbhome_1 -statefile /tmp/dbhome.state
```

Start Grid components before database components because the database may depend on:

- `ASM`
- Disk groups
- Listener resources managed from the Grid home

Which is Oracle's polite way of telling you not to start the roof before the walls.

---

## 28. Practical Takeaways

Before installing Oracle Grid Infrastructure for a standalone server, confirm:

- Grid Infrastructure is being installed before the database software
- The host meets the `19c` memory, swap, temp, and disk requirements
- ASM storage has been sized for database files, recovery files, or both
- A redundancy model has been selected for each disk group
- You know the OS group mappings for `OSASM`, `OSDBA for ASM`, and `OSOPER for ASM`
- Disk device persistence is configured, preferably with `udev` rules for this class
- You know whether `ASMCA` will create a separate recovery disk group after the install
- You know which resources Oracle Restart will manage
- You know when to use `crsctl` versus `srvctl`
- You know which home and which OS user should run the `srvctl` command you are about to type
- You know that Grid-home maintenance may require disabling and stopping Oracle Restart itself

If these answers are vague, then the install may still succeed, but the resulting environment will be held together by documentation debt and crossed fingers.

---

## 29. Wrap-Up (Grid Infrastructure first, regrets later if not)

This lesson covered Oracle Grid Infrastructure for a standalone server: what it installs, why it must come before the database software, how to plan ASM storage, what system requirements matter in `19c`, and how the `OUI` pages guide the install from configuration choice through root-script handling and completion. It also covered how `ASMCA` fits into creating a recovery disk group, what Oracle Restart manages, how `crsctl` and `srvctl` divide responsibilities, and how to stop and start the stack cleanly for maintenance. Next comes the database software installation itself, where all this preparation finally gets to stop being theoretical and start consuming actual disk space with intent.
