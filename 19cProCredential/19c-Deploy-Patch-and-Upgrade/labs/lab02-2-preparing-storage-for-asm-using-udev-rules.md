# Lab 2-2 - Preparing Storage for ASM Using `udev` Rules

This lab verifies that the operating-system storage for ASM is presented with stable names, ownership, and permissions. The course image already includes the `udev` rules, so the job here is mostly review and refresh rather than writing the rules from scratch like some kind of block-device poet.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Log in as `demo1`
- Switch to `root`
- Review the precreated `udev` rules
- Verify the ASM device mappings
- Reload the `udev` rules
- Confirm that the disks end up owned by `grid:asmadmin`

---

## 2. Log In and Become `root`

1. Sign in to the desktop as `demo1` using Secure Global Desktop or TigerVNC.
2. Open a terminal.
3. Become `root`:

```bash
su -
```

---

## 3. Review the Existing `udev` Rules

The class setup already created the rules for the ASM disks, so start by reviewing them.

Typical location:

```bash
cat /etc/udev/rules.d/99-oracle-asmdevices.rules
```

Expected result:

- Entries for `DISK01` through `DISK10`
- Ownership targeting `grid`
- Group targeting `asmadmin`

If your lab image uses a different rules filename, review that file instead. The point is to confirm the mappings, not to worship one exact pathname.

---

## 4. Verify the ASM Device Mappings

Display the ASM disk aliases created by the rules:

```bash
ls -l /dev/DISK*
```

Then inspect the underlying block devices:

```bash
ls -l /dev/xvd*
```

You should see the `DISK01` through `DISK10` names mapped to block devices such as `/dev/xvda5` through `/dev/xvda14`.

Note:

- The transcript called out a typo in the original slide. If you see a command that looks like `ls -l /dev/xvda` when the intent is clearly to review multiple devices, use the broader `xvd*` form instead of trusting the typo with your whole heart.

---

## 5. Reload the `udev` Rules

Reload the rules and trigger device processing:

```bash
udevadm control --reload-rules
udevadm trigger
```

This forces Linux to reapply the rule set so the expected names, ownership, and group assignments are refreshed.

---

## 6. Recheck Ownership and Group

Run the alias listing again:

```bash
ls -l /dev/DISK*
```

Expected result:

- Owner: `grid`
- Group: `asmadmin`

If the ownership does not look correct immediately, wait a few seconds and rerun the command. The course demonstration noted that the expected values can take a moment to settle in the lab environment.

---

## 7. What You Just Verified

By the end of this lab you:

- Confirmed the class-provided `udev` rules exist
- Verified the alias mapping from `DISK01`-`DISK10` to the underlying block devices
- Reloaded the rules
- Confirmed the ASM devices resolve to the correct owner and group

This is the part where storage goes from "I think Linux can see some disks" to "Oracle can actually use them without a reboot turning everything into folklore."
