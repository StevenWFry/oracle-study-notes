# Lesson 2 - Prepare for the Exam (In Which We Confront the Gap Between What You Think You Know and What You Actually Know)

This module is about building a study plan — not the kind you sketch out, feel good about, and then ignore, but an actual structured one based on an honest assessment of your own knowledge. Branislav has watched enough exam candidates walk in overconfident and walk out humbled that he has feelings about this. Strong ones.

By the end of this lesson, you should be able to:

- Locate the official exam topic list on Oracle's website
- Categorize every exam topic into one of three knowledge tiers
- Build a study plan that allocates time where it actually matters
- Understand how Oracle tests you and what that means for how you prepare

---

## 1. Finding the Official Exam Topics

Before you can analyze your gaps, you need the topic list. Here is where to get it:

1. Go to **education.oracle.com**
2. Search for **exam 1Z0-082**
3. On the exam page, scroll down and expand **"Review Exam Topics"**

That section lists every tested topic in two categories:
- **Basic Database Administration skills**
- **SQL skills appropriate for DBAs**

Bookmark it. Refer back to it throughout your preparation. The full list was covered in [Lesson 1](01_Exam_Overview.md) if you need a reference without opening a browser.

---

## 2. Skill Gap Analysis (The Uncomfortable Part)

Before writing a single study plan entry, you must honestly assess what you know. This is called a **skill gap analysis**, and the goal is not to make yourself feel good — it is to identify where your time will actually matter.

Take the full exam topic list and assign every item to one of three categories:

---

### Category 1 — Things You Know Well ✅

You do this regularly, in the current release, and you are confident in it.

Examples:
- Administering users and tablespaces
- Querying performance views
- Writing single-table SQL statements
- Creating indexes and sequences

Also includes things you have done many times in the past **and you are certain have not changed** between releases.

> These still get reviewed — briefly — before exam day. See Section 5 on overconfidence.

---

### Category 2 — Things You Know But Need to Brush Up ⚠️

You have done this, but not recently, not in this release, or only partially. Common situations:

- **Your team split responsibilities.** You are the performance specialist now and someone else handles network admin. You know what a listener is, but you have not touched `listener.ora` in two years.
- **You changed jobs.** Your new employer does not use features your last one did.
- **You use a feature but not all of it.** Data Guard, for example — you may run it daily but only use a subset of its capabilities.
- **You used it in an older release.** The feature may have changed.

Also in Category 2: **anything new in 19c**.

Oracle publishes a *Database New Features Guide* for every release. The 19c version lives at:

> **docs.oracle.com → Oracle Database 19c → Database New Features Guide**

Focus on the **SQL** and **Administration** sections. If a feature was introduced or changed in 19c and it is on the exam topic list, it belongs in Category 2 at minimum — and possibly Category 3.

You can also find new feature coverage in blogs written by Oracle employees and independent consultants. These are often more readable than the official docs and highlight what actually matters in practice.

---

### Category 3 — Things You Do Not Know Much About ❌

You did it once in a course and mostly forgot it. You read about it but never touched it. You have never heard of it.

Examples from the transcript:
- **RPM-based database installation** — done once in a lab
- **Scalable sequences** — never encountered

This category is where the **majority of your study time** should go.

---

## 3. Building the Study Plan

Once your gap analysis is done, the study plan writes itself — roughly:

| Category | Time Allocation |
|----------|----------------|
| Category 3 (don't know) | **Most of your time** |
| Category 2 (know but rusty) | **Significant time** |
| Category 1 (know well) | **Brief review before the exam** |

**On deadlines:** Do not set an arbitrary exam date. Pick one when your study plan is genuinely complete, not because three weeks from now feels motivating right now. Set a date only if there is a real business reason for it — a project, a job requirement, an employer deadline. Otherwise, let the material tell you when you are ready.

**On a sandbox:** Set one up. Some topics cannot be fully understood without running them. Features like undo configuration, listener setup, and startup/shutdown modes have behavioral nuances that only become obvious when you break something and have to fix it.

> The [lab VM install SOP](../19cAdminWorkshop/sops/lab-vm-oracle-install.md) in this repo walks through setting up Oracle 23ai Free on Oracle Linux with the HR schema — everything you need to practice all three categories.

---

## 4. Common Obstacles (A Field Guide to Self-Sabotage)

### Obstacle 1: Overconfidence

The most common trap. You are excellent at something in your daily work, so you skip studying it. You are not wrong that you know it — you are wrong about knowing *all of it*.

**The OCM candidate story:** Branislav once proctored an Oracle Certified Master exam. A candidate was visibly excited about the Data Guard section. Daily use. Total confidence. He was not visibly excited when it was over. The reason: he had been using a specific subset of Data Guard features in his job. The exam tested the rest of them.

The lesson is not that daily experience is worthless. It is that **daily practice covers the path you walk, not the entire map**.

### Obstacle 2: Preparing for the Wrong Release

This matters differently depending on the subject area:

| Area | Volatility | Implication |
|------|-----------|-------------|
| **SQL** | Low — SQL is stable | Previous release knowledge transfers well |
| **DBA topics** | High — features change, get deprecated, get added | A correct answer from 12c may be a wrong answer in 19c |

Always verify DBA topics against the **19c documentation and new features guide**. Do not trust muscle memory on administrative procedures without checking whether they still work the same way.

---

## 5. How Oracle Actually Tests You

Oracle is explicit about this: the exam does **not** test memorization. It tests recognition and application.

There are two question styles:

### Learning Style Items
- Testing facts and features
- "What is the purpose of X?"
- "What are the components of Y?"

### Scenario Style Items
- A situation is described, and you answer based on that context
- Closer to real-world DBA work
- Tests whether you can **apply** knowledge, not just recall it

What this means for preparation:
- You do not need to memorize exact syntax for everything — but you need to understand what things do and why
- Focus on **concepts** (what is this for, how does it work)
- Focus on **implementation choices** (parameter implications, when to choose option A over option B)
- Practice reading scenarios and identifying the relevant feature or procedure

---

## 6. Wrap-Up (Make the Plan, Work the Plan)

You now have the framework:

1. Pull the official topic list from education.oracle.com
2. Assign every topic to Category 1, 2, or 3
3. Build a study plan weighted toward Category 3, with meaningful time on Category 2, and a review pass on Category 1
4. Check the 19c New Features Guide for anything that might have changed
5. Set up a sandbox and actually practice
6. Pick your exam date when the plan is done, not before

Next up: the actual exam content begins — starting with Oracle Database Architecture, the topic that underpins everything else on the list.
