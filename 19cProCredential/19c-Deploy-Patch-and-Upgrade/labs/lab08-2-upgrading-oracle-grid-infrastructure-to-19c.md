# Lab 8-2 - Upgrading Oracle Grid Infrastructure to 19c

This lab upgrades the standalone Grid Infrastructure home to `19c` by using the prepatched gold image prepared earlier. The work is mostly driven through `OUI`, which is Oracle's recommended route and, frankly, the option least likely to turn the exercise into a manual archaeology project.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Switch to the `grid` user
- Unzip the patched `19c` Grid gold image into the new home
- Start `gridSetup.sh`
- Accept the detected upgrade path
- Validate the Oracle base and new Grid home
- Let the installer run the privileged scripts automatically
- Complete the upgrade to `19c`
- Verify the upgraded Grid Infrastructure release

---

## 2. Assumptions

This lab assumes:

- Lab 8-1 is complete
- The earlier `12.2` Grid Infrastructure home is still intact
- The databases that use `ASM` are stopped
- A patched `19c` Grid gold image already exists from the patching exercises
- The new Grid home directory already exists and is owned by `grid:oinstall`

The transcript refers to the prepatched home as the `19.20` home created from the earlier gold image exercise. This lab keeps that logic but uses a placeholder for the exact gold-image zip filename.

---

## 3. Switch to the `grid` User

Open a terminal and become the `grid` user:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

Change to the new `19c` Grid home:

```bash
cd /u01/app/19c/grid
```

---

## 4. Unzip the Patched Grid Gold Image

Unzip the staged Grid gold image into the new home:

```bash
unzip -q /u01/stage/gold-images/<grid_gold_image_zip>
```

When extraction finishes, confirm the home contents exist:

```bash
ls
```

This is the prepatched home you built in the patching chapter. Which is convenient, because upgrading into an already-patched home is much less absurd than upgrading into a stale one and then immediately patching it again.

---

## 5. Launch the Grid Infrastructure Installer

From the new home, start the installer:

```bash
./gridSetup.sh
```

Because an older standalone Grid Infrastructure release is already present, the installer should detect that and preselect:

- **Upgrade Oracle Grid Infrastructure**

Click **Next**.

If the installer detects running databases still using `ASM`, it warns you. In the classroom flow that warning does not appear because the databases were already stopped.

---

## 6. Walk Through the Upgrade Screens

On the installer pages:

1. Leave Cloud Control registration disabled unless your classroom image actually uses it
2. Confirm the Oracle base remains:

```text
/u01/app/grid
```

3. Confirm the new Grid home is the `19c` home you launched from, for example:

```text
/u01/app/19c/grid
```

4. Choose automatic execution of the required root scripts
5. Provide the `root` password:

```text
Oracle
```

6. Let the prerequisite checks complete

If the checks pass, submit the upgrade job and let the installer proceed.

---

## 7. Approve the Root Script Execution

Partway through the upgrade, the installer prompts that privileged scripts need to run.

Because this lab uses automatic execution:

- click **Yes** when the popup appears

If your environment does not use automatic execution, then run the displayed scripts manually as `root` and return to the installer afterward.

The demo notes that the popup appears after the upgrade has made some visible progress. So if nothing happens for a while, that is Oracle being Oracle, not necessarily a crisis.

---

## 8. Finish the Upgrade

When the installer reports success:

- close the installer window

Now set the environment for the upgraded Grid home:

```bash
export ORACLE_HOME=/u01/app/19c/grid
export PATH=$ORACLE_HOME/bin:$PATH
```

If you want the `ASM` environment in the shell, set it as well:

```bash
export ORACLE_SID=+ASM
```

Verify the upgraded release:

```bash
crsctl query has releaseversion
```

You should now see the `19c` Grid Infrastructure release.

Optional extra validation:

```bash
crsctl check has
srvctl status asm
```

---

## 9. Optional Follow-Up

The lesson notes that after the Grid Infrastructure upgrade completes successfully, you can start the databases that were shut down before the upgrade.

If your next exercise expects them online, restart them from the existing database home as the `oracle` user:

```bash
srvctl start database -db orcl
srvctl start database -db cdb122
```

If the next lab begins with the databases down, leave them that way and follow the later instructions instead.

---

## 10. What You Just Finished

By the end of this lab you:

- unpacked the patched `19c` Grid gold image
- launched `gridSetup.sh` from the new home
- let `OUI` upgrade the standalone Grid Infrastructure
- ran the privileged upgrade scripts through installer automation
- verified that Oracle Restart / Grid Infrastructure now reports the `19c` release

Which means the local high-availability stack has moved into its new home, `ASM` came along for the ride, and the old `12.2` home can now begin its long journey toward respectful retirement instead of immediate deletion.
