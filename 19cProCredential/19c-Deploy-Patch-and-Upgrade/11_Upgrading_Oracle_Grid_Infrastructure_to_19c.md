## Lesson 11 - Upgrading Oracle Grid Infrastructure to 19c (where the new home is patched, the old home is touchy, and everybody wants root)

Upgrading Oracle Grid Infrastructure to `19c` is not an in-place makeover. It is an **out-of-place** move into a new, already-prepared home, followed by an upgrade process that rewires Oracle Restart, `ASM`, and the surrounding metadata to trust the new location. This lesson focuses on upgrading a standalone Grid Infrastructure deployment to `19c` by using `Oracle Universal Installer (OUI)`, which is Oracle's recommended path and, by Oracle standards, the version least likely to punish you for optimism.

By the end of this lesson, you should be able to:

- Explain the supported upgrade path for Oracle Grid Infrastructure / Oracle Restart to `19c`
- Describe the major restrictions and safety rules for a Grid Infrastructure upgrade
- Prepare the operating system, users, and environment for the upgrade
- Explain why databases using `ASM` must be shut down first
- Walk through the `OUI`-based upgrade flow from the new `19c` home
- Validate that the upgraded Grid Infrastructure stack is running from the new release

---

## 1. The Big Rule: This Is an Out-of-Place Upgrade

For Oracle Restart / standalone Grid Infrastructure upgrades to `19c`:

- The upgrade is **always out of place**
- You do **not** upgrade the existing Grid home in place
- You install or unzip a **new** `19c` Grid home
- You run the upgrade from that new home

That is not Oracle being difficult for sport. The upgrade needs a separate home so the old software remains available until the new stack is fully wired in and ready to take over.

In practice, the preferred pattern is:

1. Build the new `19c` home first
2. Patch it to the target `RU` before upgrade
3. Use that patched home to perform the upgrade

In other words, do not show up to the upgrade window with a fresh, unpatched home and a dream.

---

## 2. Supported Direct Upgrade Paths to 19c

For Oracle Restart / standalone Grid Infrastructure, Oracle supports direct upgrade to `19c` from:

- `11.2.0.4`
- `12.1.0.2`
- `12.2`
- `18c`

That means:

- If you are older than `11.2.0.4`, you do **not** jump straight to `19c`
- You first move to a supported intermediate release
- Then you upgrade again to `19c`

So yes, some estates must suffer a two-step journey before they are allowed into the present century.

---

## 3. Restrictions and Safety Rules (the part Oracle really means)

Before upgrading Grid Infrastructure to `19c`, keep these rules in view:

- Use the **same Grid Infrastructure owner** that owned the earlier release
- Make sure the required administrative privileges are available
- Do **not** delete directories from the old Grid home before the upgrade is complete
- Do **not** treat the new `19c` Grid home as fully usable until the upgrade finishes
- Shut down databases that use the old `ASM` instance before upgrading `ASM`

Two especially important operational points:

### Do not delete the old Grid home too early

During the upgrade, Oracle still relies on information from the existing Grid Infrastructure setup.

So before validation is complete:

- do not remove old directories
- do not clean up "unused" files
- do not let impatience cosplay as housekeeping

Only after the upgrade is complete, validated, and in use should you consider deinstalling or removing the earlier home.

### The new home is not truly live until the upgrade completes

Until the upgrade finishes:

- `srvctl`
- `crsctl`
- and related tools from the **new** home

are not the authoritative truth yet.

Oracle does not consider the new Grid Infrastructure home fully functional until the upgrade scripts complete and the configuration pointers have been updated. Until then, the new home is more of a candidate than a government.

---

## 4. Environment Hygiene Before You Start

One of the easiest ways to sabotage an upgrade is to let commands or the installer drift into the wrong home because your shell still remembers yesterday.

Before starting the upgrade:

- unset `ORACLE_HOME`
- unset `ORACLE_BASE`
- unset `ORACLE_SID`
- unset `TNS_ADMIN`
- unset `NLS_LANG`
- review the user's shell profile for stale Oracle environment settings
- make sure `PATH` does not point to old Oracle home `bin` directories

Why this matters:

- The upgrade is started from the **new** `19c` home
- The installer discovers older-home details as part of the process
- You do not want stray session variables redirecting tools into the wrong release

This is one of those steps that feels fussy right up until the moment the wrong binary gets called and the maintenance window becomes a courtroom drama.

---

## 5. OS and User Preparation for the Upgrade

The transcript reuses the same basic pattern from installation prep:

- install the appropriate `19c` preinstallation package or `RPM`
- confirm the OS users and groups meet `19c` expectations
- refresh the `limits.d` configuration for both `oracle` and `grid`

The important idea is not that the `RPM` is magical. It is that it updates:

- required packages
- user and group expectations
- security limits and profile settings

For this course's lab flow, the `19c` preinstall package creates the `oracle` limits file, and then that file is copied and adapted for the `grid` user so both software owners satisfy the `19c` expectations.

That is practical, boring, and exactly the sort of thing that prevents avoidable upgrade failures.

---

## 6. Shut Down Databases That Depend on ASM

If the existing databases use the `ASM` instance managed by the earlier Grid Infrastructure home, then those databases must be shut down before the Grid Infrastructure upgrade proceeds.

Why:

- `ASM` is upgraded as part of the Grid Infrastructure upgrade
- databases actively using that `ASM` instance must not keep writing during the transition
- Oracle wants the storage layer quiet before it starts rearranging its furniture

So before you click through the installer:

- stop `ORCL`
- stop any additional databases such as the classroom `CDB122`
- confirm they are down

This is not optional caution. This is how you avoid mixing a storage upgrade with active database I/O and then acting surprised when Oracle objects.

---

## 7. Use a Patched 19c Home, Ideally from a Gold Image

The course deliberately builds the future Grid Infrastructure home in advance by:

- taking the base `19.3` software
- patching it to the classroom target level, such as `19.20`
- creating a **gold image**

Then the upgrade uses that prepatched home instead of a raw base release.

That is the right instinct because it means:

- fewer immediate post-upgrade patching chores
- a more repeatable deployment
- less time spent explaining why the new home was already outdated on arrival

So when the lesson says "use the new home," the sane interpretation is:

- use the already-patched `19c` home you prepared earlier

---

## 8. Starting the Upgrade with OUI

From the new `19c` Grid home:

- unzip or stage the software into the target home
- run `gridSetup.sh`

Because you started from a newer Grid Infrastructure home and Oracle detects an earlier release on the server, the installer preselects:

- **Upgrade Oracle Grid Infrastructure**

For a standalone-server environment, this is the expected and welcome behavior. It means the installer understands the assignment for once.

---

## 9. What You Validate in the Upgrade Wizard

During the `OUI` flow, you review and confirm several items.

### Upgrade target and ASM warning

The installer detects the existing standalone Grid Infrastructure deployment and prepares to upgrade it.

If databases using `ASM` are still running, Oracle warns you. If you already shut them down, the installer is much less theatrical.

### Enterprise Manager / Cloud Control registration

If your environment uses Cloud Control:

- validate or supply the registration details

If not:

- leave it unregistered and continue

### Privileged operating system groups

Validate the OS groups mapped to the relevant privileges for the Grid Infrastructure owner.

This is where the earlier user/group preparation becomes installer input instead of theory.

### Oracle base and new home

Validate:

- the existing Oracle base used by the Grid owner
- the new `19c` Grid home path

The new home path should already be the directory from which you launched `gridSetup.sh`, so this screen is mostly a sanity check against your own typing.

### Root script execution method

Choose whether privileged scripts will run:

- automatically with supplied credentials
- automatically through `sudo`
- or manually

Oracle recommends automation here, and for the classroom flow that is the cleanest choice because it allows the installer to call the required upgrade scripts when the moment arrives.

### Prerequisite checks

Let the installer run the full prerequisite validation.

If issues appear:

- fix what must be fixed
- understand which items are warnings versus hard failures
- do not wave them away just because the maintenance window already has emotional momentum

---

## 10. The Root-Script Moment

Partway through the upgrade, typically around the early visible portion of progress, Oracle reaches the point where privileged scripts must run.

If you configured automatic execution:

- a popup asks for confirmation
- you approve it
- Oracle runs the scripts for you

If you did **not** configure automatic execution:

- the popup tells you which scripts to run manually
- you execute them as `root`
- then return to the installer and continue

This is one of the most important handoff points in the whole upgrade. If this step goes sideways, the rest of the upgrade has no reason to keep pretending things are fine.

---

## 11. What Success Looks Like

When the upgrade completes successfully:

- the installer reports success
- Oracle Restart is now based on the `19c` Grid Infrastructure home
- the upgraded `ASM` stack is available
- the local high-availability services are running from the new release

At that point you can:

- close the installer
- validate the Grid Infrastructure version
- restart the databases that were shut down for the upgrade

Typical validation includes checking the Oracle Restart / Grid Infrastructure release from the new home, for example with:

```bash
crsctl query has releaseversion
```

Depending on what you want to verify, you may also use:

```bash
crsctl check has
srvctl status asm
```

The important thing is not the ritual itself. The important thing is proving the new stack is actually the one in charge.

---

## 12. Practical Upgrade Checklist

Before upgrading Oracle Grid Infrastructure / Oracle Restart to `19c`, confirm:

- the source release is on a supported direct-upgrade path
- the target is a **new** `19c` Grid home
- that new home is already patched to the intended `RU`
- the `grid` owner has the required privileges
- stale Oracle environment variables are unset
- `oracle` and `grid` limits/config files reflect the `19c` requirements
- databases using `ASM` have been stopped
- you know whether root scripts will run automatically or manually
- you are not deleting the old Grid home before validation is complete

If any of those items are fuzzy, then the upgrade window is being run on vibes, which is not one of Oracle's documented upgrade methods.

---

## 13. Wrap-Up (new home, same responsibility, slightly newer panic)

This lesson covered the `19c` Oracle Grid Infrastructure upgrade path for a standalone environment: the out-of-place rule, supported source releases, environment cleanup, OS preparation, shutting down databases that depend on `ASM`, using a patched new home, walking through the `OUI` upgrade flow, handling root scripts, and validating the new release afterward. Next comes the database upgrade itself, where the storage stack is now modernized and the actual data dictionary gets its turn under the lamp.
