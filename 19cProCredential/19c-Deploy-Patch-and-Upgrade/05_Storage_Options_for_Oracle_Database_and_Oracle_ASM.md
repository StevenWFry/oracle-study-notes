## Lesson 5 - Storage Options for Oracle Database and Oracle ASM (where "just put it on disk" becomes an expensive last statement)

Storage is one of the most important installation decisions because Oracle will happily use whatever you give it, including bad layouts that perform poorly, fail awkwardly, and make future recovery conversations much more dramatic than necessary. This lesson covers the two major storage paths: traditional file systems and `Oracle ASM`, plus the related question of how ASM disks keep sane names and permissions across reboots.

By the end of this lesson, you should be able to:

- Compare file-system storage and `Oracle ASM` as Oracle Database storage options
- Identify common file-system deployment patterns, including basic disks, `LVM`/`RAID`, and `NFS`
- Explain where `Oracle Direct NFS (dNFS)` fits into an `NFS` design
- Describe how `Oracle ASM` balances I/O and provides redundancy
- Distinguish `udev`, `ASMLib`, and `ASM Filter Driver` as approaches to ASM disk persistence

---

## 1. The Big Storage Choice (file system or ASM, pick your adventure carefully)

For Oracle Database storage, you generally choose between:

- **File-system storage**
- **`Oracle ASM` storage**

Both are valid, but they lead to very different administration models.

At a high level:

- File systems are familiar and broadly understood
- `Oracle ASM` is Oracle-managed storage designed specifically for Oracle workloads

The wrong storage decision does not usually fail immediately. It waits until performance tuning, scaling, patching, or recovery, then reveals it has been collecting interest the whole time.

---

## 2. File-System Storage Options (traditional, familiar, and fully capable of being misdesigned)

If you choose file-system storage for the database, you have several options.

Common choices include:

- File systems created directly on physical disks
- File systems created on top of `LVM`
- File systems created on top of `RAID` devices
- `NFS`-based file systems

Oracle supports this approach, but the layout matters.

### Basic disks and the OFA mindset

If you create the database on basic disks that are **not** part of logical volumes or `RAID`, Oracle recommends following **Optimal Flexible Architecture (`OFA`)** principles and spreading database files across multiple disks.

Why:

- Avoid a single point of failure
- Improve performance distribution
- Reduce the chance that one busy device becomes the universal bottleneck

Putting everything on one disk because it was "available" is how a storage design becomes a confession.

### `LVM` and `RAID`

If you use multiple disks combined through a logical volume manager or `RAID`, Oracle recommends the **Stripe And Mirror Everything (`SAME`)** methodology.

The idea is straightforward:

- Stripe broadly for performance
- Mirror for reliability
- Let the storage stack handle the physical distribution more intelligently

With this model, you do not have to create a parade of separate mount points just to fake good layout discipline.

### `NFS` and `dNFS`

If you use `NFS`, Oracle recommends using **Oracle Direct NFS (`dNFS`)** to take advantage of Oracle's built-in performance optimizations for database I/O over `NFS`.

`dNFS` helps by:

- Simplifying `NFS` administration from the database side
- Improving performance relative to generic client behavior
- Giving Oracle more direct control over how database I/O uses network-attached storage

`NFS` can be perfectly valid. It just stops being simple the moment someone says, "Storage over the network is basically the same thing as local disk."

---

## 3. Oracle ASM Storage (Oracle decides to manage the disks itself, and frankly it has opinions)

`Oracle ASM` is an alternative storage option for Oracle Database files.

Conceptually, `Oracle ASM` provides Oracle-managed storage by integrating functionality that would otherwise be split across:

- A file system
- A volume manager

Once `ASM` is configured, the database can store its files directly in ASM disk groups.

This is why `ASM` is attractive:

- Oracle manages disk layout at the storage layer
- I/O is distributed across disks automatically
- Manual storage micromanagement is reduced
- The DBA spends less time doing hand-tuned file placement like it's still 2004

Oracle's general guidance is to use a dedicated set of disks for `ASM`.

---

## 4. ASM Redundancy and Performance (because "hope the disk survives" is not a protection level)

`Oracle ASM` distributes I/O across disks to optimize performance and reduce the need for manual I/O tuning.

It also supports redundancy through mirroring policies.

Common redundancy choices include:

- **External redundancy**
  - Redundancy is handled outside `ASM`, typically by storage hardware

- **Normal redundancy**
  - Two-way mirroring

- **High redundancy**
  - Three-way mirroring

- **Flex redundancy**
  - Available with flex disk groups for more flexible redundancy behavior

The course transcript emphasized `normal`, `high`, and `flex`, but in real environments you also need to account for **external redundancy** whenever the storage layer beneath ASM is already doing the protection work.

The practical goal is always the same:

- Protect against disk failure
- Maintain availability
- Avoid hand-crafted recovery drama every time hardware gets ideas

---

## 5. Oracle ACFS (because eventually somebody wants to store normal files too)

`Oracle ASM` is not limited to core database files. Oracle also provides **Oracle ACFS**:

- **Oracle Advanced Cluster File System (`ACFS`)**

`ACFS` extends `ASM` into a general-purpose, multi-platform, scalable file system.

It is useful for storing files outside the core Oracle Database file set, such as:

- Application files
- Executables
- Images
- Videos
- Other non-database content

This is the important distinction:

- `ASM` disk groups can store Oracle Database files directly
- `ACFS` extends `ASM` so the same storage stack can also support broader file-system style use cases

So if the database team wants Oracle-managed storage and the application team also wants room for ordinary files, `ACFS` is how Oracle says, "Fine, but we're still doing it my way."

---

## 6. Persistent Disk Ownership and Naming for ASM (because reboots should not scramble reality)

When you use `ASM`, you must ensure that disk ownership, permissions, and device naming remain stable across reboots.

By default, Linux device names can be created dynamically during startup. If you rely on those defaults without planning, a reboot can change device names or ownership in ways that make the disks inaccessible to Oracle software.

That is how a perfectly functional storage design turns into a post-reboot scavenger hunt.

The course highlights three common approaches:

- Custom `udev` rules
- `ASMLib`
- `ASM Filter Driver (ASMFD)`

---

## 7. Option 1: Custom `udev` Rules (the explicit and durable approach)

Custom `udev` rules can be used to set:

- Persistent device names
- Persistent ownership
- Persistent permissions

This ensures that after each reboot the ASM disks are still presented with values Oracle can use.

This is the standard "be deliberate and write the rules down" approach. It works well, but only if someone actually writes the rules correctly instead of assuming Linux will remember what you meant.

---

## 8. Option 2: `ASMLib` (labeled disks, simpler discovery, fewer permission tantrums)

`ASMLib` is another option for managing ASM disks.

The course material highlights advantages such as:

- Simplified device scanning
- Built-in disk labeling
- Less direct fiddling with raw device permissions
- Efficient I/O handling for Oracle workloads

Operationally, `ASMLib` helps by giving Oracle a cleaner, labeled view of the disks so administrators spend less time arguing with device names and more time doing something marginally useful.

It is essentially Oracle saying, "If you are going to hand me disks, at least label them like an adult."

---

## 9. Option 3: `ASM Filter Driver (ASMFD)` (historically useful, currently not where Oracle is placing its long-term bets)

`ASM Filter Driver` can be configured during Oracle Grid Infrastructure installation.

Its goals include:

- Helping prevent accidental corruption of ASM disks
- Simplifying disk management by avoiding repeated device rebinding after reboot

That said, there is an important modern caveat:

- Current Oracle documentation marks `ASMFD` as **deprecated beginning with Oracle Database 19c** on Linux and Solaris

So if you are studying the course material, treat `ASMFD` as part of the historical option set, but do not assume it is the preferred long-term answer in current deployments.

This is one of those delightful moments where the training slides say, "Here are your options," and current documentation quietly replies, "Yes, but one of them is already leaving."

---

## 10. Practical Storage Guidance

Before installation, confirm:

- Whether the database will use file-system storage or `ASM`
- If file-system storage is used, whether the layout is based on basic disks, `LVM`, `RAID`, or `NFS`
- If `NFS` is used, whether `dNFS` should be enabled
- If `ASM` is used, whether disk groups and redundancy strategy have been chosen
- Whether `ACFS` is needed for non-database files
- How disk naming and ownership will remain persistent across reboots

Storage decisions are foundational. If you get them right, later admin work becomes boring in the best possible way. If you get them wrong, later admin work becomes educational in the worst possible way.

---

## 11. Wrap-Up (the disks are only quiet until you make bad choices)

This lesson covered the major storage decisions for Oracle software installation: file-system storage, `NFS` with `dNFS`, `ASM`, `ACFS`, and the device-persistence mechanisms used to keep ASM disk access stable. Next comes the actual software installation, where all of these careful planning choices finally stop being theory and start becoming directories, disk groups, and very serious-looking installer screens.
