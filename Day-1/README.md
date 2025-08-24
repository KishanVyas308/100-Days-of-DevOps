# 🚀 Day 1 of 100 Days of DevOps Challenge

## 📌 Problem Statement

The system admin team at **xFusionCorp Industries** requires a user account to be created on **App Server 2** for their backup agent tool.

However, this user should **not** be able to log in interactively (no shell access).

Instead, the account must have a **non-interactive shell**, ensuring security while still being usable by automated processes.

---

## 🎯 Task

- Create a user named **mariyam** on **App Server 2**.

- Assign the user a **non-interactive shell** (such as `/sbin/nologin` or `/bin/false`).

- Verify that the user was created successfully and cannot log in interactively.

---

## 🖥️ Infrastructure Details

- **Jumphost**: Used to access internal servers.

- **App Servers**:

  - `stapp01` → App Server 1

  - `stapp02` → App Server 2 (**Target for this task**)

  - `stapp03` → App Server 3

Each App Server has its own user credentials (e.g., `tony`, `steve`, `banner`) which are used for login via SSH.

---

## ✅ Steps to Solve

1. SSH into **App Server 2** using the provided credentials.

```bash
ssh tony@172.16.238.11
```

2. Create the user with a non-interactive shell:

```bash
sudo useradd -s /sbin/nologin mariyam
```

(If /sbin/nologin is not available, use /bin/false)

3. Verify user creation:

```bash
grep mariyam /etc/passwd
```

Output should show /sbin/nologin or /bin/false as the shell.

---

## 📚 Key Learning

- How to create Linux users with restricted shell access.
- Basics of secure user management in server environments.
- Importance of non-interactive shells for service accounts.
