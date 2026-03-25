# Lab 2-1 - Preparing Oracle Linux for Oracle Grid Infrastructure Installation

This lab walks through the host-preparation work for Grid Infrastructure. And yes, the course is labeled 19c, but the exercise intentionally installs the 12.2 preinstall package first so later patching and upgrade labs have something older to improve instead of a spotless environment with no narrative tension.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Log in to the practice environment as `demo1`
- Switch to `root`
- Install the staged 12cR2 preinstall `RPM`
- Normalize lab passwords for `root`, `oracle`, and `grid`
- Create the extra ASM groups needed for job-role separation
- Extend the `oracle` account's secondary group memberships
- Create the `grid` user and matching PAM limits file
- Create the initial software directories

---

## 2. Log In and Become `root`

1. Sign in to the lab desktop using Secure Global Desktop or TigerVNC as `demo1`.
2. Open the Terminal icon from the desktop.
3. Become `root`:

```bash
su -
```

If this is the first time you are using the environment, you may be prompted to change the initial password first. Follow the lab instructions and set the new password when required.

---

## 3. Install the Staged 12cR2 Preinstall RPM

The course uses the staged 12cR2 preinstall package to prepare the host for later upgrade work.

If the staged `RPM` directory is mounted in the expected location, run:

```bash
cd /stage/rpm
yum -y install ./oracle-database-server-12cR2-preinstall*.rpm
```

This step installs prerequisite packages and creates the base `oracle` user and primary groups used by the rest of the lab.

Note:

- The exact staged directory can vary by lab image.
- Copy and paste line by line if your remote session is being difficult. Hidden newline junk loves turning simple commands into improv theater.

---

## 4. Simplify the Lab Passwords

To keep later exercises from turning into a password-memory competition, set the lab passwords for `root` and `oracle` to `Oracle`.

```bash
passwd root
passwd oracle
```

This is obviously a lab convenience, not a production-security recommendation unless your threat model is "nobody here has eyes."

---

## 5. Create the Additional ASM Groups

The preinstall package already creates most database-side groups such as `dba`, `oper`, `backupdba`, `dgdba`, `kmdba`, and `racdba`.

Create the ASM-specific groups that are still needed:

```bash
groupadd asmdba
groupadd asmoper
groupadd asmadmin
```

These groups support ASM access, limited ASM operations, and full ASM administration.

---

## 6. Review and Extend the `oracle` Account

First, display the current group membership:

```bash
id oracle
```

Then add the secondary groups needed for this lab flow:

```bash
usermod -aG asmdba,backupdba,dgdba,kmdba,racdba oracle
```

Verify the result:

```bash
id oracle
```

The goal is for `oracle` to remain in `oinstall` and `dba`, while also gaining the extra memberships needed for ASM access, backup, Data Guard, key management, and RAC-related work.

---

## 7. Create the `grid` User

Create a dedicated `grid` user for Grid Infrastructure ownership:

```bash
useradd -g oinstall -G asmadmin,asmdba,asmoper,dba grid
id grid
```

Set the password to `Oracle` for lab convenience:

```bash
passwd grid
```

The split is intentional:

- `grid` owns and manages Grid Infrastructure
- `oracle` owns and manages the Database home

That is actual job-role separation, not just a motivational poster about it.

---

## 8. Create a PAM Limits File for `grid`

The preinstall package configures limits for `oracle`, but not automatically for the new `grid` user you just created.

Create a copy of the preinstall limits file and retarget it to `grid`:

```bash
cd /etc/security/limits.d
cp oracle-database-server-12cR2-preinstall.conf grid-database-server-12cR2-preinstall.conf
sed -i 's/^oracle/grid/' grid-database-server-12cR2-preinstall.conf
cat grid-database-server-12cR2-preinstall.conf
```

If your lab image uses a slightly different preinstall filename, copy that file instead. The important part is not the filename vanity; it is giving `grid` the same session limits as `oracle`.

---

## 9. Create the Initial Installation Directories

Create the base directories for the two Oracle homes:

```bash
mkdir -p /u01/app/grid
mkdir -p /u01/app/oracle
chown -R grid:oinstall /u01/app/grid
chown -R oracle:oinstall /u01/app/oracle
ls -ld /u01/app/grid /u01/app/oracle
```

If the staged installation media is not already readable by all required install users, adjust the permissions on that staging area as instructed in the lab image.

---

## 10. What You Just Finished

By the end of this lab you:

- Installed the staged 12cR2 preinstall package
- Standardized lab passwords
- Created ASM role-separation groups
- Expanded the `oracle` account memberships
- Created the `grid` user
- Added a PAM limits file for `grid`
- Built the initial software directory layout

In other words, the host is now much closer to something Oracle might agree to install on instead of just something Linux happened to boot.
