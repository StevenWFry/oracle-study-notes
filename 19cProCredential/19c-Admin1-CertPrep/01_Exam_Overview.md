1# Lesson 1 - Exam Overview (In Which We Establish What You Are Getting Yourself Into)

Welcome to Oracle University's preparation course for the **1Z0-082: Oracle Database Administration I** exam. Your guide is **Branislav Valny**, Senior Principal Instructor at Oracle University and — crucially — a **12c Oracle Certified Master DBA**. That is the database equivalent of a black belt, a Michelin star, and a strongly-worded letter of recommendation from the database itself. He has tips. You should listen.

This module covers the target audience, exam structure, every topic the exam will test you on, and where to find resources that are better than frantically Googling "what is an SGA" the night before the test.

By the end of this lesson, you should be able to:

- Identify whether this exam is for you and what gaps you need to fill
- Understand the two-exam structure required for full OCP status
- Name every DBA and SQL topic category on the 1Z0-082 exam
- Know where to look for official prep materials

---

## 1. Target Audience (All DBAs, Not Just the Comfortable Ones)

This exam is for **Database Administrators** — but Oracle is fully aware that "DBA" means different things to different people:

- **Production DBA** — manages instances, monitors performance, installs software, runs backups
- **Developer DBA** — works alongside developers, stronger SQL/PL/SQL knowledge, lighter on operations
- **System Integrators / Support Engineers** — broader role, varied skill mix

Here is the uncomfortable truth: Oracle is not certifying *your kind* of DBA. They are certifying **a full-skillset DBA** who could walk into any of those roles. If your entire career has been buried in SQL and you have never touched a listener, this is your sign to fix that. And vice versa.

**Action item:** Compare the exam topic list against your actual experience and write down the gaps. A study plan built around gaps beats a study plan built around things you already know.

---

## 2. Prerequisites (Fewer Than You Think, More Than Zero)

**Officially:** No prerequisites.

**Realistically:** You should already have:

- Basic administrative knowledge using **command-line tools** (SQL*Plus)
- Familiarity with **GUI tools** (SQL Developer, Enterprise Manager)
- Some hands-on DBA experience — this is a Professional exam, not an introduction

Also, **SQL is now part of this exam**. In previous versions, SQL was its own separate exam. Oracle merged it in, so if you planned to skip the SQL sections because "I'm a DBA not a developer," that plan has been cancelled. SQL that is relevant to DBAs is fair game.

---

## 3. Exam Structure (The Two-Part Problem)

Exam 1Z0-082 is **not** a standalone credential. It is Part 1 of a two-part certification:

| Exam | Name | Purpose |
|------|------|---------|
| **1Z0-082** | Oracle Database Administration I | Part 1 — this course |
| **1Z0-083** | Oracle Database Administration II | Part 2 — required for full OCP |

To earn the **Oracle Certified Professional (OCP)** credential, you must pass **both**. Passing only 1Z0-082 gets you partway there — which is a bit like running a marathon and stopping at mile 13 because you feel pretty good about your progress.

> Note: 1Z0-083 also serves as the **upgrade exam** for OCP holders from previous Oracle releases.

---

## 4. DBA Exam Topics (The Administrative Half)

These are the DBA-focused topic areas drawn directly from the official Oracle exam page:

### Understanding Oracle Database Architecture
- Database instance configuration
- Memory and process structures (SGA, PGA, background processes)
- Logical and physical database structures
- Oracle Database Server architecture overview

### Managing Users, Roles, and Privileges
- Users, roles, privileges, and profiles
- Moving data: **External Tables**, **Oracle Data Pump**, **SQL Loader**

### Accessing an Oracle Database with Oracle-Supplied Tools
- Command-line: SQL*Plus
- GUI: SQL Developer, Enterprise Manager
- You are expected to know all of them, not just your favourite

### Managing Tablespaces and Data Files
- Create, alter, and drop tablespaces
- Manage table data storage
- Implement **Oracle Managed Files (OMF)**

### Managing Database Instances
- Startup and shutdown modes
- V$ dynamic performance views
- **Automatic Diagnostic Repository (ADR)** and alert log

### Managing Storage
- **Deferred segment creation** — know it even if you have never used it
- Space-saving features

### Configuring Oracle Net Services
- Configure the Oracle Net Listener
- Dedicated server and shared server configuration
- Administer naming methods

### Managing Undo
- Create and configure an undo tablespace
- Proper undo configuration
- **Temporary undo**

---

## 5. SQL Exam Topics (The Other Half That Developers Think They Own)

These SQL topics are DBA-relevant, but they are also just... SQL. There is no escape.

### Retrieving Data Using the SQL SELECT Statement
- Column aliases, `DESCRIBE` command
- Concatenation operator, `DISTINCT` keyword
- Arithmetic expressions and NULL values in SELECT

### Restricting and Sorting Data
- Limiting rows returned
- Substitution variables
- `DEFINE` and `VERIFY` commands

### Using Single-Row Functions to Customize Output
- Arithmetic with date data
- `ROUND`, `TRUNC`, and `MOD` functions for numbers
- Date manipulation functions

### Using Conversion Functions and Conditional Expressions
- `NVL`, `NULLIF`, `COALESCE`
- Implicit vs explicit data type conversions
- `TO_CHAR`, `TO_NUMBER`, `TO_DATE`

### Displaying Data from Multiple Tables Using Joins
- Non-equijoins
- Self-joins
- OUTER joins and all the other kinds

### Using Subqueries
- Single-row subqueries
- Multiple-row subqueries

### Using SET Operators
- `INTERSECT`, `MINUS`, `UNION`, `UNION ALL`

### Reporting Aggregated Data Using Group Functions
- Group functions (`COUNT`, `SUM`, `AVG`, etc.)
- `GROUP BY` and `HAVING`
- Restricting group results

### Understanding Data Definition Language (DDL)
- DDL commands and their effects
- Managing data in different time zones
- `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `LOCALTIMESTAMP`
- Interval data types

### DML and Managing Tables Using DML Statements
- DML language (`INSERT`, `UPDATE`, `DELETE`, `MERGE`)
- Managing database transactions (`COMMIT`, `ROLLBACK`, `SAVEPOINT`)

### Managing Sequences, Synonyms, and Indexes
- Overlap territory between DBAs and developers — own both sides

### Managing Views
- Creating, altering, and dropping views

### Managing Schema Objects
- Temporary tables
- Constraints

---

## 6. Preparation Resources (Where to Actually Study)

Branislav suggests the following official resources, which is the database equivalent of "eat your vegetables":

**Oracle Official Documentation** (docs.oracle.com):
- *SQL Language Reference* — the full SQL rulebook
- *Oracle Database Concepts* — the "why does any of this work" guide
- *Database Administrator's Guide* — your operational bible

**Oracle University Learning Subscription** (if you have access):
- **Administration Workshop** — the hands-on DBA course
- **SQL Workshop** — the hands-on SQL course

If you are in this collection, you likely already have notes from both of those courses. Use them.

---

## 7. Wrap-Up (Now You Know What You're Up Against)

You now have a complete map of the 1Z0-082 exam: who it is for, what it tests, and how it fits into the OCP credential path. The rest of this course will go topic by topic through the content.

Oracle made sure "people who are not prepared are not able to pass" — and Branislav said that with the energy of someone who has watched a lot of people not be prepared. Make a study plan. Mind the SQL sections. Come back for the next module.

Next up: the actual exam content begins.
