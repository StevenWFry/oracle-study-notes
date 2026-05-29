# Practice 1-4 - Using Enterprise Manager Express (The One Practice You Can Actually Run in Your Personal Lab)

Unlike the previous two practices, this one uses **EM Express** — which comes pre-installed with the Oracle 19c RPM on your OL8 VM. Everything here is directly applicable. Follow along.

---

## 1. Check the EM Express Port

Connect as SYSDBA and verify the port:

```sql
sqlplus / as sysdba

SELECT dbms_xdb_config.gethttpsport() FROM dual;
```

Expected result: `5500`

If it shows `0`, EM Express is not configured. Enable it:

```sql
EXEC dbms_xdb_config.sethttpsport(5500);
```

---

## 2. Enable the Global Port (Important for Multitenant)

By default, EM Express is configured per-PDB on separate ports. A **global port** lets you access EM Express for both the CDB and all PDBs through a single port, with a container selector in the UI.

```sql
-- Connect as SYSDBA at CDB root
EXEC dbms_xdb_config.setglobaltxnport(5500);
```

Verify both settings:

```sql
SELECT dbms_xdb_config.gethttpsport()      AS https_port,
       dbms_xdb_config.getglobaltxnport()  AS global_port
FROM dual;
```

Both should return `5500`.

---

## 3. Access EM Express

Open a browser on the VM (or via X11/port forwarding from Windows):

```
https://localhost:5500/em
```

### SSL Certificate Warning

On first access, the browser will block the connection due to an untrusted certificate. This is expected — it is a self-signed certificate from the Oracle XDB component.

**In Firefox:**
1. Click **Advanced**
2. Click **Accept the Risk and Continue** (or "Add Exception" in older Firefox versions)
3. The EM Express login page appears

**In Chrome:**
1. Click **Advanced**
2. Click **Proceed to localhost (unsafe)**

You only need to do this once per browser profile.

---

## 4. Log In to EM Express

| Field | Value |
|-------|-------|
| Username | `sys` |
| Password | Your SYS password (set during `oracledb_ORCLCDB-19c configure`) |
| Container | `ORCLCDB` (for CDB-level view) |
| Role | `SYSDBA` |

> To connect to a specific PDB instead, select the PDB name from the Container dropdown on the login page.

---

## 5. EM Express CDB Homepage

Once logged in at the CDB level, the homepage shows:

| Section | What It Shows |
|---------|--------------|
| **Database summary** | Name, version, host, open mode |
| **Activity** | CPU, I/O, wait activity chart |
| **Services** | Active services — hover to see graph highlight |
| **Containers** | All PDBs — hover to highlight per-PDB resource usage |
| **Top SQL** | High-load SQL statements identified by the system |
| **Incidents** | Any ORA- errors or critical events logged |

On a quiet lab database, most of these sections will be blank or near-zero — that is normal. The value comes when you start running workloads.

---

## 6. Performance Hub

Navigate to: **Performance → Performance Hub**

### Time Range Selector
Choose the history window to display:
- Last **Hour**
- Last **Day**
- Last **Week**
- Last **30 Days**
- **Custom** period

### What Each Section Shows

| Section | Description |
|---------|-------------|
| **Activity** | Overall database load — CPU, wait classes over time |
| **Wait Events** | Breakdown of what sessions are waiting on |
| **SQL Statements** | Active SQL with hash value, plan info, resource consumption |
| **User Sessions** | Connected sessions with their activity statistics |
| **Monitored SQL** | SQL flagged for monitoring (manually or automatically by the database) |

### Sorting and Drilling Down

Within the SQL Statements and User Sessions tabs, you can sort by:
- Elapsed time
- CPU time
- I/O
- Buffer gets
- Executions

Click any SQL or session row to drill down into its execution details.

---

## 7. Switching Between CDB and PDB Views

With the global port enabled, you can switch containers without logging out:

1. Click the container name displayed in the top-right of the EM Express UI
2. Select `CDB$ROOT` for the full CDB view, or any PDB name for a PDB-scoped view
3. The homepage and all Performance Hub data refresh to show only that container's information

This is the GUI equivalent of:
```sql
ALTER SESSION SET CONTAINER = ORCLPDB1;
```

---

## 8. What EM Express Does Not Have (vs Cloud Control)

EM Express is a lightweight, local tool. It intentionally omits several things EMCC provides:

| Feature | EM Express | Cloud Control |
|---------|-----------|--------------|
| Multi-database management | No — single database only | Yes |
| Named credentials | No | Yes |
| Job scheduling | No | Yes |
| Patch management | No | Yes |
| Real-time SQL monitoring detail | Limited | Full |
| RMAN backup management | No | Yes |
| Configuration history | No | Yes |

For the 1Z0-082 exam: know that EM Express is a **local, single-database, performance-focused tool**. Cloud Control is the **enterprise, multi-target management platform**. They are not interchangeable.

---

## 9. SQL*Plus Equivalents for EM Express Screens

If EM Express is inaccessible (or just for exam preparation — knowing the underlying queries matters):

```sql
-- Check XDB/EM Express configuration
SELECT dbms_xdb_config.gethttpsport()     AS https_port,
       dbms_xdb_config.getglobaltxnport() AS global_port
FROM dual;

-- Current database activity (rough equivalent of the activity chart)
SELECT wait_class, COUNT(*) AS active_sessions
FROM   v$session
WHERE  wait_class != 'Idle'
AND    type = 'USER'
GROUP BY wait_class
ORDER BY active_sessions DESC;

-- Top wait events right now
SELECT event, COUNT(*) AS sessions_waiting
FROM   v$session
WHERE  wait_class != 'Idle'
AND    type = 'USER'
GROUP BY event
ORDER BY sessions_waiting DESC
FETCH FIRST 10 ROWS ONLY;

-- Container-level resource summary
SELECT con_id, COUNT(*) AS active_sessions
FROM   v$session
WHERE  type = 'USER'
GROUP BY con_id
ORDER BY con_id;
```

---

## 10. Wrap-Up (Chapter 1 Complete)

EM Express is now configured and accessible on your lab VM. With the global port set, a single URL serves both the CDB-level view and any PDB-level view through the container selector. Keep the browser tab open as you work through the course — the Performance Hub is a useful live view when you are running lab exercises that generate database activity.

This concludes the Chapter 1 practices. Next: creating CDBs and PDBs.
