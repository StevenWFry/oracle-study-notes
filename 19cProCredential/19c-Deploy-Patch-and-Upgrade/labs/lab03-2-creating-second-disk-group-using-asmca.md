# Lab 3-2 - Creating a Second Disk Group Using `ASMCA`

This lab creates the second ASM disk group for the Fast Recovery Area. The first disk group, `DATA`, was created during the Grid Infrastructure install. Now you use `ASMCA` to create `FRAASM` with the remaining disks so backups and recovery files do not have to squat in the same storage neighborhood as the primary database files.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `grid` user
- Set the environment for `+ASM`
- Start `ASMCA`
- Review the existing `DATA` disk group
- Create a second disk group named `FRAASM`
- Use disks `DISK07` through `DISK10`

---

## 2. Switch to `grid`

Open a terminal and become the `grid` user:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

---

## 3. Set the ASM Environment

Use `oraenv` and point it to the ASM instance:

```bash
. oraenv
```

When prompted, enter:

```text
+ASM
```

This ensures `ASMCA` starts in the correct Oracle environment instead of wandering off into the wrong home like an overconfident tourist.

---

## 4. Start ASMCA

Launch the ASM Configuration Assistant:

```bash
asmca
```

When the tool opens:

- Click **Disk Groups**

You should already see the existing `DATA` disk group created during the Grid Infrastructure install.

---

## 5. Create the `FRAASM` Disk Group

1. Click **Create**.
2. Enter the disk group name:

```text
FRAASM
```

3. Choose:

```text
External Redundancy
```

This lab uses external redundancy to conserve space in the classroom environment. In real life, that choice only makes sense if the storage layer beneath ASM is already doing the protection work.

---

## 6. Select the Remaining ASM Disks

On the disk selection page, choose the remaining four disks:

- `DISK07`
- `DISK08`
- `DISK09`
- `DISK10`

These disks become the new `FRAASM` disk group used later for recovery-related storage.

Click **OK** to create the disk group.

---

## 7. Verify the Result

After creation, `ASMCA` should show two disk groups:

- `DATA`
- `FRAASM`

That confirms the environment now has separate ASM storage for:

- Primary database files
- Recovery and backup-related files

Click **Exit** to close `ASMCA`.

---

## 8. What You Just Finished

By the end of this lab you:

- Set the environment for the ASM instance
- Started `ASMCA`
- Confirmed the existing `DATA` disk group
- Created the `FRAASM` disk group with `DISK07` through `DISK10`
- Finished with two ASM disk groups ready for later database and recovery configuration

Which is how the lab moves from "we have some disks" to "we have an actual storage layout with a survival instinct."
