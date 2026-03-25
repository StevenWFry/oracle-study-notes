## Lesson 6 - Preparing to Install Oracle Software (where the real install starts long before the installer does)

Oracle software installation succeeds or fails long before you click through the GUI. This lesson is about the practical preparation work: verifying hardware and memory, updating Oracle Linux, creating the right accounts and groups for job-role separation, setting session limits, and preparing the directory layout so the installer has somewhere sensible to live instead of improvising on your behalf.

By the end of this lesson, you should be able to:

- Verify server hardware, memory, swap, and temporary-space requirements before installation
- Update Oracle Linux and understand where the Oracle preinstallation `RPM` fits
- Create or modify users and groups for job-role separation
- Extend resource limits for additional software owners such as `grid`
- Prepare initial installation directories for software, database files, and recovery files

---

## 1. Check the Server Before You Check Your Optimism

Before installing Oracle software, verify that the server meets minimum requirements in several areas.

Important checks include:

- Physical `RAM`
- Swap space
- Temporary directory free space
- Processor architecture
- Shared memory (`/dev/shm`)

If any of these are undersized, the right fix is not "continue anyway and see what happens." The right fix is to stop and correct the system first.

### Memory and swap

Check whether the installed physical memory meets Oracle's minimum requirement.

If physical `RAM` is too small:

- Install more memory before continuing

If swap is insufficient:

- Review operating system procedures for adding more swap
- Extend swap before starting the installation

This is not optional tuning. It is basic survival.

### Temporary space

Check the free space available in the temporary directory, typically `/tmp`.

If `/tmp` does not have enough free space:

- Delete unnecessary files
- Or set `TMP` and `TMPDIR` to a different directory with enough space

This is a very common installer tantrum source. The binaries are ready, the user is ready, and then `/tmp` quietly says, "Absolutely not."

### Architecture and shared memory

You can use:

```bash
uname -m
df -h /dev/shm
free -h
```

These checks help confirm:

- The processor architecture matches the Oracle software release
- Shared memory is mounted and large enough
- `RAM` and swap availability are visible before installation begins

If `uname -m` does not report the expected architecture, that is not a warning. That is the system telling you this install is going nowhere.

---

## 2. Update Oracle Linux First (because old packages have a hobby of becoming support tickets)

Oracle strongly recommends updating Oracle Linux before the installation to pick up:

- Security advisories
- Bug fixes
- Package improvements

Older course material often uses commands such as:

```bash
yum -y update
yum -y upgrade
```

On newer Oracle Linux releases, `dnf` may be the native package manager even if `yum` compatibility still exists.

The practical point is:

- Update the operating system packages before the install
- Reboot if a new kernel or other critical package requires it

If the host has been updated but not rebooted into the new kernel, congratulations, you now have one of those environments where nobody is entirely sure what is actually running.

---

## 3. Oracle Preinstallation RPM (the package that handles the boring parts so humans can stop breaking them)

Oracle recommends using the Oracle preinstallation `RPM` where possible because it automates many operating-system preparation tasks.

Typical benefits include:

- Installing required prerequisite packages
- Creating the default `oracle` user
- Creating the `oinstall` and `dba` groups
- Setting kernel parameters
- Configuring other OS-level prerequisites

Typical package naming follows the Oracle release, for example:

- `oracle-database-preinstall-19c`

This does not eliminate all preparation work, but it removes a large amount of repetitive setup that humans are otherwise happy to do inconsistently.

One important limitation:

- The preinstallation `RPM` typically prepares the default `oracle` account
- If you use job-role separation with an additional owner such as `grid`, you still need to configure that account and its limits yourself

So yes, the `RPM` saves time. No, it does not finish the job while you go get coffee.

---

## 4. Create Groups for Job Role Separation (optional, until it isn't)

If the environment uses a **single owner** model, you may not need the extended set of privilege groups.

If the environment uses **job-role separation**, then you create additional operating system groups for:

- Oracle Grid Infrastructure
- Oracle Database software

The numeric `GID` values shown in Oracle examples are only examples.

What matters is that the group IDs:

- Are unique on the server
- Match your organization's standards

Common job-role-separation groups include:

- `oinstall`
- `dba`
- `asmadmin`
- `asmdba`
- `asmoper`
- `backupdba`
- `dgdba`
- `kmdba`
- `racdba`

This is one of those areas where documentation examples are not commandments. Use sane, unique IDs. Do not treat the sample numbers like sacred relics.

---

## 5. Modify the `oracle` Account (because the RPM gives you a starter kit, not a finished identity)

The preinstallation `RPM` creates the `oracle` account by default, but it does **not** set a password.

So you must set one manually, for example:

```bash
passwd oracle
```

After the `RPM` install, the default `oracle` account is typically configured with:

- `oinstall` as the primary group
- `dba` as a secondary group

If you are using job-role separation, modify the `oracle` account so it belongs to any additional required groups.

Examples of useful secondary memberships include:

- `asmdba`
  - Access to `ASM`-managed storage

- `backupdba`
  - Backup and recovery administration

- `dgdba`
  - Data Guard and Data Guard Broker tasks

- `kmdba`
  - Encryption key and wallet management

- `racdba`
  - `RAC`-specific operations where required

This is where privilege design becomes real. If the account will need those capabilities, assign the right groups now instead of discovering the omission during a maintenance window.

---

## 6. Create the `grid` Account for Grid Infrastructure (the transcript said "create"; the operating system would prefer an actual username)

If you are using job-role separation and installing Grid Infrastructure with a separate owner, create a dedicated `grid` user.

Typical characteristics for the `grid` user include:

- Primary group: `oinstall`
- Membership in `asmadmin`
- Membership in `asmdba`
- Membership in `asmoper`
- Membership in `dba` where required by the installation model

After creating the `grid` account:

- Set its password to a value that meets your organization's security rules

The important distinction is:

- `oracle` typically owns the Database home
- `grid` typically owns the Grid Infrastructure home

If both homes are owned by the same account in a role-separated environment, then congratulations, you built a separation model entirely out of decorative language.

---

## 7. PAM Limits and Session Resources (because Oracle processes do not thrive under tiny default limits)

The Oracle preinstallation `RPM` also configures **PAM limits** for the Linux Pluggable Authentication Modules stack.

These limits control resources a user session can consume, such as:

- Open files
- Processes
- Stack size
- Other session-related limits

Depending on the operating system version and the specific preinstallation package, the limits for `oracle` may be defined in:

- A default limits file
- Or an individual file under a limits configuration directory

If you are using job-role separation, then the `grid` user needs equivalent resource limits as well.

Typical approach:

- Review the file that defines the `oracle` limits
- Copy the same entries for `grid`
- Or create a separate limits file dedicated to `grid`

This is another classic trap. The Oracle user gets the correct limits, the Grid user does not, and then the environment behaves like it resents your entire family line.

---

## 8. Create Initial Installation Directories (the filesystem should not look surprised when Oracle arrives)

Before installation, create the base directories needed for the software homes.

These initial directories are used for:

- Oracle Grid Infrastructure software
- Oracle Database software

In a job-role-separation model, the key difference is ownership:

- Grid Infrastructure directories are owned by `grid`
- Database software directories are owned by `oracle`

This matters because a correct directory path with the wrong owner is still wrong.

---

## 9. Create Database and Recovery Storage Locations

If the database will use **file-system storage**, also create directories for:

- Oracle database files
- Fast Recovery Area (`FRA`) files

Oracle's operationally sane recommendation is to keep recovery files separate from the main database files.

For file-system deployments, that usually means:

- One mount point or directory tree for database files
- A separate mount point or directory tree for recovery-related files

The reason is straightforward:

- Separation improves manageability
- It reduces shared-failure risk
- Losing the main data mount and the recovery area at the same time is a terrible way to learn about storage design

If the database will use **ASM**, the same principle still applies:

- Create one disk group for database files
- Create another disk group for recovery-related files

This is not mandatory in every conceivable design, but it is very often the right move if you enjoy having recovery options after something breaks.

---

## 10. Practical Preparation Checklist

Before running the installer, confirm:

- `RAM`, swap, `/tmp`, and `/dev/shm` meet requirements
- The processor architecture matches the Oracle software release
- Oracle Linux packages are updated and the server has been rebooted if required
- The correct preinstallation `RPM` is installed if you are using that approach
- All required OS groups exist
- The `oracle` account has the right password and secondary group memberships
- The `grid` account exists and has matching resource limits if job-role separation is used
- Software home directories exist with the correct ownership
- Database and recovery storage locations are created separately where appropriate

If this checklist is incomplete, the installer will eventually complete it for you by failing in a way that wastes more time.

---

## 11. Wrap-Up (preparation is the install before the install)

This lesson covered the preparation work required before Oracle software installation: hardware and memory checks, Oracle Linux updates, use of the preinstallation `RPM`, job-role-separation accounts and groups, PAM limits, and initial directory layout. Next comes the actual software installation, where all this prep either looks wonderfully boring or catastrophically insufficient.
