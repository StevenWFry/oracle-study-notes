## Lesson 12 - Choosing Oracle Database Upgrade Methods and Data Migration (where "just upgrade it" becomes a small pile of strategy decisions)

Upgrading the Oracle **database** is not the same thing as upgrading Grid Infrastructure, and it is definitely not the same thing as migrating data into a new database and calling it a day. This lesson is about choosing the right path before the upgrade starts: direct upgrade versus migration, `AutoUpgrade` versus `DBUA` versus manual upgrade, and when `Data Pump` is the cleaner answer because the source release, platform, or character set has other plans for your weekend.

By the end of this lesson, you should be able to:

- Distinguish a database **upgrade** from a **data migration**
- Compare the main Oracle Database 19c upgrade methods
- Explain why Oracle recommends `AutoUpgrade`
- Identify when `DBUA`, manual upgrade, or `Data Pump` migration may still be used
- Recognize the direct-upgrade source releases for Oracle Database 19c
- Summarize the planning steps to complete before the actual database upgrade begins

---

## 1. Upgrade vs. Migration (same destination, very different pain profile)

Before choosing a method, separate two ideas that people constantly blur together.

### Database upgrade

A database **upgrade** means:

- You install or prepare a **new Oracle home**
- You switch the database to that new software home
- You upgrade the **data dictionary**, metadata, and Oracle-owned components
- You recompile objects and perform required post-upgrade adjustments

What it does **not** mean:

- It does not directly change user data just for sport
- It does not create a brand-new database and repour the contents into it

This is the "same database, new software generation, upgraded metadata" path.

### Data migration

A **data migration** means:

- You install a new Oracle home
- You create a **new target database**
- You move metadata and data from the old database into the new one

Usually, Oracle recommends:

- `Oracle Data Pump` export/import

This is not a dictionary-upgrade path. It is a **new database build plus data movement** path.

So if an upgrade is replacing the engine while the car stays the same car, migration is buying a new car and then moving all the boxes into it while hoping nobody forgot the title.

---

## 2. Why the Difference Matters

These two paths solve different problems.

### Direct upgrade is usually best when:

- the source release is directly supported
- the platform stays the same
- the character set stays the same
- you want Oracle to preserve the existing database identity and structure

### Migration is often better when:

- the source release is too old for direct upgrade
- you are changing operating system or hardware platform
- you want to change the character set
- you want a clean rebuild of storage structures
- you want to move from a non-`CDB` into a `PDB`

That last point matters because "upgrade" is sometimes technically possible but operationally ugly. Migration is slower, but it can be much more flexible.

---

## 3. The Oracle-Recommended Method: `AutoUpgrade`

For Oracle Database 19c, Oracle recommends:

- **`AutoUpgrade`**

Oracle's 19c upgrade docs are explicit here. `AutoUpgrade` is the preferred method, and Oracle has deprecated older approaches like `DBUA` and manual upgrades in 19c so it can focus support and engineering effort on the tool it actually wants you to use.

### What `AutoUpgrade` does

`AutoUpgrade` can:

- analyze the source database before upgrade
- identify issues
- run fixups where possible
- deploy the upgrade
- run many post-upgrade steps
- restart the upgraded database

It works from a **configuration file**, which is one of its biggest advantages. Instead of clicking through a wizard and hoping you remember every choice later, you define what you want in a repeatable file and let Oracle do the choreography.

### Why Oracle likes it

`AutoUpgrade` is good at:

- automating repetitive work
- handling multiple databases from one configuration
- reducing manual mistakes
- producing upgrade logs and reports

It can also be used for:

- **out-of-place patching**

So yes, Oracle built a tool that tries very hard to remove the human from the most failure-prone parts of the process. This is growth.

### Standard Edition caveat

`AutoUpgrade` is available for:

- Enterprise Edition
- Standard Edition / Standard Edition 2

But there is an important fallback limitation for Standard Edition:

- Standard Edition does **not** support `Flashback Database`
- therefore the `AutoUpgrade` restoration path using guaranteed restore points is not available the same way

Which means Standard Edition users need a more traditional fallback plan instead of assuming Oracle will rewind the mistake automatically.

---

## 4. `DBUA` (the older GUI path, still here, less favored)

`DBUA`, the **Database Upgrade Assistant**, is the traditional graphical tool for upgrading a database.

It can:

- guide you through the upgrade with a GUI
- perform prerequisite checks
- run many upgrade scripts automatically
- generate logs and HTML reports
- handle some pre-upgrade and post-upgrade tasks

Oracle still supports it in 19c, but Oracle no longer wants it to be your first instinct.

### Strengths of `DBUA`

- easier for administrators who prefer a wizard
- useful for single-database upgrades
- integrates prechecks into the workflow
- can run in silent mode as well as GUI mode

### Weaknesses compared with `AutoUpgrade`

- less flexible than a full `AutoUpgrade` configuration file
- not as strong for managing many databases at scale
- older operational model
- deprecated in 19c, along with manual upgrades

So yes, `DBUA` still works. It is just no longer the tool Oracle wants on the recruitment poster.

---

## 5. Manual Upgrade (maximum control, maximum opportunity for self-harm)

A manual upgrade means you do the work yourself with command-line scripts and utilities.

The key upgrade utility in that path is:

- `catctl.pl`

That is the **Parallel Upgrade Utility**, and it drives the database upgrade scripts.

### What a manual upgrade involves

You are responsible for:

- running pre-upgrade analysis
- preparing the new Oracle home
- backing up the database
- starting the database from the new home
- opening it in upgrade mode
- running the upgrade scripts
- recompiling invalid objects
- handling post-upgrade cleanup
- validating parameters and obsolete settings

Oracle documentation still allows this path, but it is the method most likely to punish typos, omissions, stale parameters, and misplaced confidence.

### Why anyone still uses it

Manual upgrade can still make sense when:

- you need fine-grained control
- your environment has very specific requirements
- you are troubleshooting or recovering from an interrupted assisted upgrade

But as a default strategy, it is mostly a fine way to create avoidable work.

---

## 6. Supported Direct Upgrade Paths to Oracle Database 19c

For Oracle Database 19c, direct upgrade is supported from:

- `11.2.0.4`
- `12.1.0.2`
- `12.2.0.1`
- `18c`

If the source database is **earlier** than `11.2.0.4`, then direct upgrade to `19c` is not supported.

That means:

- upgrade first to a supported intermediate release
- then upgrade again to `19c`

Oracle's own examples call this out for old `10g` and older `11g` patch levels.

And yes, if you are looking at a `10g` database and thinking "maybe migration would be easier," you are probably not wrong.

---

## 7. Platform and Environment Boundaries

A normal direct database upgrade assumes:

- the database remains on the **same platform path**
- the operating system and hardware are not being changed as part of the direct-upgrade workflow

If you need to change:

- operating system
- hardware platform
- storage layout in a major way

then you are usually looking at:

- migration
- transport
- `Data Pump`
- or other move-and-upgrade procedures

Oracle's upgrade docs are clear that cross-platform moves are their own topic. So if your plan is "upgrade and also move Solaris to Linux while changing the character set," congratulations, you do not have one project. You have several projects wearing a trench coat.

---

## 8. `Data Pump` Migration (the preferred rebuild-and-move method)

For data migration, Oracle's preferred tool is:

- **`Oracle Data Pump`**

The basic pattern is:

1. Install the new Oracle software
2. Create a new database, often a `CDB`
3. Create the target `PDB` if needed
4. Export metadata and data from the source database
5. Import into the target database or `PDB`

This method is especially useful because it lets you move into a **brand-new** target database rather than upgrading the old one in place.

### Why people choose `Data Pump`

`Data Pump` migration can help when you need to:

- move to a different platform
- change the character set
- move from non-`CDB` to `PDB`
- rebuild storage structures cleanly
- selectively migrate only certain schemas or objects

Oracle explicitly documents that you can:

- export from a **non-`CDB`**
- import into a **`PDB`**

So migration is not just the fallback for old versions. It is also the "while we're here, let's modernize the architecture" option.

### Full export/import vs transportable approaches

With `Data Pump`, the movement can happen in different ways:

- conventional export/import
- full transportable export/import
- network-based import in some cases

The common theme is:

- metadata is recreated in the new database
- data is then loaded or attached using the chosen method

This is why compatibility review matters so much. If you are jumping many releases, changing architecture, or changing character sets, the new database may need structural or schema-level adjustments that a direct upgrade would never attempt.

### Tradeoff

The big downside is time:

- migration can take much longer than a direct upgrade

Oracle says that plainly. `Data Pump` is flexible and powerful, but it is not the fastest path when the entire database has to move.

---

## 9. When Migration Is Better Than Upgrade

Choose migration instead of direct upgrade when one or more of these are true:

- the source release is too old for direct upgrade
- you want to move to a different operating system or hardware platform
- you need a character-set change
- you want side-by-side testing with old and new databases
- you want to redesign storage layout, tablespaces, or object placement

Migration is also useful when older database design choices need cleanup, such as legacy storage structures or layouts that are technically survivable but no longer worth carrying into the future.

In short:

- direct upgrade preserves continuity
- migration buys flexibility

Both are valid. One is just usually slower and more like moving house.

---

## 10. Planning Before the Actual Upgrade

Before you upgrade a production Oracle database, you should have more than optimism and a maintenance window.

At minimum, plan to:

- review the target-release features and deprecations
- choose the upgrade or migration method deliberately
- build and test the target Oracle home
- run pre-upgrade analysis
- review and complete required fixups
- create and validate a rollback or fallback plan
- take a fresh backup immediately before production upgrade
- test the process on a non-production copy first

Oracle explicitly recommends running:

- `AutoUpgrade` in `analyze` mode

before the actual upgrade, because it can identify issues and generate fixups before you start changing anything.

This is the part where adults make a test plan so production does not become the first time anyone sees the process end to end.

---

## 11. What Happens After the Upgrade

The transcript only previews post-upgrade work here, but the reminder is important:

After the database upgrade completes, you still need to:

- run required post-upgrade actions
- recompile and validate objects as needed
- check invalid objects and component status
- review performance and optimizer behavior
- validate application compatibility
- consider post-upgrade tuning and parameter review

So no, the upgrade is not finished the moment the tool says "success." That is simply the moment the next category of responsibility begins.

---

## 12. Practical Decision Guide

Use this rough logic:

- Choose **`AutoUpgrade`** when you want the Oracle-recommended, automated path for a supported direct upgrade
- Choose **`DBUA`** only if you specifically want the GUI-driven flow and your environment is simple enough that the older tool still makes sense
- Choose **manual upgrade** only when you truly need fine-grained control or are handling an unusual situation
- Choose **`Data Pump` migration** when version gaps, platform changes, character-set changes, or architectural changes make direct upgrade unattractive or unsupported

If you find yourself choosing the manual path because "it feels more in control," pause and ask whether what you actually mean is "more chances to make the mistake personally."

---

## 13. Wrap-Up (choose the method before the method chooses you)

This lesson covered the difference between upgrading a database and migrating data, the main Oracle Database 19c upgrade methods, why `AutoUpgrade` is the preferred tool, how `DBUA` and manual upgrades compare, when `Data Pump` migration is the better answer, and which source releases support direct upgrade to `19c`. Next comes the actual Oracle Database upgrade process, where the planning stops being theoretical and the data dictionary gets dragged into the future whether it likes it or not.
