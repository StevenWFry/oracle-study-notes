# Lab 6-3 - Out-of-Place Patching Grid Infrastructure and Creating a Gold Image

This lab builds a new `19c` Grid Infrastructure home out of place, applies the `19.20` `RU`, and then creates a gold image zip file for later upgrade work. The whole point is to patch the new home before it is used, which is much saner than trying to modernize a home while it is still actively employed.

Before starting, review [practice-environment-credentials.md](practice-environment-credentials.md).

---

## 1. Lab Goal

In this lab you will:

- Create a new `19c` Grid Infrastructure target directory
- Create an empty directory for the `19.20` patch unzip
- Extract the `19.3` Grid Infrastructure image
- Replace the bundled `OPatch` with the staged newer version
- Use `gridSetup.sh` to save a response file
- Cancel the GUI install and rerun it silently with `-applyRU`
- Create a Grid Infrastructure gold image

---

## 2. Assumptions

This lab assumes:

- The `grid`, `oracle`, and `root` accounts already exist
- The `19.3` Grid Infrastructure image zip is already staged
- The `19.20` patch bundle is already staged
- The newer `OPatch` zip is already staged
- The host has enough free space for an out-of-place `19c` home plus a gold image zip

The exact staged filenames are not preserved cleanly in the transcript, so this lab uses placeholders such as `<grid_19c_zip>`, `<ru_patch_zip>`, and `<opatch_zip>`.

---

## 3. Create the Target Directories as `root`

Open a terminal and become `root`:

```bash
su -
```

Use the lab password:

```text
Oracle
```

Create the new `19c` Grid Infrastructure home and a clean patch-unzip directory:

```bash
mkdir -p /u01/app/19c/grid
mkdir -p /u01/stage/grid_ru_1920
```

Set ownership correctly:

```bash
chown -R grid:oinstall /u01/app/19c/grid /u01/stage/grid_ru_1920
```

The demo also creates the later `19c` database home directory during this broader exercise. That is fine if you want to mirror the classroom exactly, but this lab focuses on the Grid side.

---

## 4. Unzip the Patch and the Base `19.3` Grid Image

Open a second terminal and become the `grid` user:

```bash
su - grid
```

Use the lab password:

```text
Oracle
```

Unzip the `19.20` patch into the empty patch directory:

```bash
cd /u01/stage/grid_ru_1920
unzip -q /stage/19c_patches/<ru_patch_zip>
```

Now extract the base `19.3` Grid Infrastructure image into the new home:

```bash
cd /u01/app/19c/grid
unzip -q /stage/19c_software/<grid_19c_zip>
```

---

## 5. Replace the Bundled `OPatch`

Remove the image's bundled `OPatch` directory:

```bash
rm -rf /u01/app/19c/grid/OPatch
```

Extract the staged newer `OPatch` into the Grid home:

```bash
unzip -q /stage/19c_patches/<opatch_zip> -d /u01/app/19c/grid
```

Oracle images routinely ship with an `OPatch` version that is perfectly capable of disappointing you later. Replacing it up front is the less dramatic path.

---

## 6. Use the GUI Once to Save a Response File

Launch the Grid Infrastructure installer from the new `19c` home:

```bash
cd /u01/app/19c/grid
./gridSetup.sh
```

In the wizard:

1. Select **Set Up Software Only**
2. Accept the standalone defaults used by the lab image
3. Verify the OS group mappings
4. Set Oracle base to:

```text
/u01/app/grid
```

5. Choose automatic root script execution
6. Let the prerequisite checks complete
7. On the summary page, click **Save Response File**
8. Save the file with a memorable name such as:

```text
grid_1920.rsp
```

9. Return to the summary page and click **Cancel**
10. Confirm that you really want to exit

The point of the GUI run is not to install. It is to harvest a response file with the values you want, then use that file for the silent patched build.

---

## 7. Apply the `RU` Silently to the New Grid Home

From the `grid` terminal, run the silent install using the saved response file:

```bash
cd /u01/app/19c/grid
./gridSetup.sh -silent -responseFile /home/grid/grid_1920.rsp -applyRU /u01/stage/grid_ru_1920/<ru_patch_dir>
```

If your response file was saved under a different name or directory, use that full path instead.

Because the installer was configured to run root scripts automatically, it will eventually prompt for the `root` password. Enter:

```text
Oracle
```

The demo notes that the password prompt may take around 10 to 12 minutes to appear. Oracle enjoys suspense.

---

## 8. Create the Grid Infrastructure Gold Image

After the patched `19c` Grid home is built successfully, create a gold image:

```bash
cd /u01/app/19c/grid
./gridSetup.sh -createGoldImage -destinationLocation /u01/stage/gold-images -silent
```

This creates a reusable zip image of the patched Grid home for later activities.

The demo notes that gold image creation takes roughly 10 minutes in the lab environment.

---

## 9. Finish the Lab

When the gold image completes successfully:

- confirm the zip file exists in the destination directory
- close the terminal windows

---

## 10. What You Just Finished

By the end of this lab you:

- built a new out-of-place `19c` Grid home
- patched it from `19.3` to `19.20`
- captured the installer choices in a response file
- created a reusable Grid gold image

Which means future upgrade work can start from a prepatched home instead of a fresh pile of software optimism.
