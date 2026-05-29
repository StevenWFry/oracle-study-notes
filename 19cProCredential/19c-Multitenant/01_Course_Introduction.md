# Lesson 1 - Course Introduction (In Which Oracle Puts Many Databases Inside One Database and Calls It Simplification)

Welcome to **Oracle Database 19c: Multitenant Architecture** — the course about the architectural decision Oracle made in 12c that affects every single thing you do as a DBA today. The multitenant model is not a feature you opt into. In 19c, it is the architecture. The sooner you understand it deeply, the fewer times you will find yourself staring at a `CDB$ROOT` prompt wondering which layer of the onion you are currently peeling.

This is a broad course covering design, creation, management, security, backup, performance, upgrade, and data migration — all through the lens of CDBs and PDBs. Two topics (performance tuning and database upgrade) are so large they have their own dedicated courses, so this one gives you the foundation and points you at the door.

By the end of this course, you should be able to:

- Explain the benefits of multitenant container databases and why Oracle moved this direction
- Create and configure CDBs and PDBs from scratch
- Manage CDB and PDB lifecycle — create, duplicate, migrate, drop
- Administer security at both the container level and within individual PDBs
- Configure parameters, resources, and settings at CDB and PDB scope
- Perform backups and recovery centrally via the CDB or locally at the PDB level
- Understand the basics of performance monitoring across CDB and PDB layers
- Move data between PDBs and CDBs using Data Pump and other methods
- Understand how upgrade works at both the CDB and PDB level

---

## 1. What This Course Covers

### Multitenant Architecture and Components
- The case for CDBs — why consolidation matters
- CDB structure: root container, seed PDB, and pluggable databases
- How PDBs are isolated from each other while sharing the CDB infrastructure

### Creating and Configuring CDBs and PDBs
- Building the container database
- Provisioning PDBs: from seed, from clone, from plug-in
- Configuration at the CDB level vs the PDB level

### Security
- Application security within a PDB
- Database-level security across the CDB
- User administration: common users vs local users
- Auditing at container and PDB scope

### Parameters, Resources, and Settings
- Initialization parameters that apply to the whole CDB
- Parameters that can be set per-PDB
- Resource management and isolation between PDBs

### Backup and Recovery
- Centralized RMAN backups at the CDB level covering all or selected PDBs
- Local backup and recovery within individual PDBs
- When to back up the whole container vs a single PDB

### Performance Tuning and Monitoring *(Introduction Only)*
- Performance components available at the CDB level
- Performance components available at the PDB level
- This is a deep topic — a dedicated performance course covers it in full

### Database Upgrade *(Introduction Only)*
- Upgrading CDBs between release numbers
- Moving PDBs between CDBs at different release levels
- The Deploy, Patch, and Upgrade course covers this in full detail

### Data Movement and Migration
- Moving data at the application level within a PDB
- Moving an entire PDB between CDBs
- Cross-platform migration
- Using Data Pump for PDB-level and application-level migration

---

## 2. Target Audience

This course is for three types of people, all of whom will be doing different things with the same information:

| Role | How They Use This Material |
|------|---------------------------|
| **Database Architect** | Designing CDB/PDB structures for applications and tenants |
| **Database Administrator** | Day-to-day operations — backups, security, parameters, performance, patching |
| **Application / Cloud Administrator** | Managing the application layer within PDBs, data movement, migrations |

If you are a DBA who has been deploying non-CDB databases and quietly hoping the multitenant thing would go away: it did not. Non-CDB architecture is desupported in 20c and later. This is the architecture now.

---

## 3. Prerequisites

The course assumes you already have solid grounding in the following. If any of these are gaps, address them first — they are load-bearing:

| Knowledge Area | Where to Get It |
|----------------|----------------|
| SQL fundamentals | [SQL Workshop](../19cSQLWorkshop/) |
| DBA role and Oracle administration | [Administration Workshop](../19cAdminWorkshop/) |
| Backup and recovery concepts, RMAN | [Backup and Recovery](../19c-Backup-and-Recovery/) |
| Deployment, patching, and upgrade of CDB and grid infrastructure | Deploy, Patch, and Upgrade course |

The backup and recovery prerequisite is particularly load-bearing here — centralized CDB backup strategy is a significant chunk of the course, and it assumes you are not meeting RMAN for the first time.

---

## 4. What Comes Next

After this course, the recommended path is:

1. **Performance Tuning and Monitoring** — the dedicated course that picks up where this one leaves off on the monitoring and tuning topics
2. **Professional Certification Exam** — by this point you will have covered the full DBA skill set across SQL, administration, backup/recovery, and multitenant

The full picture for the OCP credential:

```
SQL Workshop
    └── Administration Workshop (1Z0-082)
            └── Backup and Recovery
                    └── Multitenant Architecture  ← you are here
                            └── Performance Tuning and Monitoring
                                    └── 1Z0-083 exam → OCP
```

---

## 5. Wrap-Up (The Onion Has Many Layers — Let's Start Peeling)

This course is the bridge between knowing how to administer an Oracle database and knowing how to administer an Oracle database *at scale* — multiple applications, multiple tenants, centralized management, and the architectural flexibility that the CDB/PDB model was designed to provide.

The rest of the course gets specific. Next up: the architecture itself — what a CDB actually is, what lives where, and why Oracle designed it this way.
