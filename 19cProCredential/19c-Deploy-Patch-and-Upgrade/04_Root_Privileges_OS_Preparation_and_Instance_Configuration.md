## Lesson 4 - Root Privileges, OS Preparation, and Instance Configuration (where the installer asks for power, and you decide how much trust to fake)

Oracle software installation is not just about binaries and disk space. At some point the installer needs root-level help, the operating system needs to be prepared properly, and you still have to decide whether you are building a simple single-instance database or a multi-node `RAC` arrangement with shared storage and more moving parts than a committee meeting.

By the end of this lesson, you should be able to:

- Explain why Oracle installation requires root-privilege access
- Compare the three common methods for running required root scripts
- Distinguish manual operating system preparation from the Oracle preinstallation `RPM` approach
- Summarize what the Oracle preinstallation `RPM` can configure for you
- Compare single-instance and `RAC` instance configurations

---

## 1. Why Root Access Is Needed (because some tasks are above the pay grade of `oracle`)

Installing Oracle software products requires operating system **root privileges** at specific points in the process.

This is typically needed to run one or more mandatory scripts that complete privileged setup tasks for the installation.

The practical point is simple:

- The Oracle software owner can start the installation
- The installer pauses when privileged work must be done
- Someone with the right authority must complete that work

So before the install starts, decide **how** those root-level tasks will be handled. Waiting until the popup appears is the systems-administration equivalent of beginning a flight checklist after takeoff.

---

## 2. Method 1: Run the Root Scripts Manually (the traditional handoff ceremony)

The first option is to run the required scripts **manually**.

With this method:

- You clear or unselect the option to **Automatically Run Configuration Scripts**
- The installer pauses partway through the installation
- A popup window shows that mandatory scripts must be run
- A root user runs the required script or scripts manually

In practice, this usually means the DBA stops, waves politely at whoever has root access, and waits for the operating system equivalent of a royal signature.

Use this method when:

- Root access is tightly controlled
- The installer does not know the root password
- Your organization requires explicit manual approval of privileged actions

This is slower, but it is often the cleanest approach in security-conscious environments.

---

## 3. Method 2: Supply the Root Password to the Installer (convenient, if your policies allow it)

The second option is to provide the **root password** directly to the installation program.

With this method:

- The person running the installation knows the root password
- The installer runs the required scripts automatically
- A dialog window indicates that the scripts are being run

This is commonly used when the DBA is also trusted with root credentials or when the host is a lower-friction environment such as a lab or dev system.

The obvious tradeoff:

- It is faster and more convenient
- It may violate local security policy in environments where DBAs are not supposed to know or use the root password

Convenience is wonderful right up until the audit team arrives and starts asking who typed what.

---

## 4. Method 3: Use a Sudo-Enabled Account (same automation, less root-password drama)

The third option is to use a different account that has been configured with Linux `sudo` privileges.

With this method:

- The installer uses a designated account and password
- That account is allowed to elevate with `sudo`
- The installer runs the required scripts automatically
- A dialog indicates that the scripts are being executed

This gives you the automation benefits of Method 2 without requiring the installer operator to work directly as root.

Use this method when:

- Your organization does not allow sharing the root password
- You still want the installer to run required scripts automatically
- `sudo` is the standard privilege-escalation method on your Linux systems

This is often the sweet spot: automation without pretending root passwords should be party favors.

---

## 5. Choosing a Root-Script Method (security policy wins, not personal preference)

The right method depends on:

- Whether the installer operator knows the root password
- Whether the organization permits direct use of root credentials
- Whether `sudo` has been configured and approved
- How strictly duties are separated between DBAs and system administrators

General guidance:

- **Manual root scripts**: best when privileged actions must be explicitly controlled
- **Root password in installer**: best only where policy allows and simplicity matters
- **Sudo-enabled account**: best when automation is wanted but direct root sharing is restricted

The installer does not care about your org chart. It just needs the scripts to run. Unfortunately, your security team cares very much.

---

## 6. Operating System Preparation: Manual vs Preinstallation RPM

Before installing Oracle software, you must also prepare the operating system.

This lesson presents two broad approaches:

- **Manual preparation**
- **Oracle preinstallation `RPM`**

### Manual preparation

With the manual method:

- You review the installation guide
- You perform each prerequisite step yourself
- You create the required users and groups
- You adjust operating system settings manually

This works, but it depends on careful execution. Humans are fully capable of skipping steps, mistyping parameters, and then blaming the documentation with impressive confidence.

### Oracle preinstallation `RPM`

Oracle recommends using the preinstallation `RPM` where it is available and appropriate.

The goal is simple:

- Automate many prerequisite tasks
- Reduce configuration mistakes
- Standardize host preparation

If the operating system can be prepared consistently by a package instead of improvisation, that is usually the less embarrassing choice.

---

## 7. What the Oracle Preinstallation RPM Does (the package that saves you from yourself)

The Oracle preinstallation `RPM` can automate many operating system preparation tasks, including:

- Creating required OS users
- Creating required OS groups
- Modifying kernel parameters
- Applying other prerequisite operating system settings
- Recording what it changed and preserving a backup of prior settings

The package name depends on factors such as:

- The Oracle software version being installed
- The Oracle Linux version in use

So the exact `RPM` name is not universal. You do not just yell "install the Oracle package" at Linux and expect maturity to happen.

The main operational benefit is consistency:

- Fewer manual errors
- Faster host preparation
- Easier repeatability across environments

---

## 8. Instance Configuration Decision: Single Instance vs RAC

Another installation decision is the **instance configuration**.

### Single-instance database configuration

In a **single-instance** configuration:

- There is one physical database on storage
- There is one database instance that accesses it
- The database-to-instance ratio is effectively **1:1**

This is the simpler model:

- Fewer components
- Easier installation
- Easier day-to-day administration

The downside is obvious: one instance means one main access point, so host or instance failure is a much bigger problem.

### Real Application Clusters (`RAC`) configuration

In a **`RAC`** configuration:

- One physical database is placed on shared storage
- Multiple database instances access that same database
- Each instance acts as an access point to the database

This provides a **multi-instance** architecture that supports higher availability.

Why people choose `RAC`:

- Better protection against server failure
- Better protection against certain hardware failures
- Multiple instances can continue providing access to the same database

Why people do not choose `RAC` casually:

- More infrastructure
- More configuration work
- Shared storage requirements
- More places for networking and cluster coordination to become your personality for the week

---

## 9. Practical Decision Checklist

Before proceeding with installation, confirm:

- Which method will be used to run mandatory root scripts
- Whether the installer operator can use root directly, or must rely on `sudo`
- Whether OS preparation will be manual or use the Oracle preinstallation `RPM`
- Whether the required users, groups, and kernel settings are already in place
- Whether the instance model is **single-instance** or **`RAC`**
- Whether the selected instance model matches the availability requirements of the environment

If these answers are not clear before installation starts, the install session will eventually force the issue in the least convenient way possible.

---

## 10. Wrap-Up (before the actual installation gets a vote)

This lesson covered three major installation decisions: how required root scripts will be run, how the operating system will be prepared, and whether the database will use a single-instance or `RAC` configuration. Next comes the actual software installation work, where the planning either pays off or sends everyone back to the whiteboard with worse attitudes.
