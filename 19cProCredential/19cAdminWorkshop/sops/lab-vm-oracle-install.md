# SOP: Oracle Database 19c — Lab VM Install on Oracle Linux 8

**Purpose:** Set up a personal Oracle Database lab environment for practicing 1Z0-082/083 DBA topics and SQL training against the correct exam release.
**Target OS:** Oracle Linux 8 (OL8) — officially certified for Oracle 19c without patching
**Database:** Oracle Database 19c Enterprise Edition (OTN Developer License — free for personal dev/test/learning)
**Time to complete:** ~60–90 minutes including OS install

> **Why not 23ai Free?** The 1Z0-082 exam tests 19c-specific behaviour. DBA topic answers (parameters, features, shutdown behaviour, etc.) can differ between releases. Use the version the exam tests.
>
> **Why not OL9?** Oracle 19c requires a Release Update (19.19+) for OL9 support. OL8 works cleanly out of the box and saves a patching step in a lab.

---

## Part 1 — Minimum Hardware Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU | 2 cores | 4 cores |
| RAM | 2 GB | 4–8 GB |
| Disk (OS) | 20 GB | 30 GB |
| Disk (Oracle) | 15 GB | 30 GB |
| Swap | Equal to RAM | Equal to RAM |

> **VM tip:** If using VirtualBox or VMware, create a separate virtual disk for Oracle data mounted at `/u01`. Keeps things clean and easy to snapshot before destructive practice.

---

## Part 2 — Install Oracle Linux 8

### 2.1 Download Oracle Linux 8

Oracle Linux is **free** — no subscription needed.

- **Download:** https://yum.oracle.com/oracle-linux-isos.html
- Choose **Oracle Linux 8** — pick the `x86_64` full ISO

### 2.2 VM / Bare Metal Setup

1. Create your VM or prepare your physical machine
2. Boot from the Oracle Linux 8 ISO
3. On the installer welcome screen, select **Install Oracle Linux 8**

### 2.3 Key Installer Choices

| Section | Setting |
|---------|---------|
| **Software Selection** | **Server** (not Workstation — saves RAM) |
| **Root password** | Set a strong root password |
| **User creation** | Create a regular user (e.g. `oracle_student`) |
| **Disk partitioning** | See below |
| **Network** | Enable your NIC — set a hostname (e.g. `oralab.local`) |

### 2.4 Recommended Disk Partitioning (Manual, LVM)

| Mount Point | Size | Notes |
|-------------|------|-------|
| `/boot` | 1 GB | Standard |
| `/boot/efi` | 600 MB | If UEFI |
| `swap` | = RAM size | Required by Oracle preinstall |
| `/` | 20 GB | OS root |
| `/u01` | Remaining | Oracle software and data lives here |

> If you added a second virtual disk for Oracle, skip `/u01` in the partitioner and set it up after install (see Part 3.3).

### 2.5 Complete the Install

- Click **Begin Installation** and let it run
- **Reboot** when prompted
- Remove the ISO from the virtual drive before reboot

---

## Part 3 — Post-OS Configuration

Log in as root or your user with `sudo`.

### 3.1 Set the Hostname

```bash
sudo hostnamectl set-hostname oralab.local
```

Also add it to `/etc/hosts` so Oracle can resolve it:

```bash
echo "127.0.0.1  oralab.local oralab" | sudo tee -a /etc/hosts
```

### 3.2 Update the OS

```bash
sudo dnf update -y
```

### 3.3 Configure /u01 (if using a second disk)

If Oracle data is on a second virtual disk (e.g. `/dev/sdb`):

```bash
# Create a partition
sudo fdisk /dev/sdb
# Press: n → p → 1 → Enter → Enter → w

# Format it
sudo mkfs.xfs /dev/sdb1

# Create the mount point
sudo mkdir -p /u01

# Get the UUID
sudo blkid /dev/sdb1

# Add to /etc/fstab (replace UUID with your actual value)
echo 'UUID=your-uuid-here /u01 xfs defaults 0 2' | sudo tee -a /etc/fstab

# Mount and verify
sudo mount -a
df -h /u01
```

### 3.4 Disable Firewall (Lab Only)

> ⚠️ Personal lab VM only — never in production.

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

### 3.5 Set SELinux to Permissive (Lab Only)

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
```

Verify it took:

```bash
grep ^SELINUX= /etc/selinux/config
# Should show: SELINUX=permissive
```

---

## Part 4 — Download Oracle Database 19c

Oracle 19c is available for free under the **OTN Developer License** for personal development, testing, and learning. A free Oracle account is required.

### 4.1 Create a Free Oracle Account (if needed)

Go to: **https://profile.oracle.com/myprofile/account/create-account.jspx**

### 4.2 Download the 19c RPM Installer

Go to: **https://www.oracle.com/database/technologies/oracle-database-software-downloads.html**

Under **Oracle Database 19c**, select:
- **Linux x86-64**
- Download the **RPM** package: `oracle-database-ee-19c-1.0-1.x86_64.rpm`

> Oracle also offers a zip-based OUI installer. The RPM method is simpler for a lab and produces the same result.

Transfer the RPM to your VM (USB, shared folder, or scp).

---

## Part 5 — Install Oracle Database 19c

### 5.1 Install the Preinstall RPM

The preinstall RPM handles all OS prerequisites in one command — kernel parameters, OS limits, oracle user and group creation, and required package dependencies.

```bash
sudo dnf install -y oracle-database-preinstall-19c
```

This creates:
- OS user: `oracle`
- OS groups: `oinstall`, `dba`, `oper`, `backupdba`, `dgdba`, `kmdba`, `racdba`
- All required kernel parameters in `/etc/sysctl.d/99-oracle-database-preinstall-19c-sysctl.conf`
- All required OS user limits in `/etc/security/limits.d/oracle-database-preinstall-19c.conf`

### 5.2 Set a Password for the Oracle OS User

```bash
sudo passwd oracle
```

### 5.3 Install the Database RPM

```bash
sudo dnf localinstall -y oracle-database-ee-19c-1.0-1.x86_64.rpm
```

This installs the Oracle software under `/opt/oracle/product/19c/dbhome_1`. It does **not** create the database yet.

### 5.4 Create and Configure the Database

```bash
sudo /etc/init.d/oracledb_ORCLCDB-19c configure
```

This step:
- Creates a CDB named **`ORCLCDB`**
- Creates a default PDB named **`ORCLPDB1`**
- Configures the listener on port 1521
- Takes 10–20 minutes depending on hardware

The SYS and SYSTEM passwords are set to `Oradoc_db1` by default. You can change them after install.

When complete, you should see:

```
Database configuration completed successfully. The passwords were auto generated,
you must change them by connecting to the database using 'sqlplus / as sysdba'
as the oracle user.
```

### 5.5 Enable Oracle to Start Automatically on Boot

```bash
sudo systemctl enable oracledb_ORCLCDB-19c
sudo systemctl start oracledb_ORCLCDB-19c
```

---

## Part 6 — Verify the Installation

### 6.1 Switch to the Oracle User

```bash
sudo su - oracle
```

### 6.2 Set Environment Variables

Add these to `/home/oracle/.bash_profile` for persistence:

```bash
cat >> ~/.bash_profile << 'EOF'

# Oracle Environment
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=ORCLCDB
export PATH=$ORACLE_HOME/bin:$PATH
EOF

source ~/.bash_profile
```

### 6.3 Change the Default Passwords

```bash
sqlplus / as sysdba
```

```sql
ALTER USER sys   IDENTIFIED BY YourNewPassword;
ALTER USER system IDENTIFIED BY YourNewPassword;
ALTER SESSION SET CONTAINER = ORCLPDB1;
ALTER USER pdbadmin IDENTIFIED BY YourNewPassword;
EXIT
```

### 6.4 Verify the Database

```bash
sqlplus / as sysdba
```

```sql
SELECT name, cdb, open_mode, db_unique_name FROM v$database;
SHOW PDBS;
EXIT
```

Expected output:

```
NAME      CDB  OPEN_MODE  DB_UNIQUE_NAME
--------- ---- ---------- --------------
ORCLCDB   YES  READ WRITE ORCLCDB

CON_ID CON_NAME   OPEN MODE  RESTRICTED
------ ---------- ---------- ----------
     2 PDB$SEED   READ ONLY  NO
     3 ORCLPDB1   READ WRITE NO
```

### 6.5 Connect to the PDB

```bash
sqlplus sys/YourNewPassword@localhost:1521/ORCLPDB1 as sysdba
```

---

## Part 7 — Install the HR Sample Schema

### 7.1 Install Git and Clone the Sample Schemas

```bash
sudo dnf install -y git
git clone https://github.com/oracle-samples/db-sample-schemas.git
cd db-sample-schemas/human_resources
```

### 7.2 Run the HR Install Script

```bash
sqlplus sys/YourNewPassword@localhost:1521/ORCLPDB1 as sysdba
```

```sql
@hr_install.sql
```

When prompted:
- HR user password → `hr` (or whatever you prefer)
- Default tablespace → press Enter (`USERS`)
- Temp tablespace → press Enter (`TEMP`)
- Log file location → press Enter (current directory)

### 7.3 Verify HR

```sql
CONNECT hr/hr@localhost:1521/ORCLPDB1

SELECT table_name FROM user_tables ORDER BY table_name;
```

Expected tables:

```
COUNTRIES
DEPARTMENTS
EMPLOYEES
JOB_HISTORY
JOBS
LOCATIONS
REGIONS
```

---

## Part 8 — Optional: Install SQL Developer

```bash
# Install Java
sudo dnf install -y java-11-openjdk

# Download SQL Developer from:
# https://www.oracle.com/tools/downloads/sqldev-downloads.html
# Get the "Other Platforms" zip (no JDK included)

unzip sqldeveloper-*.zip -d /opt/
sudo ln -s /opt/sqldeveloper/sqldeveloper.sh /usr/local/bin/sqldeveloper

# Run it
sqldeveloper &
```

Connection settings for SQL Developer:

| Field | Value |
|-------|-------|
| Name | `ORCLPDB1` |
| Username | `hr` |
| Password | `hr` |
| Hostname | `localhost` |
| Port | `1521` |
| Service name | `ORCLPDB1` |

---

## Part 9 — Useful Day-to-Day Commands

```bash
# Start the database service
sudo systemctl start oracledb_ORCLCDB-19c

# Stop the database service
sudo systemctl stop oracledb_ORCLCDB-19c

# Check service status
sudo systemctl status oracledb_ORCLCDB-19c

# Switch to oracle user and set environment
sudo su - oracle
source ~/.bash_profile

# Connect as SYSDBA at CDB level (for admin practice)
sqlplus / as sysdba

# Open the PDB if it's closed
sqlplus / as sysdba <<< "ALTER PLUGGABLE DATABASE ORCLPDB1 OPEN;"

# Connect to PDB as HR (for SQL practice)
sqlplus hr/hr@localhost:1521/ORCLPDB1

# Check the listener
lsnrctl status

# View the alert log
tail -f $ORACLE_BASE/diag/rdbms/orclcdb/ORCLCDB/trace/alert_ORCLCDB.log

# ADRCI — diagnostic repository CLI
adrci
```

---

## Part 10 — What You Can Practice on This Setup

| Topic Area (1Z0-082) | Practice Tasks |
|----------------------|---------------|
| Startup / Shutdown | `STARTUP`, `SHUTDOWN IMMEDIATE`, `STARTUP MOUNT`, `STARTUP RESTRICT` |
| PDB management | Open/close ORCLPDB1, create additional PDBs from seed |
| Tablespaces & datafiles | Create, resize, drop tablespaces; add datafiles; OMF |
| Users & Privileges | Common users in CDB, local users in PDB, roles, profiles |
| Undo management | Query undo tablespace, set `UNDO_RETENTION`, temp undo |
| Listener / Net Services | `lsnrctl` commands, `tnsnames.ora`, create a second listener |
| Shared server | Configure dispatchers, test shared server connections |
| RMAN | Backup ORCLCDB, practice restore, configure retention policy |
| SQL (HR schema) | All SELECT, DML, DDL, joins, subqueries, set operators |
| V$ Views | `v$database`, `v$session`, `v$parameter`, `v$tablespace` |
| Alert Log / ADR | `adrci`, `SHOW ALERT`, navigate incident packages |
| Instance parameters | `SHOW PARAMETER`, `ALTER SYSTEM`, SPFILE vs PFILE |

---

## Troubleshooting Quick Reference

| Problem | Fix |
|---------|-----|
| `ORA-12541: TNS no listener` | `lsnrctl start` |
| `ORA-01034: Oracle not available` | `sqlplus / as sysdba` → `STARTUP` |
| PDB in MOUNTED state after restart | `ALTER PLUGGABLE DATABASE ORCLPDB1 OPEN;` — then set `SAVE STATE` to persist it |
| Forgot SYS password | Boot to mount: `STARTUP MOUNT` → `ALTER USER sys IDENTIFIED BY newpwd;` → `ALTER DATABASE OPEN;` |
| Database not starting on boot | `sudo systemctl enable oracledb_ORCLCDB-19c` |
| SELinux blocking Oracle | `sudo setenforce 0` (lab only) |
| PDB not opening automatically | `ALTER PLUGGABLE DATABASE ORCLPDB1 OPEN; ALTER PLUGGABLE DATABASE ORCLPDB1 SAVE STATE;` |
| Preinstall RPM not found | `sudo dnf install -y oracle-database-preinstall-19c` — requires Oracle Linux repos active |
