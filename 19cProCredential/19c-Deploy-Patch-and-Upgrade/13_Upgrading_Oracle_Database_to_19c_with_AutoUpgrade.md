## Lesson 13 - Upgrading Oracle Database to 19c with AutoUpgrade (where the jar file does the choreography and you try not to sabotage the stage)

This lesson is the actual **AutoUpgrade** chapter. The planning work is already done, the target `19c` home is supposed to exist, and now Oracle would like to automate the rest of the database upgrade before a human gets creative. `AutoUpgrade` can analyze the source database, perform many required fixups, run the upgrade, handle a large chunk of the post-upgrade work, and restart the upgraded database in the new home. Which is very helpful, because the manual alternative remains available for people who feel underchallenged.

By the end of this lesson, you should be able to:

- Explain what `AutoUpgrade` automates during a database upgrade
- Prepare the patched target `19c` home used for the upgrade
- Describe the `AutoUpgrade` processing modes and when to use each
- Explain how `deploy` differs from `upgrade` mode
- Create and organize an `AutoUpgrade` configuration file
- Describe the backup and restore requirements behind a safe `AutoUpgrade` run

---

## 1. What AutoUpgrade Is Supposed to Do

`AutoUpgrade` is Oracle's automation layer for database upgrades.

It can:

- run pre-upgrade analysis
- identify upgrade blockers
- perform many automated fixups
- shut down and restart the database in the correct home
- run the upgrade itself
- perform many post-upgrade checks and fixups
- bring the upgraded database online again

That is why Oracle recommends it. It is basically the official answer to the question, "Can we stop hand-driving every upgrade step like it's 2008?"

One important operational rule:

- run `AutoUpgrade` from the **target release tooling**

In practice, that means from the new `19c` environment or with the latest downloaded `autoupgrade.jar` that is compatible with the target release.

Oracle explicitly recommends downloading the **latest AutoUpgrade version**, because the tool is updated more often than the base software image. So the course's "patched target home first" advice is sound, but Oracle's current docs push one step further: use the newest `AutoUpgrade` you can get, not just whatever happened to ship in the home.

---

## 2. The Target 19c Home Comes First

Before `AutoUpgrade` can upgrade the database, the target `19c` Oracle home must already exist.

That means:

- choose a **new** Oracle home location
- install the `19c` database software there
- patch it to the target `RU`
- validate that the home is ready

The course's preferred pattern is to:

- patch the software first
- create or use a gold image
- unzip that image into the future Oracle home
- install the software-only home

This matters because `AutoUpgrade` is not meant to upgrade your database into a half-built, half-patched target home that still smells like unzip output and denial.

---

## 3. Software-Only Install for the Target Home

The target `19c` home is installed with:

- `runInstaller`

And for an upgrade path, the key installer choice is:

- **Set Up Software Only**

Not:

- create and configure a new database

That would be useful for migration scenarios, but not for an out-of-place direct database upgrade where the existing database will later be switched into the new home.

During the install, validate:

- single-instance versus `RAC`
- Enterprise Edition versus Standard Edition
- Oracle base
- Oracle home path
- OS group mappings
- root script automation
- prerequisite checks

Once the installer finishes successfully, the target home exists and is ready for `AutoUpgrade`.

So yes, the software installation is the warm-up act. The actual database upgrade still has not happened yet.

---

## 4. Preparing to Run AutoUpgrade

Before running `AutoUpgrade`, Oracle expects more than "we installed the home and felt optimistic."

At minimum, prepare by:

- reviewing `19c` features and behavior changes
- validating operating system and Oracle home readiness
- choosing the upgrade method
- testing the process against a non-production copy
- making sure the new home is complete and patched
- creating a reliable backup strategy

The course also correctly emphasizes testing tools such as:

- Database Replay
- SQL Performance Analyzer

Those are useful because a successful upgrade is not just "dictionary finished upgrading." It is also "the application did not discover new and inventive ways to behave badly afterward."

### Security option caveat

If the source database uses options such as:

- Oracle Database Vault
- Oracle Label Security

then you may need extra preparation before upgrade.

Oracle's upgrade docs are explicit that:

- Database Vault may need to be disabled or handled with `DV_PATCH_ADMIN`
- Oracle Label Security may require preprocess or disable/enable steps depending on the source release and configuration

So treat those features as special preparation work, not as background decoration.

---

## 5. AutoUpgrade Processing Modes

Oracle documents four main processing modes for `AutoUpgrade`:

- `analyze`
- `fixups`
- `deploy`
- `upgrade`

These modes are not interchangeable. They exist because Oracle knows not every upgrade happens on one host, in one step, with one neat rollback story.

### `analyze`

Use `analyze` to:

- run pre-upgrade checks
- identify blockers
- produce reports and fixup guidance

Important behavior:

- `analyze` is **read only**
- it does not change the database

That means you can run it early and often without immediately creating new problems. A rare kindness.

### `fixups`

Use `fixups` to:

- perform automated corrections on the source database before upgrade

Important behavior:

- `fixups` **does** modify the source database
- Oracle recommends running `analyze` first
- `fixups` does **not** create a guaranteed restore point

That last detail matters. The transcript drifted a little here. Guaranteed restore point handling belongs to the actual deployment path, not to fixups mode.

### `deploy`

Use `deploy` when the database upgrade is happening in the usual same-environment flow.

`deploy` performs:

- prechecks
- any pending fixups
- the actual upgrade
- many post-upgrade checks and fixups

This is the "do the whole thing" mode, and it is usually the best choice for a normal single-host out-of-place upgrade.

### `upgrade`

Use `upgrade` when:

- you already performed analyze/fixups
- the database has been moved to another host or home
- you cannot use the normal `deploy` flow

This mode is especially useful when the database has already been relocated and is ready to be upgraded in its new context.

So the rule of thumb is:

- same-host, standard path: prefer `deploy`
- moved database / split workflow: use `upgrade`

---

## 6. Single-Host vs Cross-Host Behavior

On a typical single-host upgrade:

- `deploy` is enough

It can:

1. analyze readiness
2. run pending fixups
3. perform the upgrade
4. run post-upgrade processing

On a cross-host or moved-database workflow:

1. run `analyze`
2. run `fixups`
3. move or recreate the database in the target context as required
4. start it in the appropriate home and state
5. run `upgrade`

This is why Oracle provides both `deploy` and `upgrade` modes. If the database has already been moved, `deploy` no longer owns the entire life story.

---

## 7. AutoUpgrade and Fallback Requirements

`AutoUpgrade` is only as safe as the fallback path behind it.

For **Enterprise Edition**, Oracle's preferred fallback is:

- `Flashback Database`
- using a **guaranteed restore point** during deploy processing

For that to work, the source database must have:

- `ARCHIVELOG` mode enabled
- a properly configured `FRA`
- `DB_RECOVERY_FILE_DEST`
- `DB_RECOVERY_FILE_DEST_SIZE`

If those are not configured correctly, `AutoUpgrade` cannot create the guaranteed restore point it expects for a full deploy path.

Oracle documents this clearly. `ARCHIVELOG` and a usable `FRA` are not optional if you want the full restore-point safety net.

### Standard Edition caveat

For Standard Edition / Standard Edition 2:

- `Flashback Database` is not available
- guaranteed restore point rollback is not the same safety mechanism

Which means the backup strategy matters even more, because Oracle will not politely rewind the mistake for you.

---

## 8. AutoUpgrade Stages at a High Level

When Oracle runs `AutoUpgrade`, the work moves through a staged flow.

At a practical level, you should expect phases such as:

- setup / job initialization
- prechecks
- pre-upgrade fixups
- restore-point creation for deploy workflows
- shutdown / drain
- database startup in the correct mode and home
- actual upgrade execution
- invalid object handling and recompilation
- post-upgrade checks
- post-upgrade fixups
- final handoff into the new home

Oracle's docs also note an operationally useful detail:

After a successful deploy, `AutoUpgrade` can copy database-related files from the source home to the target home, such as:

- `sqlnet.ora`
- `tnsnames.ora`
- `listener.ora`

It also handles home-local files such as password files when they need to follow the database into the new home. If those files were already centralized outside the old database home, then the new setup keeps using that central location instead of inventing a second identity crisis.

---

## 9. The AutoUpgrade Configuration File

`AutoUpgrade` lives or dies by its configuration file.

That file identifies:

- which database or databases to upgrade
- source and target homes
- database names and SIDs
- logging locations
- upgrade options
- post-upgrade behavior

Oracle supports:

- **global parameters**
- **local parameters**

### Global parameters

Global parameters apply to all jobs in the configuration file.

These are useful for:

- shared logging locations
- common target-release settings
- standard behaviors across many database jobs

### Local parameters

Local parameters apply only to a specific job.

These are useful for:

- per-database names
- per-database home paths
- per-database scheduling or options

If a parameter is defined both globally and locally:

- the **local** value wins

Which is exactly what you want, because otherwise a multi-database config file would be a very elegant way to upgrade everything incorrectly at once.

---

## 10. Building the Config File Without Hating Yourself

Oracle provides a much saner starting point than typing the whole file from memory:

- create a **sample config file**

That sample file shows:

- global options
- local job options
- common upgrade parameters
- optional clauses you may want later

The practical approach is:

1. create the sample file
2. store it in a central upgrade-config directory
3. edit it into the real config for your environment
4. keep logs in a separate, predictable log directory

That organization matters, especially when upgrading more than one database or when you need to hand logs to Oracle Support later instead of saying, "I think the file was somewhere under `/tmp`, unless I overwrote it."

Common values the course calls out include:

- database name
- SID
- source home
- target home
- target version
- log location
- whether to run `utlrp`
- whether to upgrade time zone data

And yes, there are many more. Oracle has never met a config surface it could not make more expressive.

---

## 11. Useful AutoUpgrade Command-Line Options

The transcript rattles off a lot of command-line options. The ones worth keeping in working memory are the high-value ones.

### Core execution options

- `-config`
  - points to the configuration file

- `-mode analyze|fixups|deploy|upgrade`
  - chooses the processing mode

- `-create_sample_file`
  - generates a starter config file

### Visibility and troubleshooting options

- `-console`
  - enables the interactive console for monitoring jobs

- `-noconsole`
  - runs without the interactive console, useful for scripts

- `-debug`
  - provides extra diagnostic output

- `-zip`
  - packages logs and related data, useful for support cases

### Recovery and restart options

- `-clear_recovery_data`
  - clears stored job recovery metadata so a fresh run can begin

- `-restore_on_fail`
  - enables automatic restore on failure where supported

### Password handling

- `-load_password`
  - securely loads required passwords, such as keystore or SYS credentials, into the AutoUpgrade keystore

These options are useful because they map directly to real-world pain:

- bad config syntax
- needing a clean rerun
- monitoring a long-running job
- handing zipped logs to Oracle Support after things become exciting

---

## 12. Where the JAR File Lives

`AutoUpgrade` is run as a Java archive:

- `autoupgrade.jar`

The course points to the target Oracle home's `rdbms/admin` area, which is a normal place to find it in an installed home.

In practice, the important rules are:

- use the **latest AutoUpgrade version** Oracle recommends
- use a compatible Java runtime
- run the jar with the config file and chosen mode

Typical syntax looks like:

```bash
$ORACLE_HOME/jdk/bin/java -jar $ORACLE_HOME/rdbms/admin/autoupgrade.jar -config myconfig.cfg -mode analyze
```

The exact path can vary if you downloaded a newer copy separately, but the mental model is the same: Java launches the jar, the config file tells it what to do, and the mode tells it how invasive to be.

---

## 13. Backup Strategy Before AutoUpgrade

Even when `AutoUpgrade` can create a guaranteed restore point, Oracle still wants a real backup strategy.

That means:

- take a fresh backup before the upgrade
- validate that the backup is usable
- tag it clearly for the upgrade event
- consider isolating it from routine backup rotation

The course leans toward:

- a full `RMAN` backup
- often a cold backup pattern for the upgrade event
- separate backup of the control file
- possibly a text backup of the parameter/control configuration where useful

Oracle's docs are slightly more general here:

- design the backup strategy first
- run `AutoUpgrade` preupgrade checks before the clean shutdown
- ensure no conflicting `RMAN` jobs are removing archive logs during the backup

The operational point is simple:

- guaranteed restore point is a convenience
- backup is the actual adult fallback plan

If the upgrade goes bad and both your restore-point path and your backup path are weak, then congratulations, you have built a lesson in preventable regret.

---

## 14. Practical AutoUpgrade Checklist

Before running `AutoUpgrade`, confirm:

- the target `19c` home is installed and patched
- you are using the latest suitable `autoupgrade.jar`
- the config file is organized and reviewed
- source and target homes are correct
- `ARCHIVELOG` mode is enabled if you want deploy-mode GRP protection
- the `FRA` is configured and sized correctly
- the backup strategy is complete and tested
- special options like Database Vault or OLS have been handled
- you know whether this job should use `deploy` or `upgrade`
- log directories are prepared and easy to find later

If those answers are vague, then `AutoUpgrade` may still be automated, but the surrounding project is still being run like a superstition.

---

## 15. Wrap-Up (automation is not magic, but it is better than heroics)

This lesson covered the actual Oracle Database 19c upgrade flow with `AutoUpgrade`: preparing the patched target home, understanding the processing modes, choosing `deploy` versus `upgrade`, meeting the `ARCHIVELOG` and `FRA` requirements for restore-point protection, structuring the configuration file, using high-value command-line options, and backing up properly before the run. Next comes the remaining upgrade paths and cleanup work, where Oracle reminds you that even automated upgrades still expect you to read the logs and clean up after yourself.
