## Lesson 1 - Course Overview (where we volunteer to install, patch, and upgrade the thing that runs payroll)

Welcome to Oracle Database 19c: Deploy, Patch, and Upgrade. This course is for the people who get handed an environment and told, "Just keep it working," which is management-speak for "Please prevent fire without interrupting service."

By the end of this lesson, you should be able to:

- Identify the target audience and prerequisites for the course
- Summarize the three major course tracks: deployment, patching, and upgrading
- Explain what parts of the Oracle stack will be installed, patched, and upgraded
- Recognize where post-upgrade tasks and Data Pump migration fit into the bigger picture

---

## Course Table of Contents

- Lesson 2 - Installation Overview and Planning Decisions
- Lesson 3 - Software Owners and Privileged Operating System Groups
- Lesson 4 - Root Privileges, OS Preparation, and Instance Configuration
- Lesson 5 - Storage Options for Oracle Database and Oracle ASM
- Lesson 6 - Preparing to Install Oracle Software
- Lesson 7 - Installing Oracle Grid Infrastructure for a Standalone Server and Managing Oracle Restart
- Lesson 8 - Installing Oracle Database Software
- Lesson 9 - Creating the Oracle Database
- Lesson 10 - Patching Grid Infrastructure and Database Software
- Lesson 11 - Upgrading Oracle Grid Infrastructure to 19c
- Lesson 12 - Choosing Oracle Database Upgrade Methods and Data Migration
- Lesson 13 - Upgrading Oracle Database to 19c with AutoUpgrade
- Lesson 14 - Upgrading Oracle Database with DBUA and Manual Methods
- Lesson 15 - Post-Upgrade Tasks for Oracle Database
- Lesson 16 - Migrating to Oracle Database 19c Using Data Pump

---

## 1. Who This Course Is For (the usual suspects)

This course is aimed at:

- Developers maintaining their own development environments
- System administrators supporting departmental Oracle databases
- Database administrators responsible for enterprise environments

In other words: anyone who might be expected to install Oracle software correctly, keep it patched, and then upgrade it later without turning the server room into a grief counseling session.

---

## 2. Prerequisites (mercifully light)

This is positioned at the **beginning to intermediate** level.

You do **not** need prior experience administering an Oracle database, which is generous of Oracle and slightly terrifying for everyone else.

Helpful background includes:

- Basic computer knowledge
- Basic database knowledge
- General comfort working with software installations and system-level tasks

If you already know what a database is, what an operating system user is, and why patching production on a Friday is a terrible personality trait, you are in acceptable shape.

---

## 3. Deployment Track (building the thing before we can break it)

The first major topic is **deploying Grid Infrastructure and Oracle Database software**.

This part of the course covers:

- The overall deployment process
- Required operating system users and groups
- Core architectural concepts
- Storage considerations
- Preparation steps before installation
- Installing the software and creating the Oracle database

This is the "build the runway before landing the aircraft" phase. Oracle installation is much less exciting when you prepare properly, which is exactly what you want.

Operationally, this matters because bad early decisions about users, groups, storage, or architecture do not stay politely contained. They come back later during patching, upgrades, troubleshooting, and maintenance windows, usually carrying a bat.

---

## 4. Patching Track (because "we'll do it later" is how estates become haunted)

The second major topic is **patching both Grid Infrastructure and database software**.

You will cover:

- The overall patching process
- Types of Oracle patches
- Tools available for applying patches

This matters because Oracle software is not a decorative object. It needs maintenance. Patching is how you address bug fixes, security issues, and supportability requirements without pretending the environment will somehow heal itself through positive thinking.

You should expect patching decisions to depend on:

- What component is being patched
- What patch type is being applied
- What tooling Oracle provides for that patch path
- How much downtime or coordination the maintenance requires

---

## 5. Upgrade Track (the controlled demolition portion of the syllabus)

The third major topic is **upgrading Grid Infrastructure and Oracle Database software**.

This section includes:

- Upgrading Grid Infrastructure
- Upgrading the database software
- Performing post-upgrade tasks

An upgrade is not just "install newer bits and hope for enlightenment." It is a controlled transition where compatibility, sequencing, validation, and cleanup all matter.

Post-upgrade work is part of the job, not an optional afterparty. If the upgrade finishes but validation, cleanup, and follow-up tasks are skipped, you have not completed an upgrade. You have merely created a newer mystery.

---

## 6. Alternative Upgrade Path: Data Pump Migration (when direct upgrade is not the winning move)

The course also introduces an alternative approach for moving to a newer environment:

- Migrating data by using the **Data Pump** utility

This is useful because not every upgrade story needs to be a direct in-place journey through the Valley of Regret. In some cases, exporting and importing data into a new environment is the cleaner or more practical path.

The important takeaway is that "upgrade" is not always synonymous with "run the upgrade tool in place." Sometimes the better answer is migration.

---

## 7. What You Should Expect to Learn

Across the full course, you will learn how to:

- Prepare systems for Oracle Grid Infrastructure and database software installation
- Understand the OS, storage, and architecture requirements behind a successful deployment
- Identify patch categories and the tools used to apply them
- Upgrade Oracle software in a structured way
- Perform the necessary tasks after an upgrade
- Recognize when a Data Pump migration may be a viable alternative

This is the lifecycle view: build it, maintain it, modernize it, and try not to discover major mistakes during an outage bridge.

---

## 8. Wrap-Up (the calm before the installers start asking questions)

This lesson framed the course: who it is for, what background you need, and how the material is organized into deployment, patching, and upgrading. Next, the course moves into the deployment process, where the real fun begins and the checklists start breeding.

And yes, when the course is over, Oracle would like your feedback. Continuous improvement is the official phrase. Unofficially, they would prefer to know which parts caused the most suffering.
