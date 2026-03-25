## Lesson 2 - Installation Overview and Planning Decisions (where "just install it" meets reality and loses badly)

Before you install Oracle software, you need answers. Not vibes. This lesson covers the planning decisions that determine whether your installation goes smoothly or turns into a long afternoon of prerequisites, missing groups, and angry infrastructure teams pretending they were never part of the project.

By the end of this lesson, you should be able to:

- Identify the planning questions that should be answered before installation starts
- Distinguish between Oracle Database software, Oracle Grid Infrastructure, Oracle Clusterware, and Oracle Restart
- Compare single-owner and job-role separation installation models
- Differentiate single-instance and RAC database configurations
- Compare common Oracle storage options, including file systems, `NFS`, logical volumes, and `Oracle ASM`

---

## 1. Start With the Questions (because reinstalling is not a strategy)

Before installing anything, stop and ask the questions that determine what you actually need to deploy.

The major planning questions include:

- What Oracle software are you installing?
- Does the hardware meet the minimum required specifications?
- If multiple Oracle products are involved, what is the correct installation order?
- Are there prerequisite tasks that must be completed by teams other than the DBA?

These questions matter because Oracle installation is not one product, one checkbox, one destiny. The components you choose affect ownership, storage, networking, sequencing, and the amount of pain available later.

---

## 2. What Software Are You Installing? (the first fork in the road)

The first planning decision is identifying the Oracle software stack you need.

Typical options include:

- **Oracle Database software only**
  - Used when you only need the database binaries
  - Suitable when you are not deploying high-availability components or `Oracle ASM`

- **Oracle Grid Infrastructure for a standalone server**
  - Includes **Oracle Restart**
  - Used when you want local high-availability management and/or `Oracle ASM` on a standalone host

- **Oracle Grid Infrastructure for a cluster**
  - Includes **Oracle Clusterware**
  - Used for clustered environments such as `RAC`
  - Also includes the binaries needed for `Oracle ASM`

If you need `Oracle ASM` as the storage option, then you need **Oracle Grid Infrastructure**. That is not optional. `ASM` does not magically appear because somebody likes the acronym.

---

## 3. Single Owner vs Job Role Separation (same software, different political structure)

Another installation decision is the ownership model.

### Single-owner model

In a **single-owner** installation:

- One operating system owner manages the Oracle software
- The same owner may control both infrastructure and database components
- Administration is simpler in smaller or less segregated environments

This is operationally straightforward, which is why people like it. It is also less separated from a duties and governance perspective, which is why auditors start making that face.

### Job role separation model

In **job role separation**:

- Different operating system users own different Oracle software homes
- Responsibilities are separated between infrastructure management and database administration
- This is commonly used where security, operational control, or organizational boundaries require clearer separation

This model is more structured and usually more annoying to set up, which is often how you can tell it was designed for real enterprises.

---

## 4. Instance Type Decisions (single-instance vs RAC)

You also need to decide what kind of database configuration you are building.

### Single-instance database

A **single-instance database** means:

- One database
- One instance
- One server

This is simpler to install and manage, but the server and instance are obvious single points of failure.

### RAC configuration

A **Real Application Clusters (`RAC`)** configuration is a **multi-instance database** architecture:

- One database is accessed by multiple instances
- Instances run on multiple servers in a cluster
- `Oracle Clusterware` is required to coordinate the cluster
- Shared storage is required

`RAC` brings higher availability and scale-out capability, but it also brings more prerequisites, more networking requirements, and more opportunities for a misconfigured private interconnect to ruin your day.

---

## 5. Storage Choices (where the files live, and where mistakes live forever)

You must also choose the storage model for the Oracle database.

Common options include:

- **Regular file system**
  - Traditional file-based storage on local disks or mounted storage
  - Simple and familiar

- **`NFS`**
  - File-based storage provided across the network
  - Useful when supported and configured correctly

- **Logical volume management**
  - Abstracts physical storage into manageable logical volumes
  - Often used to simplify allocation and growth

- **`Oracle ASM`**
  - Oracle-managed storage layer
  - Designed specifically for Oracle workloads
  - Common in environments using Grid Infrastructure

The storage choice affects:

- Installation prerequisites
- How the database files are created and managed
- Whether Grid Infrastructure is required
- Which team has to be involved before installation begins

So yes, storage is a technical decision. It is also a scheduling decision, a permissions decision, and occasionally a bureaucratic hostage situation.

---

## 6. Hardware Requirements (because optimism is not a supported configuration)

Before installation, identify all hardware involved and verify that it meets Oracle's minimum suggested specifications.

That includes checking items such as:

- Server capacity
- Memory
- CPU resources
- Storage availability
- Networking components involved in the design

In clustered deployments, this applies to all participating nodes, not just the one currently sitting in front of you looking innocent.

If the hardware does not meet minimum requirements, the installation will not become successful through determination alone.

---

## 7. Installation Order Matters (Oracle is extremely fussy about sequence)

When multiple Oracle products are involved, the installation order matters.

Example:

1. Install **Oracle Grid Infrastructure** first
2. Install **Oracle Database software** second

This is the required order when you are using high-availability software such as `Oracle Clusterware`, or when the environment depends on Grid Infrastructure components.

If you reverse the order, you can still arrive at the same destination, but only by performing extra manual tasks that nobody wanted in the first place. So the recommended order is not decorative documentation. It exists to keep the deployment sane.

---

## 8. Prerequisites Owned by Other Teams (the part where the DBA discovers society)

Not every installation prerequisite is performed by the DBA.

Examples from this lesson include:

- Network configuration required to create a cluster
- Storage configuration required to use `Oracle ASM`
- System-level preparation performed by server, storage, or network teams

Before installation begins, determine:

- Which tasks belong to other teams
- What must be completed before the Oracle installer is run
- Whether those tasks affect timing, access, or sequencing

This is one of the easiest places for projects to derail. The DBA shows up ready to install. The network is not ready. The storage is not presented. The server team thought somebody else was creating groups. And now everyone attends a meeting that should have been an email.

---

## 9. Practical Planning Checklist

Before starting the installation, confirm:

- The exact Oracle products being installed
- Whether `Oracle ASM` is required
- Whether the environment is standalone or `RAC`
- Whether single-owner or job-role separation will be used
- Whether the storage model has been selected and provisioned
- Whether all hardware meets minimum requirements
- Whether Oracle products will be installed in the correct order
- Whether prerequisite tasks owned by other teams are complete

If these answers are fuzzy, then the installation plan is also fuzzy, and fuzzy Oracle plans usually become very specific disasters later.

---

## 10. Wrap-Up (planning first, heroics later)

This lesson covered the installation decisions that need to be made before Oracle software is deployed: software components, ownership model, instance type, storage model, hardware readiness, installation sequence, and prerequisite work from other teams. Next comes the hands-on preparation work, where the theory gets replaced by users, groups, paths, and the installer asking increasingly personal questions.
