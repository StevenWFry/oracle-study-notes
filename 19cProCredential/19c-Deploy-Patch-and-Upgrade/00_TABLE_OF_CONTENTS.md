# Oracle Database 19c Deploy, Patch, and Upgrade - Table of Contents

This folder is now the full deployment-to-migration saga: install the thing, patch the thing, upgrade the thing, validate the thing, and finally migrate the thing when direct upgrade starts looking like a hostage situation.

The notes are written to be:

- technically accurate
- fast to review
- a little mean to bad Oracle decisions
- much more readable than the source material that spawned them

---

## Core Lessons

## 01 - Course Overview

- [01_Course_Overview.md](01_Course_Overview.md)

## 02 - Installation Overview and Planning Decisions

- [02_Installation_Overview_and_Planning_Decisions.md](02_Installation_Overview_and_Planning_Decisions.md)

## 03 - Software Owners and Privileged Operating System Groups

- [03_Software_Owners_and_Privileged_Operating_System_Groups.md](03_Software_Owners_and_Privileged_Operating_System_Groups.md)

## 04 - Root Privileges, OS Preparation, and Instance Configuration

- [04_Root_Privileges_OS_Preparation_and_Instance_Configuration.md](04_Root_Privileges_OS_Preparation_and_Instance_Configuration.md)

## 05 - Storage Options for Oracle Database and Oracle ASM

- [05_Storage_Options_for_Oracle_Database_and_Oracle_ASM.md](05_Storage_Options_for_Oracle_Database_and_Oracle_ASM.md)

## 06 - Preparing to Install Oracle Software

- [06_Preparing_to_Install_Oracle_Software.md](06_Preparing_to_Install_Oracle_Software.md)

## 07 - Installing Oracle Grid Infrastructure for a Standalone Server and Managing Oracle Restart

- [07_Installing_Oracle_Grid_Infrastructure_for_a_Standalone_Server.md](07_Installing_Oracle_Grid_Infrastructure_for_a_Standalone_Server.md)

## 08 - Installing Oracle Database Software

- [08_Installing_Oracle_Database_Software.md](08_Installing_Oracle_Database_Software.md)

## 09 - Creating the Oracle Database

- [09_Creating_the_Oracle_Database.md](09_Creating_the_Oracle_Database.md)

## 10 - Patching Grid Infrastructure and Database Software

- [10_Patching_Grid_Infrastructure_and_Database_Software.md](10_Patching_Grid_Infrastructure_and_Database_Software.md)

## 11 - Upgrading Oracle Grid Infrastructure to 19c

- [11_Upgrading_Oracle_Grid_Infrastructure_to_19c.md](11_Upgrading_Oracle_Grid_Infrastructure_to_19c.md)

## 12 - Choosing Oracle Database Upgrade Methods and Data Migration

- [12_Choosing_Oracle_Database_Upgrade_Methods_and_Data_Migration.md](12_Choosing_Oracle_Database_Upgrade_Methods_and_Data_Migration.md)

## 13 - Upgrading Oracle Database to 19c with AutoUpgrade

- [13_Upgrading_Oracle_Database_to_19c_with_AutoUpgrade.md](13_Upgrading_Oracle_Database_to_19c_with_AutoUpgrade.md)

## 14 - Upgrading Oracle Database with DBUA and Manual Methods

- [14_Upgrading_Oracle_Database_with_DBUA_and_Manual_Methods.md](14_Upgrading_Oracle_Database_with_DBUA_and_Manual_Methods.md)

## 15 - Post-Upgrade Tasks for Oracle Database

- [15_Post_Upgrade_Tasks_for_Oracle_Database.md](15_Post_Upgrade_Tasks_for_Oracle_Database.md)

## 16 - Migrating to Oracle Database 19c Using Data Pump

- [16_Migrating_to_Oracle_Database_19c_Using_Data_Pump.md](16_Migrating_to_Oracle_Database_19c_Using_Data_Pump.md)

---

## Labs

## Practice Environment

- [labs/practice-environment-credentials.md](labs/practice-environment-credentials.md)

## Deployment Preparation

- [labs/lab02-1-preparing-oracle-linux-for-grid-infrastructure-installation.md](labs/lab02-1-preparing-oracle-linux-for-grid-infrastructure-installation.md)
- [labs/lab02-2-preparing-storage-for-asm-using-udev-rules.md](labs/lab02-2-preparing-storage-for-asm-using-udev-rules.md)

## Grid Infrastructure Installation and ASM

- [labs/lab03-1-installing-oracle-grid-infrastructure.md](labs/lab03-1-installing-oracle-grid-infrastructure.md)
- [labs/lab03-2-creating-second-disk-group-using-asmca.md](labs/lab03-2-creating-second-disk-group-using-asmca.md)

## Oracle Database Software Installation

- [labs/lab04-1-installing-oracle-database-software.md](labs/lab04-1-installing-oracle-database-software.md)

## Database Creation with DBCA

- [labs/lab05-1-creating-an-oracle-database-using-dbca-gui.md](labs/lab05-1-creating-an-oracle-database-using-dbca-gui.md)
- [labs/lab05-2-creating-an-oracle-database-using-dbca-silent-mode.md](labs/lab05-2-creating-an-oracle-database-using-dbca-silent-mode.md)

## Patching and Gold Image Creation

- [labs/lab06-1-applying-a-patch-with-opatchauto.md](labs/lab06-1-applying-a-patch-with-opatchauto.md)
- [labs/lab06-2-applying-a-patch-manually-with-opatch.md](labs/lab06-2-applying-a-patch-manually-with-opatch.md)
- [labs/lab06-3-out-of-place-patching-grid-infrastructure-and-creating-a-gold-image.md](labs/lab06-3-out-of-place-patching-grid-infrastructure-and-creating-a-gold-image.md)
- [labs/lab06-4-out-of-place-patching-oracle-database-and-creating-a-gold-image.md](labs/lab06-4-out-of-place-patching-oracle-database-and-creating-a-gold-image.md)

## Oracle Restart Management

- [labs/lab07-1-managing-oracle-restart-with-crsctl.md](labs/lab07-1-managing-oracle-restart-with-crsctl.md)
- [labs/lab07-2-testing-oracle-restart-component-recovery.md](labs/lab07-2-testing-oracle-restart-component-recovery.md)
- [labs/lab07-3-managing-oracle-restart-resource-configuration.md](labs/lab07-3-managing-oracle-restart-resource-configuration.md)

## Grid Infrastructure Upgrade

- [labs/lab08-1-preconfiguring-for-grid-infrastructure-upgrade.md](labs/lab08-1-preconfiguring-for-grid-infrastructure-upgrade.md)
- [labs/lab08-2-upgrading-oracle-grid-infrastructure-to-19c.md](labs/lab08-2-upgrading-oracle-grid-infrastructure-to-19c.md)

## Database Upgrade with AutoUpgrade

- [labs/lab10-1-installing-the-19c-database-home-for-upgrade.md](labs/lab10-1-installing-the-19c-database-home-for-upgrade.md)
- [labs/lab10-2-backing-up-the-database-before-upgrade.md](labs/lab10-2-backing-up-the-database-before-upgrade.md)
- [labs/lab10-3-running-autoupgrade-analyze-and-fixups.md](labs/lab10-3-running-autoupgrade-analyze-and-fixups.md)
- [labs/lab10-4-upgrading-the-orcl-database-with-autoupgrade.md](labs/lab10-4-upgrading-the-orcl-database-with-autoupgrade.md)

## Post-Upgrade Validation and Cleanup

- [labs/lab11-1-performing-post-upgrade-validation-and-cleanup.md](labs/lab11-1-performing-post-upgrade-validation-and-cleanup.md)

## Data Pump Migration

- [labs/lab12-1-migrating-a-pdb-to-19c-using-data-pump.md](labs/lab12-1-migrating-a-pdb-to-19c-using-data-pump.md)

---
