# Day 2: Temporary User Setup with Expiry

## 📅 Date
**2025-08-25**

## 🎯 Task Overview

As part of the temporary assignment to the Nautilus project, a developer named **yousuf** requires access for a limited duration. To ensure smooth access management, a temporary user account with an expiry date is needed.

## 📋 Requirements

- ✅ Create a user named `yousuf` on **App Server 3** (Stratos Datacenter)
- ✅ The user must be in **lowercase** (as per standards)
- ✅ Set account expiry date to **2024-03-28**

## 🧠 Concept / What It Means

### 1. Temporary User Setup with Expiry
- In Linux, when we want a user to exist only for a fixed duration, we set an expiry date
- After the expiry date, the account will be **disabled automatically** (user can't log in)
- No manual cleanup required - system handles it automatically

### 2. Command to Use → `useradd`
The `useradd` command is used to create new user accounts with various options:

| Option | Description |
|--------|-------------|
| `-e <YYYY-MM-DD>` | Set account expiry date |
| `-s <shell>` | Set login shell |
| `-d <path>` | Set home directory |
| `-m` | Create home directory |
| `-g <group>` | Set primary group |

### 3. Why This Approach Is Used
- **Temporary Access**: Needed when external developers/contractors require access for a limited period
- **Security**: Prevents accounts from remaining active indefinitely
- **Automation**: System disables the account automatically - no manual intervention needed
- **Compliance**: Meets security standards for temporary access management

## 🛠️ Solution / Implementation Steps

### Step 1: Login to App Server 3
```bash
ssh <your-user>@<app-server-3-ip>
```

### Step 2: Create the User with Expiry Date
```bash
sudo useradd -e 2024-03-28 yousuf
```

> 👉 This creates user `yousuf` and sets account expiry to **28 March 2024**

### Step 3: (Optional) Set Password for the User
```bash
sudo passwd yousuf
```
*Enter and confirm the password when prompted*

### Step 4: Verify Expiry Date
```bash
sudo chage -l yousuf
```

**✅ Expected output:**
```
Account expires : Mar 28, 2024
```

## 🔍 Additional Verification Commands

### Check User Account Details
```bash
# View user account information
id yousuf

# Check if user exists
grep yousuf /etc/passwd

# View all account aging information
sudo chage -l yousuf
```

### Modify Expiry Date (if needed)
```bash
# Change expiry date
sudo chage -E 2024-04-15 yousuf

# Remove expiry date (make permanent)
sudo chage -E -1 yousuf
```

## 📊 When to Use This Approach

| Scenario | Use Case |
|----------|----------|
| **Temporary Projects** | Giving access to developers, interns, or contractors |
| **Short-term Assignments** | Projects that don't need permanent accounts |
| **Security Compliance** | Ensuring accounts don't remain active forever |
| **Audit Requirements** | Meeting organizational security policies |
| **Guest Access** | Providing temporary system access to external users |

## 🔐 Security Best Practices

1. **Always set expiry dates** for temporary accounts
2. **Use strong passwords** for temporary users
3. **Limit user privileges** - don't give sudo access unless necessary
4. **Monitor account usage** through system logs
5. **Document all temporary accounts** for audit purposes

## 📸 Screenshots

The following screenshots document the implementation process:

1. **Screenshot 2025-08-25 153423.png** - Initial setup
2. **Screenshot 2025-08-25 153458.png** - User creation process  
3. **Screenshot 2025-08-25 153523.png** - Verification results

## 🎓 Key Learnings

- ✅ Understanding Linux user management with expiry dates
- ✅ Using `useradd` command with time-based restrictions
- ✅ Implementing automated account lifecycle management
- ✅ Following security best practices for temporary access
- ✅ Verifying account configurations using `chage` command

## 🔗 Related Commands Reference

```bash
# User Management Commands
useradd -e YYYY-MM-DD username    # Create user with expiry
userdel username                  # Delete user
usermod -e YYYY-MM-DD username    # Modify user expiry

# Account Information Commands
chage -l username                 # List account aging info
chage -E YYYY-MM-DD username      # Set expiry date
id username                       # Show user ID info
groups username                   # Show user groups

# System Verification
grep username /etc/passwd         # Check if user exists
last username                     # Show login history
```

---

## 📚 Additional Resources

- [Linux User Management Guide](https://www.linux.org/docs/)
- [useradd Manual Page](https://man7.org/linux/man-pages/man8/useradd.8.html)
- [Account Security Best Practices](https://www.cyberciti.biz/tips/)

**Next:** [Day 3 - Secure Root SSH Access](../Day-3/)
