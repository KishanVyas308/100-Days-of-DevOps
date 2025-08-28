# Day 4: Script Execution Permissions

## 📅 Date
**2025-08-28**

## 🎯 Task Overview

In a bid to automate backup processes, the **xFusionCorp Industries** sysadmin team has developed a new bash script named `xfusioncorp.sh`. While the script has been distributed to all necessary servers, it lacks executable permissions on **App Server 2** within the Stratos Datacenter.

Your task is to grant executable permissions to the `/tmp/xfusioncorp.sh` script on App Server 2. Additionally, ensure that all users have the capability to execute it.

## 📋 Requirements

- ✅ Grant executable permissions to `/tmp/xfusioncorp.sh` on **App Server 2**
- ✅ Ensure **all users** can execute the script
- ✅ Verify the permissions are correctly applied
- ✅ Maintain security while enabling automation

## 🧠 Concept / What It Means

### 1. Linux File Permissions System
Linux uses a **3-bit permission system** for each file:

| Permission | Symbol | Octal | Description |
|------------|--------|-------|-------------|
| **Read** | `r` | 4 | View file contents |
| **Write** | `w` | 2 | Modify file contents |
| **Execute** | `x` | 1 | Run file as program/script |

### 2. Permission Categories
Permissions are set for three categories:

| Category | Symbol | Description |
|----------|--------|-------------|
| **Owner** | `u` | User who owns the file |
| **Group** | `g` | Group that owns the file |
| **Others** | `o` | All other users |
| **All** | `a` | All categories (u+g+o) |

### 3. Understanding Permission Output
```bash
-rwxr-xr-x 1 root root 40 Aug 28 05:43 /tmp/xfusioncorp.sh
│└┬┘└┬┘└┬┘
│ │  │  └── Others permissions (r-x)
│ │  └───── Group permissions (r-x)  
│ └──────── Owner permissions (rwx)
└────────── File type (- = regular file)
```

### 4. Why Script Permissions Matter
- **Security**: Controls who can execute potentially dangerous scripts
- **Automation**: Enables scheduled tasks and system processes
- **Collaboration**: Allows team members to run shared scripts
- **Compliance**: Meets organizational security policies

## 🛠️ Solution / Implementation Steps

### Step 1: Connect to App Server 2
```bash
# Connect to App Server 2
ssh steve@stapp02
```

### Step 2: Check Current Permissions
```bash
# View current file permissions
ls -l /tmp/xfusioncorp.sh
```

**Current State:**
```
---------- 1 root root 40 Aug 28 05:43 /tmp/xfusioncorp.sh
```
> ❌ **Problem**: No permissions for anyone (not even owner)

### Step 3: Escalate Privileges
```bash
# Switch to root user (required for root-owned files)
sudo su -
```

### Step 4: Grant Execute Permissions
```bash
# Add execute permission for all users (owner, group, others)
chmod a+x /tmp/xfusioncorp.sh
```

**Command Breakdown:**
- `chmod` - Change file mode (permissions)
- `a` - Apply to all (user + group + others)
- `+x` - Add execute permission
- `/tmp/xfusioncorp.sh` - Target file path

### Step 5: Verify New Permissions
```bash
# Check updated permissions
ls -l /tmp/xfusioncorp.sh
```

**✅ Expected Result:**
```
---x--x--x 1 root root 40 Aug 28 05:43 /tmp/xfusioncorp.sh
```

## 🔍 Permission Analysis

### Current Implementation
```
---x--x--x (Execute-only for all)
```

| User Type | Permissions | Can Do |
|-----------|-------------|---------|
| **Owner (root)** | `--x` | Execute only |
| **Group (root)** | `--x` | Execute only |
| **Others** | `--x` | Execute only |

### Alternative: Standard Script Permissions
For typical script usage, consider:
```bash
# Standard readable + executable permissions
chmod 755 /tmp/xfusioncorp.sh
```

**Result:**
```
-rwxr-xr-x (Standard script permissions)
```

| User Type | Permissions | Can Do |
|-----------|-------------|---------|
| **Owner (root)** | `rwx` | Read, write, execute |
| **Group (root)** | `r-x` | Read, execute |
| **Others** | `r-x` | Read, execute |

## 🔒 Security Considerations

### Current Security Level: High
- ✅ **Execute-only** prevents unauthorized reading of script contents
- ✅ Only **root can modify** the script
- ✅ **All users can execute** as required
- ✅ **Minimal exposure** of script logic

### Alternative Security Level: Standard
- ⚠️ **Users can read** script contents (may expose logic)
- ✅ Only **root can modify** the script
- ✅ **All users can execute** as required
- ⚠️ **Higher visibility** but easier troubleshooting

## 🚨 Troubleshooting Common Issues

### Issue 1: Permission Denied
**Problem:** `chmod: changing permissions of '/tmp/xfusioncorp.sh': Operation not permitted`

**Solutions:**
```bash
# Check if you have root privileges
whoami

# Switch to root if needed
sudo su -

# Or use sudo with chmod
sudo chmod a+x /tmp/xfusioncorp.sh
```

### Issue 2: File Not Found
**Problem:** `ls: cannot access '/tmp/xfusioncorp.sh': No such file or directory`

**Solutions:**
```bash
# Check if file exists in different location
find / -name "xfusioncorp.sh" 2>/dev/null

# Check current directory
ls -la /tmp/

# Verify you're on correct server
hostname
```

### Issue 3: Script Won't Execute
**Problem:** Script has execute permissions but won't run

**Solutions:**
```bash
# Check script syntax
bash -n /tmp/xfusioncorp.sh

# Try running with explicit interpreter
bash /tmp/xfusioncorp.sh

# Check shebang line
head -1 /tmp/xfusioncorp.sh
```

## 📊 chmod Command Reference

### Symbolic Method (Preferred)
```bash
# Add permissions
chmod u+x file    # Add execute for owner
chmod g+r file    # Add read for group  
chmod o+w file    # Add write for others
chmod a+x file    # Add execute for all

# Remove permissions
chmod u-x file    # Remove execute for owner
chmod g-r file    # Remove read for group
chmod o-w file    # Remove write for others
chmod a-x file    # Remove execute for all

# Set exact permissions
chmod u=rwx file  # Owner: read, write, execute
chmod g=r-x file  # Group: read, execute
chmod o=r-- file  # Others: read only
```

### Octal Method
```bash
chmod 755 file    # rwxr-xr-x (most common for scripts)
chmod 644 file    # rw-r--r-- (most common for data files)
chmod 600 file    # rw------- (owner only)
chmod 111 file    # --x--x--x (execute only for all)
```

### Common Permission Patterns
| Pattern | Octal | Use Case |
|---------|-------|----------|
| `rwxr-xr-x` | 755 | Executable scripts/programs |
| `rw-r--r--` | 644 | Data files, configs |
| `rw-------` | 600 | Private files (owner only) |
| `rwxrwxrwx` | 777 | Fully open (⚠️ security risk) |
| `--x--x--x` | 111 | Execute-only scripts |

## 🔍 Verification Commands

### Check Permissions
```bash
# Detailed permission listing
ls -l /tmp/xfusioncorp.sh

# Octal permission display
stat -c "%a %n" /tmp/xfusioncorp.sh

# Check if file is executable
test -x /tmp/xfusioncorp.sh && echo "Executable" || echo "Not executable"
```

### Test Execution
```bash
# Try running the script
/tmp/xfusioncorp.sh

# Check exit status
echo $?

# Run with different user (if available)
su - testuser -c "/tmp/xfusioncorp.sh"
```

## 📸 Screenshots

The following screenshots document the implementation process:

1. **Terminal session showing initial connection to App Server 2**
2. **Permission check revealing `----------` (no permissions)**  
3. **Successful chmod execution and verification showing `---x--x--x`**

## 🎓 Key Learnings

- ✅ Understanding Linux file permission system (rwx)
- ✅ Using `chmod` with symbolic notation (`a+x`)
- ✅ Distinguishing between owner, group, and others permissions
- ✅ Applying principle of least privilege for script execution
- ✅ Troubleshooting permission-related issues
- ✅ Balancing security with functionality requirements

## 🔗 Related Commands Reference

```bash
# Permission Management
chmod [permissions] file             # Change file permissions
chown user:group file               # Change file ownership
chgrp group file                    # Change group ownership
umask                               # View/set default permissions

# Permission Viewing
ls -l file                          # List detailed permissions
stat file                          # Show detailed file information
getfacl file                        # Show ACL permissions (if enabled)

# File Testing
test -r file                        # Check if readable
test -w file                        # Check if writable  
test -x file                        # Check if executable
file /tmp/xfusioncorp.sh           # Show file type

# System Information
whoami                              # Current user
groups                              # User's groups
id                                  # User and group IDs
```

## 📋 Security Checklist

- [ ] Script has execute permissions for all users
- [ ] File ownership remains with root (security)
- [ ] Minimal permissions applied (execute-only)
- [ ] No write permissions for non-owners
- [ ] Script functionality verified
- [ ] Changes documented and logged
- [ ] Backup process automation enabled

## 🎯 Best Practices

### 1. Principle of Least Privilege
- Grant **minimum permissions** required for functionality
- Avoid `777` permissions (full access for everyone)
- Regularly **audit file permissions**

### 2. Script Security
- **Validate script contents** before granting execute permissions
- Use **execute-only permissions** for sensitive scripts
- Implement **logging and monitoring** for script execution

### 3. Documentation
- **Document permission changes** with business justification
- Maintain **change logs** for audit purposes
- Include **rollback procedures** for security incidents

---

## 📚 Additional Resources

- [Linux File Permissions Guide](https://www.linux.org/docs/man1/chmod.html)
- [Security Best Practices for Scripts](https://www.cyberciti.biz/tips/linux-file-permissions-for-security.html)
- [Advanced Permission Management](https://www.redhat.com/sysadmin/linux-file-permissions-explained)
- [Bash Scripting Security](https://mywiki.wooledge.org/BashFAQ)

**Next:** [Day 5 - SELinux Installation and Configuration](../Day-5/)
