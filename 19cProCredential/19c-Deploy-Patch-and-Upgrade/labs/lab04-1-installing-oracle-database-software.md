# Lab 4-1 - Installing Oracle Database Software

This lab installs the Oracle Database software home on the lab Linux server. The course deliberately uses the `12.2` database software here so later exercises can patch and upgrade it instead of pretending real environments begin life fully modern and emotionally well adjusted.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Open one terminal as `root` for the permission and root-script work
- Open a second terminal as `oracle`
- Extract the staged Oracle Database `12.2` media
- Launch `runInstaller`
- Choose a software-only, single-instance Enterprise Edition install
- Run the required root script
- Finish with a completed Oracle Database home

---

## 2. Open Two Terminals

Open two terminal windows.

### Terminal 1: `root`

Become `root`:

```bash
su -
```

Use the lab password:

```text
Oracle
```

The transcript references an initial permission adjustment on `/u01/app` before the database install begins. The exact command is not preserved cleanly in the source, so if your lab image includes a provided `chmod` or `chown` step for `/u01/app`, run that here first.

### Terminal 2: `oracle`

In the second terminal, become the `oracle` user:

```bash
su - oracle
```

Use the lab password:

```text
Oracle
```

---

## 3. Go to the Staged Database Media

Change to the staged `12.2` database media directory.

Typical lab path:

```bash
cd /stage/12.2.0/database
```

If your class image uses a slightly different staging path, adjust accordingly. The important point is that you are in the staged Oracle Database `12.2` media directory before extraction.

---

## 4. Unzip the Database Software

Extract the database installation media as the `oracle` user.

Typical pattern:

```bash
unzip -q LINUX.X64_12201_database_home.zip
```

After extraction, the transcript indicates removing the zip file to keep the staging area tidy. If your lab guide expects that cleanup, do it now:

```bash
rm -f LINUX.X64_12201_database_home.zip
```

Then change into the extracted database directory if needed:

```bash
cd database
```

If your lab image extracts directly into a different path, use the actual extracted directory instead. `Oracle` does not care about your desire for clean storytelling; it cares whether `runInstaller` exists where you claim it does.

---

## 5. Start Oracle Universal Installer

Launch the installer:

```bash
./runInstaller
```

This starts `OUI` for the database software install.

---

## 6. Skip Security Updates for the Practice Environment

On the **Configure Security Updates** page:

- Clear **I wish to receive security updates via My Oracle Support**
- Click **Next**
- Confirm the warning and continue

This is a lab convenience, not an operational recommendation for systems that anyone is planning to trust after lunch.

---

## 7. Choose a Software-Only Installation

On the installation options page:

- Choose **Install database software only**

This keeps the database creation step separate so later exercises can create and upgrade the database independently.

---

## 8. Choose Single-Instance Enterprise Edition

On the next installer pages, select:

- **Single instance database installation**
- **Enterprise Edition**

This lab is not creating a `RAC` database and is not creating the database itself yet. It is only building the database home.

---

## 9. Confirm Oracle Base and Oracle Home

Use these paths:

- Oracle base:

```text
/u01/app/oracle
```

- Software location:

```text
/u01/app/oracle/product/12.2.0/dbhome_1
```

Keep those values unless your specific lab image says otherwise.

---

## 10. Keep the Default Database OS Group Mappings

On the privileged operating system groups page:

- Keep the default group mappings

These should already match the earlier lab setup for the `oracle` account and the database administrative groups.

---

## 11. Let the Prerequisite Checks Finish

Wait for the prerequisite checks to complete.

If the checks pass:

- Continue to the summary page
- Review the selections
- Click **Install**

You may also save a response file if you want a reusable record of the choices, but that is optional for this lab.

---

## 12. Run the Root Script

During the installation, `OUI` opens a popup that shows the configuration script path and script name.

Use the **root** terminal you opened earlier and run the displayed script.

Typical example:

```bash
/u01/app/oracle/product/12.2.0/dbhome_1/root.sh
```

When prompted during `root.sh`, press **Enter** to accept the default value if the lab instructs you to do so.

After the script completes successfully:

- Return to `OUI`
- Click **OK**

This tells the installer that the privileged part of the work is done and Oracle may continue pretending everything happened naturally.

---

## 13. Finish the Install

When installation completes successfully, the finish page reports that Oracle Database installation succeeded.

Click **Close** to exit `OUI`.

---

## 14. What You Just Finished

By the end of this lab you:

- Opened separate `root` and `oracle` terminals for the install workflow
- Extracted the staged Oracle Database `12.2` media
- Launched `runInstaller`
- Installed the Oracle Database software only
- Chose a single-instance Enterprise Edition path
- Ran the required root script
- Finished with a completed database software home

Which means the binaries are now installed and ready for the next step, where Oracle stops being a pile of files and starts becoming an actual database with opinions.
