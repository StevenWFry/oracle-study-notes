# Practice Environment Credentials (the password circus before the real work starts)

Before doing any practice activity, know which account you are supposed to use and which password the lab image expects.

The standard access pattern is:

1. Log in to the practice environment as `demo1` using either Secure Global Desktop or TigerVNC.
2. Open a terminal.
3. Switch to `root`, `oracle`, or `grid` as the exercise requires.

---

## 1. Lab Login Flow

Use `demo1` as the initial desktop login account.

After that, switch users as needed with commands such as:

```bash
su -
su - oracle
su - grid
```

If a lab expects database authentication instead of operating-system authentication, then log in with the specific database user named in the activity.

---

## 2. Standard Password Set

The course practice environment uses these passwords:

- `root`
  - Initially set to a temporary lab password
  - Change it when the lab instructions tell you to

- `demo1`
  - Starts with its own initial password
  - Change it when instructed

- `oracle`
  - `Oracle`

- `grid`
  - `Oracle`

- Database users such as `SYS` and `SYSTEM`, and ASM-related database credentials
  - `cloud_4u`

These are **lab-only** credentials. They are intentionally simple so you can focus on the exercises instead of memorizing an entire phone book of passwords like some kind of punished archivist.

---

## 3. Practical Reminder

When a lab says "log in to the system," it usually means:

1. Sign in to the desktop as `demo1`
2. Then switch to the required OS account

That distinction matters because the practice image is set up around `demo1` as the front door, not as the account that actually installs Oracle software.
