## Lesson 7 - Installing Oracle Grid Infrastructure for a Standalone Server (where high availability moves in before the database does)

Oracle Grid Infrastructure for a standalone server installs two things the course cares deeply about: **Oracle Restart** for local high availability and **Oracle ASM** for storage management. They live together in the Grid Infrastructure home, they should be installed before the database software, and if you ignore that order Oracle will make you do the registration work manually later like a petty but technically correct bureaucrat.

By the end of this lesson, you should be able to:

- Explain what Oracle Grid Infrastructure for a standalone server installs
- Justify why Grid Infrastructure should be installed before the database software
- Identify the storage planning decisions required for `Oracle ASM`
- Compare ASM disk-group redundancy choices and supported storage sources
- Summarize the main 19c system requirements for a standalone Grid Infrastructure installation
- Describe the role of `Oracle ASM Configuration Assistant (ASMCA)` in creating a recovery disk group

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
4. Install the Grid Infrastructure home
5. Configure Oracle Restart and Oracle ASM
6. Create the required ASM disk groups
7. Only then proceed to Oracle Database software installation

This ordering matters because the database wants a stable home, stable storage, and a restart framework that already exists instead of one that shows up halfway through and asks to be introduced retroactively.

---

## 9. Creating the Recovery Disk Group with ASMCA

After Grid Infrastructure is installed, you can use **Oracle ASM Configuration Assistant (`ASMCA`)** to create an ASM disk group for recovery files.

That recovery disk group typically holds items such as:

- Fast Recovery Area contents
- Backup-related files
- Recovery-related files

Using `ASMCA` for this step gives you a guided path to:

- Select the candidate disks
- Choose the disk-group name
- Set the redundancy level
- Finish the ASM disk-group configuration

A common pattern is:

- One ASM disk group for primary database files
- One ASM disk group for recovery files

Because when a recovery area fills up or has a storage issue, it is much nicer if that problem is not sitting in exactly the same place as everything else you were hoping to recover.

---

## 10. Practical Takeaways

Before installing Oracle Grid Infrastructure for a standalone server, confirm:

- Grid Infrastructure is being installed before the database software
- The host meets the `19c` memory, swap, temp, and disk requirements
- ASM storage has been sized for database files, recovery files, or both
- A redundancy model has been selected for each disk group
- Disk device persistence is configured, preferably with `udev` rules for this class
- You know whether `ASMCA` will create a separate recovery disk group after the install

If these answers are vague, then the install may still succeed, but the resulting environment will be held together by documentation debt and crossed fingers.

---

## 11. Wrap-Up (Grid Infrastructure first, regrets later if not)

This lesson covered Oracle Grid Infrastructure for a standalone server: what it installs, why it must come before the database software, how to plan ASM storage, what system requirements matter in `19c`, and how `ASMCA` fits into creating a recovery disk group. Next comes the database software installation itself, where all this preparation finally gets to stop being theoretical and start consuming actual disk space with intent.
