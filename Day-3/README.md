# Day 3: Secure Root SSH Access

## 📅 Date
**2025-08-26**

## 🎯 Task Overview

Following security audits, the **xFusionCorp Industries** security team has rolled out new protocols, including the restriction of direct root SSH login. Your task is to disable direct SSH root login on all app servers within the **Stratos Datacenter**.

## 📋 Requirements

- ✅ Disable direct root SSH login on **all App Servers** in Stratos Datacenter
- ✅ Ensure non-root users can still access the servers
- ✅ Apply security hardening as per new protocols
- ✅ Verify the configuration is working properly

## 🧠 Concept / What It Means

### 1. SSH Root Login Security Risk
- **Direct root login** allows attackers to gain full system control if credentials are compromised
- Root has **unlimited privileges** - any mistake can damage the entire system
- **Brute force attacks** often target root accounts first

### 2. Security Best Practice: Disable Root SSH
- Force users to login with **regular accounts** first
- Use `sudo` for administrative tasks (provides **audit trail**)
- Implements **principle of least privilege**
- Reduces attack surface significantly

### 3. SSH Configuration File
The `/etc/ssh/sshd_config` file controls SSH daemon behavior:

| Directive | Description | Values |
|-----------|-------------|---------|
| `PermitRootLogin` | Controls root login via SSH | `yes`, `no`, `without-password`, `forced-commands-only` |
| `PasswordAuthentication` | Allow password-based auth | `yes`, `no` |
| `PubkeyAuthentication` | Allow key-based auth | `yes`, `no` |
| `Port` | SSH service port | Default: `22` |

### 4. Why This Approach Is Used
- **Compliance**: Meets security standards (CIS, PCI-DSS, SOX)
- **Audit Trail**: Track who performed administrative actions
- **Defense in Depth**: Multiple layers of security
- **Incident Response**: Easier to track and investigate security incidents

## 🛠️ Solution / Implementation Steps

### Step 1: Connect to App Server
```bash
# Connect to App Server (repeat for all servers)
ssh tony@172.16.238.10
```

### Step 2: Edit SSH Configuration
```bash
# Open SSH daemon configuration file
sudo vi /etc/ssh/sshd_config
```

### Step 3: Modify Root Login Setting
Find and modify the `PermitRootLogin` directive:

**Before:**
```bash
#PermitRootLogin yes
```

**After:**
```bash
PermitRootLogin no
```

> 🔑 **Vim Quick Reference:**
> - `i` - Enter insert mode
> - `Esc` - Exit insert mode  
> - `:wq` - Save and quit
> - `:q!` - Quit without saving

### Step 4: Restart SSH Service
```bash
# Restart SSH daemon to apply changes
sudo systemctl restart sshd

# Alternative command for older systems
sudo service sshd restart
```

### Step 5: Verify Service Status
```bash
# Check if SSH service restarted successfully
sudo systemctl status sshd
```

## 🔍 Verification & Testing

### Method 1: Test Root Login (Should Fail)
```bash
# From jumphost, try to login as root
ssh root@172.16.238.10
```

**✅ Expected Result:**
```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
```

**❌ If Still Prompting for Password:**
The configuration wasn't applied correctly - follow troubleshooting steps below.

### Method 2: Verify Configuration
```bash
# Check current SSH configuration
sudo sshd -T | grep permitrootlogin
```

**✅ Expected Output:**
```
permitrootlogin no
```

### Method 3: Check Configuration File
```bash
# Verify the setting in config file
sudo grep PermitRootLogin /etc/ssh/sshd_config
```

**✅ Expected Output:**
```
PermitRootLogin no
```

## 🚨 Troubleshooting Common Issues

### Issue 1: Root Login Still Works
**Problem:** `ssh root@server` still asks for password

**Solutions:**
```bash
# 1. Check if change was actually saved
sudo grep PermitRootLogin /etc/ssh/sshd_config

# 2. Verify effective configuration
sudo sshd -T | grep permitrootlogin

# 3. Check for configuration includes
sudo grep -r "PermitRootLogin" /etc/ssh/

# 4. Restart SSH service again
sudo systemctl restart sshd
```

### Issue 2: SSH Service Won't Start
**Problem:** SSH daemon fails to restart

**Solutions:**
```bash
# Check SSH configuration syntax
sudo sshd -t

# View detailed error logs
sudo journalctl -u sshd

# Check SSH service status
sudo systemctl status sshd -l
```

### Issue 3: Locked Out of Server
**Prevention:** Always ensure you have alternative access before disabling root login

```bash
# Create sudo user BEFORE disabling root
sudo useradd -m -s /bin/bash adminuser
sudo passwd adminuser
sudo usermod -aG sudo adminuser

# Test sudo access
sudo -l
```

## 📊 Implementation Across Multiple Servers

For Stratos Datacenter, repeat the process on all App Servers:

| Server | IP Address | User | Status |
|--------|------------|------|--------|
| App Server 1 | 172.16.238.10 | tony | ✅ |
| App Server 2 | 172.16.238.11 | steve | ⏳ |
| App Server 3 | 172.16.238.12 | banner | ⏳ |

### Automation Script (Optional)
```bash
#!/bin/bash
# Script to disable root SSH login

SERVERS=("172.16.238.10" "172.16.238.11" "172.16.238.12")
USERS=("tony" "steve" "banner")

for i in "${!SERVERS[@]}"; do
    echo "Configuring ${SERVERS[i]}..."
    ssh ${USERS[i]}@${SERVERS[i]} "
        sudo sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
        sudo systemctl restart sshd
        echo 'Root login disabled on ${SERVERS[i]}'
    "
done
```

## 🔐 Additional Security Hardening

### 1. Disable Password Authentication (Use Keys Only)
```bash
# Edit SSH config
sudo vi /etc/ssh/sshd_config

# Add/modify these lines
PasswordAuthentication no
PubkeyAuthentication yes
```

### 2. Change Default SSH Port
```bash
# Edit SSH config
sudo vi /etc/ssh/sshd_config

# Change port (example)
Port 2222
```

### 3. Limit User Access
```bash
# Allow only specific users
AllowUsers tony steve banner

# Or allow only specific groups
AllowGroups sysadmin
```

## 📸 Screenshots

The following screenshots document the implementation process:

1. **Screenshot 2025-08-26 105449.png** - SSH configuration editing
2. **Screenshot 2025-08-26 105520.png** - Service restart and verification
3. **Screenshot 2025-08-26 105553.png** - Testing root login restriction

## 🎓 Key Learnings

- ✅ Understanding SSH security hardening techniques
- ✅ Configuring SSH daemon through `/etc/ssh/sshd_config`
- ✅ Using `systemctl` for service management
- ✅ Implementing defense-in-depth security strategy
- ✅ Verifying security configurations effectively
- ✅ Troubleshooting SSH service issues

## 🔗 Related Commands Reference

```bash
# SSH Configuration Commands
sudo vi /etc/ssh/sshd_config          # Edit SSH config
sudo sshd -t                          # Test config syntax
sudo sshd -T                          # Show effective config
sudo systemctl restart sshd          # Restart SSH service
sudo systemctl status sshd           # Check service status

# Security Verification Commands
ssh root@server                      # Test root login (should fail)
sudo grep PermitRootLogin /etc/ssh/sshd_config  # Check config
sudo journalctl -u sshd             # View SSH logs
who                                  # Show logged in users
last                                 # Show login history

# User Management Commands
sudo useradd -m username             # Create user with home directory
sudo usermod -aG sudo username       # Add user to sudo group
sudo passwd username                 # Set user password
sudo -l                              # List sudo privileges
```

## 📋 Security Checklist

- [ ] Root SSH login disabled on all servers
- [ ] Non-root admin users have sudo access
- [ ] SSH service restarted successfully
- [ ] Root login attempts properly rejected
- [ ] Alternative access methods verified
- [ ] Changes documented and logged
- [ ] Security team notified of completion

---

## 📚 Additional Resources

- [SSH Hardening Guide](https://www.ssh.com/academy/ssh/sshd_config)
- [CIS Security Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [Linux Security Best Practices](https://www.cyberciti.biz/tips/linux-security.html)
- [SSH Key Management](https://www.ssh.com/academy/ssh/key-management)

**Next:** [Day 4 - Script Execution Permissions](../Day-4/)
