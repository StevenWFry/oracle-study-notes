## Lesson 10 - Patching Grid Infrastructure and Database Software (where "routine maintenance" meets the part of IT everyone schedules around)

Patching Oracle software is not mysterious, but it is absolutely a process. You identify the right patch, download the right patch, verify prerequisites, back up what matters, test it somewhere safer than production, apply it with the right tool, run the required post-patch steps, and then prove you did not turn the environment into a cautionary tale. This lesson covers that process for both Oracle Grid Infrastructure and Oracle Database.

By the end of this lesson, you should be able to:

- Describe a sane end-to-end Oracle patching process
- Distinguish reactive one-off patches from proactive bundle patching
- Explain the current 19c patch taxonomy, especially `RU` and `MRP`
- Identify the roles of `OPatch`, `OPatchAuto`, and `opatchauto -binary`
- Compare several common strategies for minimizing downtime during patching

---

## 1. The Basic Oracle Patching Process (shockingly, there is a sequence)

Regardless of environment, the broad patching flow is usually the same:

1. Identify the patch or patches you need
2. Read the patch README carefully
3. Download the patch and, if needed, the latest supported `OPatch`
4. Back up the Oracle home
5. Back up the database when database SQL changes may follow
6. Test the patch in a non-production environment
7. Resolve conflicts before touching production
8. Apply the patch to the intended targets
9. Run required post-patch steps
10. Verify that services, databases, and components are healthy

That sounds obvious until people skip steps 2 through 8 and then speak of "unexpected behavior" as though the system invented chaos unprovoked.

### Identify and obtain the right patch

The primary source for Oracle database patches is:

- **My Oracle Support (`MOS`)**

You may also use Oracle security advisories and related documentation to understand what needs attention, but the actual patch payloads and README instructions normally come through `MOS`.

### Backups before patching

Oracle explicitly recommends backing up the Oracle home before any patching operation.

And if the patch affects the database SQL layer:

- Back up the database too

Because once the binaries move and the post-patch SQL runs, rollback becomes a very different conversation.

### Test before production

If possible:

- Apply the patch on a test system first
- Check for conflicts
- Validate startup, services, listeners, and application behavior

That is not bureaucracy. That is how you avoid discovering a patch conflict during the exact maintenance window where executives are asking for progress updates every seven minutes.

---

## 2. Reactive vs Proactive Patching

Oracle patch maintenance falls into two big buckets.

### Reactive patching

Reactive patching is usually:

- A **one-off patch**
- A targeted fix for a specific bug or local issue
- Often something you install because your environment hit a real problem

This is the "fix my actual fire" branch of the patch tree.

### Proactive patching

Proactive patching is the scheduled, recommended maintenance stream for supported releases.

For current 19c patching, this means:

- **Release Updates (`RU`)**
- **Monthly Recommended Patches (`MRP`)** for Linux x86-64

Oracle recommends staying current with proactive patching because it bundles:

- Security fixes
- Regression fixes
- Other proven, low-risk corrections

This is the "please patch before the bug introduces itself personally" branch.

---

## 3. Patch Types: Historical Mess vs Current Reality

Oracle patch naming has had enough eras to qualify as geology.

### Historical patch terms you will still see in older material

Before the current `RU` / `MRP` world fully settled in, Oracle documentation and courseware often referred to:

- Interim patches
- One-off patches
- Bundle patches
- Security Patch Updates (`SPU`)
- Diagnostic patches
- System patches
- Patch Set Updates (`PSU`)
- Release Update Revisions (`RUR`)
- Merge patches / merge labels

Those terms are not all equally current, and some of them survive mostly because Oracle training material enjoys preserving fossils.

### Current 19c patch model

For Oracle Database 19c today, the main proactive patch types are:

- **Release Updates (`RU`)**
- **Monthly Recommended Patches (`MRP`)**

#### Release Updates (`RU`)

`RU`s are:

- Quarterly
- Cumulative
- Available on all supported platforms
- Oracle's primary proactive patching mechanism

Oracle's 19c docs state that `RU`s are released on the:

- **Third Tuesday of January, April, July, and October**

Not Thursday. Tuesday. Oracle was specific, and the transcript was apparently freelancing.

`RU`s commonly include:

- Security fixes
- Regression fixes
- Optimizer-related fixes
- Functional fixes and minor enhancements

#### Monthly Recommended Patches (`MRP`)

Starting with:

- **19.17**

Oracle introduced `MRP`s for:

- **Oracle Database 19c on Linux x86-64 only**

`MRP`s are:

- Monthly
- Cumulative
- Built from recommended one-off content for a given `RU`
- Intended to provide more frequent proactive maintenance between `RU`s

Important limitations:

- `MRP`s are available only on **Linux x86-64**
- They **do not change the release number**
- They are tied to the current `RU` lineage

### What happened to `RUR`s?

Oracle's 19c patch maintenance docs state that:

- `RUR`s were deprecated beginning with the October 2022 cycle
- Oracle moved to the `MRP` model starting with `19.17`

So if older slides talk lovingly about `RUR`s as a living patch stream, that love is now historically interesting rather than operationally current.

---

## 4. Patch Announcements, Security Alerts, and CVE/CVSS

Oracle patching is not just about bug numbers. Security content matters too.

Useful notification channels include:

- **My Oracle Support patch notes**
- **Oracle Critical Patch Updates and Security Alerts**
- Related `MOS` notes such as the proactive patch program notes

Oracle's public security advisories use:

- **CVE** identifiers
- **CVSS** scores

These are useful because they help you judge:

- Which vulnerabilities are remotely exploitable
- Whether authentication is required
- Which supported versions are affected
- How severe the risk is

In other words, the vulnerability matrix is not decorative. It is one of the few places where Oracle hands you a prioritized list of what should scare you first.

---

## 5. `OPatch` (the fundamental tool, and the one that expects you to think)

`OPatch` is Oracle's Java-based patch utility for applying and rolling back patches in an Oracle home.

Use `OPatch` when you want to perform:

- Binary patching on a specific Oracle home
- Manual patch application
- Rollback of a patch
- Inventory inspection

### Key operational points

- `OPatch` is usually located in:

```text
$ORACLE_HOME/OPatch
```

- It is typically run by the **owner of the Oracle home being patched**
- It can be added to `PATH`, or called with its full path

Oracle recommends using the latest supported `OPatch` for the Oracle home release you are patching.

### Common things `OPatch` can do

- `lsinventory`
- `apply`
- `rollback`
- `version`
- `prereq`
- `help`

It also supports:

- `-report`
  - Show the actions without executing them

- `-verbose`
  - Print more detail to the screen and logs

The `-report` option is one of the more civilized features in the Oracle toolchain because it lets you preview the damage before committing to it.

### Logs

`OPatch` logs are typically written under:

```text
$ORACLE_HOME/cfgtoollogs/opatch
```

Which is where you go when the patch claims it had a problem and declines to explain itself in the terminal like an adult.

### Important post-patch note for database homes

If you patch an **RDBMS** home with plain `OPatch`, then any required SQL changes in the database are typically completed with:

- **`datapatch`**

That is not optional when the patch README says SQL actions are required. "The binaries were updated" is not the same thing as "the database patch is actually complete."

---

## 6. `OPatchAuto` (the orchestration tool with fewer opportunities for human improv)

`OPatchAuto` is Oracle's orchestration layer over `OPatch`.

It generates target-specific instructions and can automate patching tasks such as:

- Pre-patch checks
- Stopping services
- Applying the patch
- Restarting services
- Post-patch checks
- Rollback when required

For Grid Infrastructure environments, Oracle documents that `OPatchAuto` is run:

- From the **Grid Infrastructure home**
- As the **`root`** user

This is the tool you use when you want Oracle to manage more of the sequence instead of manually typing your way into avoidable embarrassment.

### Core `OPatchAuto` commands

The main commands include:

- `apply`
- `resume`
- `rollback`
- `version`
- `help`

One especially useful option is:

- `-analyze`

This lets you verify prerequisites before trying to actually apply the patch. Which is much better than discovering a missing prereq halfway through a maintenance window while everyone develops that specific outage-call voice.

### `datapatch` behavior

In modern Oracle maintenance workflows:

- `OPatchAuto` can invoke the required post-patch SQL automation for the database
- Plain `OPatch` generally requires you to run `datapatch` manually

That difference matters. A lot.

---

## 7. `opatchauto -binary` (binary-only, one home, no hand-holding)

`opatchauto -binary` is the narrower, binary-only variant.

It is used to:

- Apply one or more patches to **one selected Oracle home per session**

Its characteristics include:

- Assumes targets are already shut down
- Applies only the binary bits
- Performs prerequisite checks when run with analyze/report-style options
- Does not perform the broader orchestration that full `OPatchAuto` handles

So this is the "just patch the home" tool, not the "please manage the whole environment for me" tool.

---

## 8. Patching Strategy: Minimize Downtime Without Creating New Problems

How you reduce downtime depends on the architecture.

### RAC rolling patching

In an Oracle `RAC` environment, rolling patching can patch one node at a time while the other nodes continue serving work.

That can provide near-zero or reduced downtime, but only when:

- The patch supports rolling application
- The environment is designed for it
- The README explicitly allows it

Read that last one twice. Not every patch is rolling just because your cluster desperately wants it to be.

### Data Guard standby-first patching

With Oracle Data Guard, a common low-downtime strategy is:

1. Patch the standby database home first
2. Test and validate it there
3. Switchover if desired
4. Patch the former primary

Oracle documents this as **Standby-First Patch Apply** for eligible patches.

Important caveat:

- Only patches certified as **Data Guard Standby-First Installable** qualify

So you do not just assume. You verify it in the patch README like a responsible adult with a change window to protect.

### Out-of-place / cloned-home patching

Another downtime-reduction technique is:

- Clone the Oracle home
- Patch the clone
- Switch the database or service over to the patched home

Oracle's current 19c patch maintenance docs increasingly steer customers toward **out-of-place patching** for proactive maintenance, which makes sense because "patch the copy, then cut over" is far less stressful than modifying the only working home while production is still emotionally attached to it.

This reduces downtime because the slow patching work happens before the final cutover.

The database is only down for:

- Shutdown
- Home switch
- Restart
- Required post-patch steps

Which is a much nicer outage story than "we took production down and then started unzipping things."

---

## 9. Practical Patching Checklist

Before patching Grid Infrastructure or an Oracle Database home, confirm:

- You identified the correct patch and read the README
- You downloaded the latest required `OPatch`
- You backed up the Oracle home
- You backed up the database if SQL changes may follow
- You know whether the patch is rolling, non-rolling, standby-first, or out-of-place capable
- You tested the patch in a lower environment if possible
- You know whether the tool is `OPatch`, `OPatchAuto`, or `opatchauto -binary`
- You know whether `datapatch` must be run after the binary patching step

If any of those answers are fuzzy, then the patch window is probably running on hope, which is not a supported Oracle utility.

---

## 10. Wrap-Up (patch now, explain less later)

This lesson covered the Oracle patching process, the current 19c patch types, the main patching tools, and several ways to reduce downtime when patching Grid Infrastructure and database software. Next comes upgrading, where the stakes get higher, the steps get longer, and the documentation starts looking like it expects you to already be worried.
