## Lesson 14 - Upgrading Oracle Database with DBUA and Manual Methods (where the wizard still exists and the manual path still threatens your weekend)

`AutoUpgrade` may be Oracle's recommended answer, but Oracle still ships other upgrade paths for people who either prefer them, inherited them, or got cornered by one of the weird edge cases that enterprise systems grow when nobody is looking. This lesson covers the remaining two database-upgrade approaches: **Database Upgrade Assistant (`DBUA`)** and the **manual upgrade path** using `catctl.pl`. Both can work. One is older and GUI-driven. The other is a direct invitation to manage every sharp edge yourself.

By the end of this lesson, you should be able to:

- Explain how `DBUA` performs an Oracle Database upgrade
- Describe when `DBUA` can be used in GUI mode versus silent mode
- Summarize the main pages and decisions in the `DBUA` workflow
- Explain the manual-upgrade workflow using `catctl.pl`
- Identify the extra cleanup and validation work manual upgrades require

---

## 1. `DBUA` in the Upgrade Story

`DBUA`, the **Database Upgrade Assistant**, is Oracle's older upgrade utility with a graphical interface.

It can:

- upgrade non-`CDB` databases
- upgrade `CDB` databases
- upgrade `PDB` databases within a container architecture
- run in **GUI mode**
- run in **silent mode**

Oracle still supports it in `19c`, but Oracle's own guidance is clear:

- prefer `AutoUpgrade` when possible

That does not mean `DBUA` is useless. It means `DBUA` is no longer Oracle's favorite child.

---

## 2. Why People Still Use DBUA

The main appeal of `DBUA` is simple:

- it gives you a visible workflow
- it walks through the upgrade page by page
- it performs many steps for you

This makes it comfortable for:

- single-database upgrades
- administrators who want a GUI review step
- environments where seeing the upgrade choices in a wizard is useful for validation

It also supports:

- command-line silent mode

So if someone wants the `DBUA` engine but not the GUI, Oracle still gives them that option.

---

## 3. Starting DBUA from the New Home

Just like `AutoUpgrade`, `DBUA` is run from the:

- **new Oracle home**

That matters because the whole upgrade is still:

- **out of place**

So the sequence is:

1. install and patch the new `19c` home
2. start `DBUA` from that new home
3. let `DBUA` discover the older database
4. perform the upgrade into the new home

In other words, if you launch `DBUA` from the old home and hope it transcends chronology, it will not.

---

## 4. What DBUA Detects and Asks For

When you start `DBUA`, it detects databases that are candidates for upgrade from the older home.

The normal workflow includes:

- selecting the target database if more than one is shown
- providing `SYSDBA` credentials
- confirming the source and target homes

If there are multiple databases on the host:

- use the filter if needed
- make sure you are upgrading the correct one

This sounds painfully obvious until somebody upgrades the wrong database and then starts talking about "environment naming issues" as if Oracle did that on purpose.

---

## 5. PDBs, Open Mode, and Multitenant Checks

When `DBUA` upgrades a container database:

- it checks the `PDB`s
- it shows which ones are open
- it shows their open mode

Operationally, the important rule is:

- `PDB`s that you want upgraded need to be available in **READ WRITE** mode

If a `PDB` is closed or not open for upgrade participation, then it is not part of the same happy story as the rest of the container.

This is the same practical concern the course called out for `AutoUpgrade`: if the pluggable databases are not open and ready, then Oracle cannot upgrade what you chose not to present.

---

## 6. DBUA Prechecks and Fixups

`DBUA` performs prerequisite validation before the upgrade.

That includes checks for:

- configuration readiness
- invalid conditions
- required fixups
- upgrade blockers

If problems are found, `DBUA` can often offer:

- **Fixups**
- **Check Again**

Which lets the tool apply what it can and then rerun the checks.

That is one of `DBUA`'s better traits. It does not just complain. It tries, within limits, to help clean up the mess first.

---

## 7. Upgrade Options in DBUA

Once prechecks pass, `DBUA` lets you configure additional upgrade behavior.

The transcript highlights options such as:

- parallel upgrade behavior
- recompilation of invalid objects
- time zone file upgrade
- pre-upgrade custom scripts
- post-upgrade custom scripts

The silent-mode syntax Oracle documents confirms these capabilities as real options, not just transcript chaos.

So if your application needs:

- preparatory SQL
- cleanup SQL
- custom hooks around the upgrade

`DBUA` can participate in that workflow.

---

## 8. Backup, Restore Point, Listener, and Management Choices

One place where `DBUA` can feel more guided than `AutoUpgrade` is that it exposes more of the surrounding choices directly in the flow.

The transcript calls out choices such as:

- whether to create a backup
- whether to create or use a guaranteed restore point
- listener and network settings
- whether to configure or reuse listeners in the new home
- Enterprise Manager Database Express options
- Cloud Control registration

Oracle's documented silent-mode flags support those themes too, including:

- backup location
- restore-point use
- listener creation
- `EM Express` and `Cloud Control` options

So yes, `DBUA` is not just "click Next until the dictionary changes." It is also a structured conversation about the environment around the upgrade.

---

## 9. Summary, Execution, and Results

Near the end of the `DBUA` workflow:

- you review a summary page
- you make any last edits
- you click **Finish**

Then `DBUA`:

- starts the upgrade
- runs the required scripts
- tracks progress
- writes detailed logs
- produces an upgrade result summary

Oracle also saves an HTML version of the results in the log area, which is useful because no upgrade is truly complete until someone stares at a report and tries to decide whether "warning" means "fine" or "we should sit down."

Once the upgrade completes:

- close `DBUA`
- then continue with the required post-upgrade tasks

And yes, there are still post-upgrade tasks. Oracle is nothing if not consistent.

---

## 10. DBUA Silent Mode

If you do not want the GUI, `DBUA` supports:

- `-silent`

Typical entry point:

```bash
dbua -silent -sid ORCL
```

Oracle documents a very long option list for silent mode, including:

- precheck execution
- fixups
- backup location
- restore-point behavior
- listener handling
- pre and post scripts
- invalid-object recompilation
- time zone upgrade
- `EM` integration

So silent-mode `DBUA` is real. It is just less flexible and less favored than `AutoUpgrade`.

---

## 11. Manual Upgrade (the path where you do the remembering personally)

If you perform a manual upgrade, then you are choosing the path with:

- the most control
- the most manual sequencing
- the most chances to miss one ugly little detail

The high-level flow looks like this:

1. complete the pre-upgrade checks and any fixups
2. back up the database
3. install the new Oracle home
4. shut down the database from the old home
5. prepare a parameter file for the new home
6. remove deprecated parameters and adjust required ones
7. start the database from the new home in **upgrade mode**
8. run `catctl.pl`
9. open the database normally afterward
10. run post-upgrade scripts and recompilation
11. validate time zone, home-local files, and network files

This is the approach for people who truly want to touch every piece of the machinery. Oracle is happy to let you do that, mostly because the logs will later prove it was your idea.

---

## 12. The Role of `catctl.pl`

The centerpiece of the manual path is:

- `catctl.pl`

This is Oracle's **Parallel Upgrade Utility**, and it drives the database-upgrade SQL scripts in the correct sequence.

It does the heavy lifting that people used to attempt through long chains of manual script execution, but you still remain responsible for the surrounding orchestration:

- startup mode
- parameter cleanup
- post-upgrade work
- validation

So `catctl.pl` is not a full upgrade framework in the way `AutoUpgrade` is. It is the engine, not the chauffeur.

---

## 13. Manual Parameter and Home Cleanup

Manual upgrade means you also inherit manual cleanup duties.

That includes things like:

- creating a `PFILE` from the old `SPFILE` if needed
- removing deprecated initialization parameters
- adding or adjusting required parameters
- moving or recreating the parameter file in the new home context
- copying or recreating home-local files such as:
  - password file
  - `tnsnames.ora`
  - `sqlnet.ora`
  - local network files

This is exactly why the manual path is so prone to errors. Oracle-owned scripts can upgrade the dictionary, but they cannot save you from forgetting that your old home contained half the environment's identity paperwork.

---

## 14. Post-Upgrade Work After Manual Upgrade

After the manual upgrade completes, you still need to do the normal cleanup and validation work.

Typical tasks include:

- open the database read/write
- run `utlrp.sql` to recompile invalid objects
- run any post-upgrade fixups that were identified
- review deprecated and obsolete parameter cleanup
- validate time zone handling
- review logs
- validate network and password-file behavior in the new home

The transcript also calls out:

- `catcon.pl`
- time zone packages and validation steps

Those belong in the post-upgrade cleanup layer, not in the "great, the upgrade script returned control to the shell" fantasy layer.

---

## 15. Which Path to Use

If you are making the decision today:

- use **`AutoUpgrade`** unless you have a compelling reason not to
- use **`DBUA`** when a GUI-driven, more guided flow genuinely helps
- use **manual upgrade** only when you specifically need that level of control or troubleshooting depth

Oracle's documentation is not subtle about this anymore. `DBUA` and manual upgrade still exist, but Oracle is steering people away from both.

Which is fair. If Oracle can automate a fragile process more reliably than the average sleep-deprived human change window, it should.

---

## 16. Wrap-Up (yes, the manual path works, no, that is not a recommendation)

This lesson covered the remaining database-upgrade methods after `AutoUpgrade`: how `DBUA` works in GUI and silent mode, what decisions it walks through, how it handles prechecks, backup and listener options, and what the manual `catctl.pl` path requires before, during, and after the upgrade. Next comes the post-upgrade chapter, where Oracle reminds you that "upgrade complete" and "environment ready" are not synonyms.
