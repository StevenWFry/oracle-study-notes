## Lesson 8 - Installing Oracle Database Software (where the database home arrives before the database itself does)

This lesson is about installing the Oracle Database software binaries, not creating the database yet. That distinction matters. `OUI` can do both in one pass if you let it, but in this course the cleaner story is to install the software first, then create the database separately so you keep more control and fewer surprises in the middle of the wizard.

By the end of this lesson, you should be able to:

- Identify the OS users and groups required for Oracle Database software installation
- Summarize the main 19c Linux hardware and storage requirements for the database home
- Explain the main `OUI` pages used during a database software-only installation
- Distinguish single-instance, `RAC`, and `RAC One Node` installation paths
- Confirm the operating system group mappings for database administrative privileges

---

## 1. Who Owns the Database Home (hint: not `grid`)

Oracle Database software is typically installed by the **database software owner**, usually:

- `oracle`

In a job-role-separation environment:

- `grid` owns the Grid Infrastructure home
- `oracle` owns the Database home

That is the expected split. If `grid` is installing the database home too, then your role separation has become interpretive dance.

The `oracle` user should already exist before installation, with:

- Primary inventory group such as `oinstall`
- Database administrative group such as `dba`
- Any optional groups needed for the environment

Common database-side privilege groups include:

- `dba` for `OSDBA` / `SYSDBA`
- `oper` for `OSOPER` / `SYSOPER`
- `backupdba` for `OSBACKUPDBA` / `SYSBACKUP`
- `dgdba` for `OSDGDBA` / `SYSDG`
- `kmdba` for `OSKMDBA` / `SYSKM`
- `racdba` for `OSRACDBA` / `SYSRAC`

These groups should already exist before `OUI` starts. The installer is very happy to map them. It is much less happy to invent them correctly on your behalf.

---

## 2. Database Software System Requirements (the host still has to be vaguely serious)

Oracle's 19c Linux guidance still uses a relatively modest baseline for the **database software** install, at least on paper.

### Memory

Minimum physical memory:

- **1 GB RAM minimum**
- **2 GB RAM recommended**

That is enough to install the software. It is not the same thing as "enough memory for a healthy production database," which is how people accidentally confuse a minimum requirement with a good idea.

### Swap

Swap guidance is based on physical `RAM`:

- If physical RAM is between **1 GB and 2 GB**, set swap to **1.5 times RAM**
- If physical RAM is between **2 GB and 16 GB**, set swap equal to RAM
- If physical RAM is **greater than 16 GB**, set swap to **16 GB**

### Oracle home disk space

This is one place where old course material and current Oracle docs drift a bit.

Older slide decks often cite:

- **7.5 GB** for Enterprise Edition
- **7.5 GB** for Standard Edition 2

Current Oracle 19c Linux installation guidance is a little lower for the software home itself:

- **At least 7.2 GB** for Enterprise Edition
- **At least 7.2 GB** for Standard Edition 2

So the transcript is directionally right, but the current documentation is more precise and slightly less dramatic.

Oracle also recommends allocating much more space overall for patching headroom later. Because one of the oldest Oracle traditions is pretending the install is the expensive part when future patching is clearly warming up in the parking lot.

### Temporary space and display

Minimum temp directory requirement:

- **1 GB** in `/tmp`

Display requirement for `OUI`:

- **1024 x 768** minimum

### Operating system support

The exact certified Linux versions, kernels, and packages change over time with release updates and support certifications.

So the safe rule is:

- Review the current Linux installation guide and certification matrix
- Do not assume an old course screenshot is your legal defense

The 19c preinstallation package commonly used for Linux remains:

- `oracle-database-preinstall-19c`

---

## 3. Start OUI and Decide About Security Updates

When you start `OUI` for Oracle Database software, the first page is typically **Configure Security Updates**.

You can:

- Enter a My Oracle Support email and password
- Receive security information and updates through My Oracle Support
- Or clear the option and continue without registering

Oracle recommends that you provide the contact information. Your environment may not care. Your security team may care very much. Those are related but not identical realities.

---

## 4. Select Installation Option (this is where you decide whether the wizard gets DBCA involved)

The installation option page typically offers:

- **Create and configure a database**
- **Install database software only**
- **Upgrade an existing database**

For this course flow, the important choice is:

- **Install database software only**

Why:

- The database home is installed first
- Database creation can be handled separately later
- You keep more explicit control over the database-creation step

If you choose **Create and configure a database**, `OUI` launches `DBCA` at the end. That is valid. It is just not the path this course is emphasizing in this lesson.

---

## 5. Choose the Database Installation Type

If Grid Infrastructure is present, `OUI` can prompt for the database installation type.

Common choices include:

- **Single instance database installation**
- **Oracle Real Application Clusters (`RAC`) database installation**
- **Oracle RAC One Node**

For the standalone-server path used in this course, the choice is:

- **Single instance database installation**

The `RAC` and `RAC One Node` choices matter in clustered environments, not here on the lonely little standalone server trying its best.

---

## 6. Select the Database Edition

The edition-selection page commonly offers:

- **Enterprise Edition**
- **Standard Edition 2**

`Enterprise Edition` is often the default.

One transcript point needed some caution here:

- In some course environments, or depending on the media and install path, `Standard Edition 2` may not appear as an available choice
- Treat that as an environment/media behavior, not a universal law that Grid Infrastructure somehow bans `SE2` from appearing on screen out of spite

In practice:

- Choose the edition that matches the licensed target environment
- Do not let the wizard make licensing decisions by accident

---

## 7. Set Oracle Base and Software Location

The install-location page asks for:

- **Oracle base**
- **Oracle software location** (the database home)

`Oracle base` is the broader directory used for Oracle software and related administrative files.

The software location is the actual database home path, for example:

```text
/u01/app/oracle/product/19.0.0/dbhome_1
```

`OUI` may prepopulate these values from:

- Existing environment variables
- Existing Oracle installations
- Prior inventory information on the host

Review them carefully. Defaults are often helpful, but they are not morally incapable of being wrong.

---

## 8. Map the Privileged Operating System Groups

The privileged operating system groups page maps database privilege groups to OS groups.

Typical mappings include:

- `OSDBA` -> `dba`
- `OSOPER` -> `oper`
- `OSBACKUPDBA` -> `backupdba`
- `OSDGDBA` -> `dgdba`
- `OSKMDBA` -> `kmdba`
- `OSRACDBA` -> `racdba`

These mappings determine which OS-authenticated users can connect with privileges such as:

- `SYSDBA`
- `SYSOPER`
- `SYSBACKUP`
- `SYSDG`
- `SYSKM`
- `SYSRAC`

This is why the groups had to be created earlier. `OUI` is not discovering your privilege model here. It is enforcing the one you should have already planned.

---

## 9. Prerequisite Checks and Summary

After the main choices are made, `OUI` runs prerequisite checks.

This page generally needs no input unless:

- A prerequisite fails
- A warning appears that you need to review

If the checks pass, `OUI` moves to the summary page.

On the summary page:

- Review all selections
- Use **Edit** if something is wrong
- Optionally save a response file for reuse in another environment

This is the last easy moment to fix a bad choice before Oracle turns it into installed reality.

---

## 10. Installation Progress and Root Scripts

Once installation starts, `OUI` displays the software installation progress.

At some point, a popup window appears to tell you that a root script must be run to complete the installation.

Depending on how root-script execution is configured in the environment, you either:

- Run the script manually as `root`
- Or confirm/allow automatic execution if that was preconfigured

This is the same basic privileged-script dance you already saw in the Grid Infrastructure install. Oracle believes firmly in repetition, especially when root is involved.

When the process completes successfully, the finish page reports that the Oracle Database software installation succeeded.

---

## 11. What This Lesson Does Not Do Yet

This lesson stops at **software installation**.

It does **not** yet create the database.

That happens next through a separate database-creation workflow, typically with `DBCA` or another controlled method. Which is exactly why choosing **Install database software only** is useful when you want the installer to stop touching things before it gets overly enthusiastic.

---

## 12. Practical Takeaways

Before starting the Oracle Database software install, confirm:

- The `oracle` software owner exists and owns the target database home
- The required OS groups are already created
- The host meets the 19c Linux memory, swap, temp, and storage requirements
- You know whether this is a single-instance, `RAC`, or `RAC One Node` path
- You know which edition should be installed
- The Oracle base and home paths are correct
- You are intentionally choosing whether `OUI` installs software only or also launches `DBCA`

If these decisions are vague going in, the installer will absolutely help you become specific in the most inconvenient possible way.

---

## 13. Wrap-Up (software home first, database later)

This lesson covered the OS users and groups behind Oracle Database software installation, the main 19c Linux requirements for the database home, and the `OUI` screens used to install the software. Next comes the database-creation step, where the binaries stop being theoretical and finally get a real instance to boss around.
