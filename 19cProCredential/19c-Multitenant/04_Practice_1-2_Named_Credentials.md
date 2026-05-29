# Practice 1-2 - Setting and Validating OEMCC Named Credentials (In Which We Learn That Sharing Passwords Is Bad, So Oracle Invented a System for Sharing Passwords Properly)

This practice is **Cloud Control specific** — it requires the second VM running Oracle Enterprise Manager Cloud Control 13.x. If you are on the single-VM personal lab setup, you do not have Cloud Control and cannot complete the hands-on steps. The concept, however, is exam-relevant and worth understanding.

---

## 1. What Named Credentials Are and Why They Exist

In an enterprise environment with multiple DBAs and consultant access, you have a problem: you need people to connect to databases with appropriate privileges, but you do not want to hand out the SYS password to everyone who needs to run an admin job.

**Named Credentials** in Cloud Control solve this by:

- Storing a database username, password, and connection role centrally in the Cloud Control repository
- Assigning a reusable name to that credential set (e.g. `CREDORCL`)
- Allowing DBAs to authenticate to a target database via the named credential without ever seeing the underlying password
- Supporting audit trails — Cloud Control logs who used which credential for what operation

Think of it as a key cabinet where the keys are labelled but the locksmiths do not need to know the key codes.

---

## 2. Cloud Control Environment Details

| Item | Value |
|------|-------|
| Cloud Control URL | `https://localhost:7803/em` (port changed from 7802 in CC 13.3+) |
| Cloud Control superuser | `sysman` |
| sysman password | `cloud_4U` |
| Management repository | `emccpdb` PDB inside `rcatcdb` CDB (VM 2) |

> **SSL certificate note:** On first access, the browser will warn about an untrusted certificate. Click **Advanced → Add Exception** and confirm the EM certificate. This is expected in a lab environment.

---

## 3. Creating the Named Credential — Steps

In Cloud Control:

1. Go to **Targets → Databases** (use Search List view)
2. Click on **ORCL** to open its homepage
3. If no login prompt appears, log out of the ORCL database session: click the username → **Logout** (logout of the database target only, not Cloud Control itself)
4. Click **New** on the login screen
5. Supply the credential:

| Field | Value |
|-------|-------|
| Username | `sys` |
| Password | `Welcome1_` |
| Role | `DBA` |
| Credential Name | `CREDORCL` |
| Set as preferred | Yes |

6. Click **Login** — this creates the named credential and stores it in the management repository

---

## 4. Validating the Named Credential in Cloud Control

Navigate to: **Setup → Security → Named Credentials**

Locate `CREDORCL` at the top of the list. The credential details panel shows:

- Credential type: Database
- Owner: `sysman`
- Target: `ORCL`
- Connection mode: `SYSDBA`
- Password: stored (not displayed)
- Preferred credential: Yes
- Access details and recent audit activity also visible

---

## 5. Validating in the Management Repository via SQL

The named credential is stored in Cloud Control's management repository inside the `emccpdb` PDB. To verify it at the SQL level:

On VM 2, connect to the Cloud Control PDB:

```bash
. oraenv    # set to rcatcdb
sqlplus sysman/cloud_4U@localhost:1521/emccpdb
```

```sql
-- Query named credentials not owned by SYSMAN itself
SELECT credential_name
FROM   sysman.mgmt_credentials2
JOIN   sysman.mgmt_manageable_entities USING (...)
WHERE  owner != 'SYSMAN';
```

Expected result: `CREDORCL` appears in the output, confirming it is stored in the management repository and available for Cloud Control operations.

---

## 6. Personal Lab — What You Can Do Instead

Without Cloud Control, you work with direct connections and EM Express. The named credential concept does not apply to EM Express (it is a single-user local tool), but you can practice the underlying security pattern directly:

**Equivalent concept — creating a dedicated admin account instead of sharing SYS:**

```sql
-- Connect as SYSDBA to CDB root
sqlplus / as sysdba

-- Create a common DBA user that acts like a named credential proxy
CREATE USER c##lab_dba IDENTIFIED BY Welcome1_ CONTAINER=ALL;
GRANT DBA TO c##lab_dba CONTAINER=ALL;

-- Connect as this user instead of SYS for admin work
CONNECT c##lab_dba/Welcome1_@localhost:1521/ORCLCDB
```

This gives you the same separation — admin work happens under a named, auditable account rather than directly as SYS — which is the principle named credentials implement at scale in Cloud Control.

---

## 7. Exam Relevance

Named credentials themselves are not a 1Z0-082 exam topic, but the underlying concepts are:

- **Why you avoid using SYS directly** — auditability, least privilege
- **Common users (`C##` prefix)** — required for users that span the whole CDB
- **Cloud Control as a management tool** — knowing it exists and what it manages is fair game

---

## 8. Wrap-Up (The Password Is Safe. The Key Cabinet Has Been Configured.)

Named credentials are an enterprise operational tool rather than a database architecture concept. The practice exists to familiarise students with the Cloud Control workflow they will use throughout the rest of the course for managing PDB targets.

For your personal lab, direct `sqlplus` connections as `SYS` or a dedicated common DBA user cover the same ground. Next up: the actual database creation and configuration exercises begin.
