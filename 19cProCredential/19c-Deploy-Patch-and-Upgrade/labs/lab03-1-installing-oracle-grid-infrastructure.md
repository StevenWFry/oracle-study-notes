# Lab 3-1 - Installing Oracle Grid Infrastructure

This lab installs Oracle Grid Infrastructure for a standalone server on the Oracle Linux 8.8 practice image. In plain English: you are about to install Oracle Restart and ASM, create the initial `DATA` disk group, and let the wizard ask a long series of questions it absolutely expects you to answer correctly.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Log in as `demo1`
- Switch to the `grid` user
- Unzip the staged Grid Infrastructure media
- Launch `gridSetup.sh`
- Install Oracle Grid Infrastructure for a standalone server
- Create the initial `DATA` ASM disk group
- Let `OUI` run the root scripts automatically

---

## 2. Log In and Become `grid`

1. Sign in to the desktop as `demo1`.
2. Open a terminal.
3. Switch to the `grid` account:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

---

## 3. Unzip the Grid Infrastructure Media

If the staged zip file is available in the expected lab location, unzip it as the `grid` user.

Typical pattern:

```bash
cd /stage
unzip -q LINUX.X64_12201_grid_home.zip -d /u01/app/12.2.0
```

After extraction, move to the Grid home:

```bash
cd /u01/app/12.2.0/grid
```

If your lab image uses a different staging path or zip filename, adjust accordingly. The key point is that the extracted Grid home ends up at:

```text
/u01/app/12.2.0/grid
```

---

## 4. Start Oracle Universal Installer

Launch the installer:

```bash
./gridSetup.sh
```

This starts `OUI` for the standalone Grid Infrastructure install.

---

## 5. Select the Standalone-Server Option

On the configuration option page:

- Choose **Configure Oracle Grid Infrastructure for a Standalone Server**

This is the Oracle Restart path, not the cluster path.

---

## 6. Create the Initial `DATA` Disk Group

On the ASM disk-group page:

- Change the disk discovery path to:

```text
/dev/DISK*
```

- Select the first six ASM disks
- Use disk group name:

```text
DATA
```

- Choose:

```text
Normal Redundancy
```

- Do **not** select:

```text
Configure Oracle ASM Filter Driver
```

This creates the first disk group for the database files using the first six lab disks.

---

## 7. Set ASM Administrative Passwords

Use the lab password for the ASM administrative accounts:

```text
cloud_4u
```

In this lab, the same password is used for the relevant ASM administrative accounts so nobody has to stop mid-demo and negotiate with their own memory.

---

## 8. Skip Enterprise Manager Registration

On the management options page:

- Do **not** register with Enterprise Manager
- Click **Next**

This practice environment does not use Enterprise Manager for this install.

---

## 9. Keep the OS Group Mappings

On the privileged operating system groups page:

- Keep the default mappings prepared by the earlier lab steps

That means the previously created ASM role-separation groups remain associated with the ASM privilege roles.

---

## 10. Set the Install Locations

On the install location page:

- Change the Oracle base to:

```text
/u01/app/grid
```

- Confirm the software location is:

```text
/u01/app/12.2.0/grid
```

You may see a warning that the Oracle home is outside the Oracle base. In this lab:

- Accept that warning
- Continue

It is intentional in this environment, not a spontaneous act of filesystem vandalism.

---

## 11. Keep the Inventory Location

On the inventory page:

- Keep the default inventory path

Typical value:

```text
/u01/app/oraInventory
```

The `grid` install user must be able to write to that inventory location.

---

## 12. Run the Root Scripts Automatically

On the root script execution page:

- Choose automatic execution
- Use the `root` password:

```text
Oracle
```

This lets `OUI` run the required root scripts during the install rather than making you bounce back and forth manually like a human clipboard.

---

## 13. Complete Prerequisite Checks and Install

1. Let prerequisite checks run.
2. Review the summary page.
3. Optionally save the response file.
4. Click **Install**.

Monitor the install progress.

At roughly the point where the root scripts are ready, `OUI` displays a confirmation popup.

When prompted:

- Click **Yes** to allow automatic execution of the root scripts

---

## 14. Confirm Successful Completion

At the end of the install, the finish page should report that:

- Oracle Grid Infrastructure for a standalone server was configured successfully

Click **Finish** to close the installer.

---

## 15. What You Just Installed

By the end of this lab you:

- Installed Oracle Grid Infrastructure `12.2` in the lab environment
- Chose the standalone-server configuration path
- Created the initial `DATA` ASM disk group with the first six disks
- Configured automatic execution of the required root scripts
- Completed the Oracle Restart and ASM setup successfully

Which means the storage and restart layer is now in place, and the database software is no longer waiting on infrastructure to stop being theoretical.
