# Day 5: SELinux Installation and Configuration

## 📅 Date
**2025-08-28**

## 🎯 Task Overview

Following a security audit, the **xFusionCorp Industries** security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for **App Server 2** in the Stratos Datacenter:

- Install the required SELinux packages
- Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes
- No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight
- Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled

## 📋 Requirements

- ✅ Install required **SELinux packages** on App Server 2
- ✅ **Permanently disable** SELinux (effective after reboot)
- ✅ Configure SELinux for future re-enablement
- ✅ Verify configuration without requiring immediate reboot

## 🧠 Concept / What It Means

### 1. What is SELinux?
**Security-Enhanced Linux (SELinux)** is a mandatory access control (MAC) security mechanism:

| Feature | Description |
|---------|-------------|
| **MAC (Mandatory Access Control)** | System enforces security policy, users cannot override |
| **DAC (Discretionary Access Control)** | Traditional Linux permissions (rwx) |
| **Type Enforcement** | Objects and subjects have security types |
| **Role-Based Access** | Users assigned to specific roles |
| **Multi-Level Security** | Confidentiality levels (classified, secret, etc.) |

### 2. SELinux Operating Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Enforcing** | SELinux policy is enforced | Production security |
| **Permissive** | Policy violations logged but allowed | Testing/debugging |
| **Disabled** | SELinux is completely turned off | Legacy compatibility |

### 3. SELinux Configuration Files

| File | Purpose |
|------|---------|
| `/etc/selinux/config` | Main SELinux configuration |
| `/sys/fs/selinux/` | SELinux filesystem interface |
| `/var/log/audit/audit.log` | SELinux audit logs |
| `/etc/selinux/targeted/` | Targeted policy files |

### 4. Why Temporarily Disable SELinux?
- **Legacy Applications**: Some applications not designed for SELinux
- **Configuration Phase**: Easier to configure applications without SELinux interference
- **Testing Environment**: Isolate application issues from SELinux policies
- **Gradual Implementation**: Phased security enhancement approach

## 🛠️ Solution / Implementation Steps

### Step 1: Connect to App Server 2
```bash
# Connect to App Server 2
ssh steve@stapp02
```

### Step 2: Escalate Privileges
```bash
# Switch to root user
sudo su -
```

### Step 3: Install Required SELinux Packages
```bash
# Install SELinux packages on CentOS/RHEL
yum install -y selinux-policy selinux-policy-targeted policycoreutils
```

**Package Breakdown:**
- `selinux-policy` - Base SELinux policy framework
- `selinux-policy-targeted` - Targeted policy for common services
- `policycoreutils` - SELinux policy management tools

**Installation Output:**
```
Installing:
 selinux-policy                  noarch 38.1.64-1.el9   baseos   42 k
 selinux-policy-targeted         noarch 38.1.64-1.el9   baseos  6.9 M
Upgrading:
 policycoreutils                 x86_64 3.6-3.el9       baseos  239 k
Installing dependencies:
 rpm-plugin-selinux              x86_64 4.16.1.3-37.el9 baseos   17 k
```

### Step 4: Configure SELinux to Disabled State
```bash
# Edit SELinux configuration file
vi /etc/selinux/config
```

**Modify the configuration:**
```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled

# SELINUXTYPE= can take one of these values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

> 🔑 **Key Change**: Set `SELINUX=disabled` (was likely `enforcing` or `permissive`)

### Step 5: Verify Configuration
```bash
# Check SELinux configuration
grep SELINUX= /etc/selinux/config
```

**✅ Expected Output:**
```
# SELINUX= can take one of these three values:
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
SELINUX=disabled
```

## 🔍 Verification & Understanding

### Current SELinux Status (Before Reboot)
```bash
# Check current runtime status (may still show enforcing/permissive)
sestatus

# Check current mode
getenforce

# Check if SELinux is loaded
ls /sys/fs/selinux/
```

> ⚠️ **Important**: Runtime status may not reflect config changes until reboot

### After Reboot (Expected Results)
```bash
# After tonight's maintenance reboot
sestatus
# Expected: SELinux status: disabled

getenforce
# Expected: Disabled

ls /sys/fs/selinux/
# Expected: Empty or "ls: cannot access '/sys/fs/selinux/': No such file or directory"
```

## 📊 SELinux Package Details

### Installed Packages Analysis
| Package | Version | Purpose |
|---------|---------|---------|
| `selinux-policy` | 38.1.64-1.el9 | Core policy framework |
| `selinux-policy-targeted` | 38.1.64-1.el9 | Targeted protection policy |
| `policycoreutils` | 3.6-3.el9 | Management utilities |
| `rpm-plugin-selinux` | 4.16.1.3-37.el9 | RPM SELinux integration |

### Alternative Installation Methods
```bash
# For Ubuntu/Debian systems
apt-get update
apt-get install -y selinux selinux-basics selinux-policy-default auditd

# For minimal installations
yum groupinstall -y "Security Tools"

# For development environments
yum install -y selinux-policy-devel
```

## 🚨 Troubleshooting Common Issues

### Issue 1: Package Installation Fails
**Problem:** `No package selinux-policy available`

**Solutions:**
```bash
# Check repository configuration
yum repolist

# Enable required repositories
yum-config-manager --enable baseos
yum-config-manager --enable appstream

# Update package cache
yum clean all && yum makecache
```

### Issue 2: SELinux Config File Not Found
**Problem:** `/etc/selinux/config` doesn't exist

**Solutions:**
```bash
# Create default config file
mkdir -p /etc/selinux
cat > /etc/selinux/config << EOF
SELINUX=disabled
SELINUXTYPE=targeted
EOF

# Install base policy first
yum install -y selinux-policy
```

### Issue 3: Permission Denied Editing Config
**Problem:** Cannot modify `/etc/selinux/config`

**Solutions:**
```bash
# Check file permissions
ls -l /etc/selinux/config

# Ensure running as root
whoami

# Check if file is immutable
lsattr /etc/selinux/config

# Remove immutable attribute if set
chattr -i /etc/selinux/config
```

## 🔧 SELinux Management Commands

### Configuration Commands
```bash
# View SELinux status
sestatus                    # Detailed status information
getenforce                  # Current enforcement mode
cat /etc/selinux/config     # Configuration file

# Temporary mode changes (runtime only)
setenforce 0               # Set to permissive
setenforce 1               # Set to enforcing

# Policy management
semodule -l                # List loaded modules
getsebool -a               # List all booleans
```

### Troubleshooting Commands
```bash
# Check SELinux denials
ausearch -m avc -ts recent

# Generate policy from denials
audit2allow -a

# Check file contexts
ls -Z /path/to/file

# Restore default contexts
restorecon -R /path/to/directory
```

## 🔒 Security Implications

### Disabling SELinux: Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Reduced Security** | No MAC protection | Implement strong DAC permissions |
| **Compliance Issues** | May violate policies | Document temporary nature |
| **Attack Surface** | Increased vulnerability | Enhanced monitoring and logging |
| **Audit Trail** | Less detailed logs | Configure additional security tools |

### Best Practices During Disabled State
```bash
# Enhanced file permissions monitoring
find / -perm /4000 -type f 2>/dev/null    # Find SUID files
find / -perm /2000 -type f 2>/dev/null    # Find SGID files

# Network security
iptables -L                               # Check firewall rules
netstat -tulpn                           # Monitor open ports

# Process monitoring
ps aux                                   # Review running processes
systemctl list-units --type=service     # Check active services
```

## 📋 Future Re-enablement Preparation

### When Ready to Re-enable SELinux
```bash
# 1. Edit configuration
vi /etc/selinux/config
# Set: SELINUX=permissive (for testing) or SELINUX=enforcing (for production)

# 2. Create filesystem relabeling trigger
touch /.autorelabel

# 3. Reboot system
reboot

# 4. Monitor for denials
tail -f /var/log/audit/audit.log | grep AVC

# 5. Adjust policies as needed
audit2allow -a -M mypolicy
semodule -i mypolicy.pp
```

## 📸 Screenshots

The following screenshots document the implementation process:

1. **SSH connection and privilege escalation to root**
2. **SELinux package installation with yum showing successful installation**
3. **Configuration file editing and verification showing SELINUX=disabled**

## 🎓 Key Learnings

- ✅ Understanding SELinux architecture and operating modes
- ✅ Installing SELinux packages using package managers
- ✅ Configuring SELinux through `/etc/selinux/config`
- ✅ Distinguishing between runtime and persistent configuration
- ✅ Preparing for future SELinux re-enablement
- ✅ Balancing security requirements with operational needs

## 🔗 Related Commands Reference

```bash
# Package Management
yum install selinux-policy*              # Install all SELinux packages
rpm -qa | grep selinux                   # List installed SELinux packages
yum info selinux-policy                  # Package information

# SELinux Status and Configuration
sestatus                                 # Complete SELinux status
getenforce                              # Current enforcement mode
cat /etc/selinux/config                 # Configuration file
selinux-config-enforcing-set            # GUI configuration tool

# File and Process Context
ls -Z                                   # List with SELinux context
ps -eZ                                  # Processes with context
id -Z                                   # User context
netstat -Z                              # Network with context

# Policy Management
semodule -l                             # List policy modules
getsebool -a                            # List all booleans
setsebool boolean_name on               # Set boolean value
```

## 📋 Implementation Checklist

- [ ] Connected to App Server 2 successfully
- [ ] Escalated to root privileges
- [ ] Installed required SELinux packages
- [ ] Modified `/etc/selinux/config` to set `SELINUX=disabled`
- [ ] Verified configuration with grep command
- [ ] Documented changes for audit purposes
- [ ] Scheduled maintenance reboot confirmed
- [ ] Future re-enablement procedure documented

## 🎯 Security Audit Compliance

### Audit Trail Documentation
- **Action Taken**: SELinux temporarily disabled for application compatibility testing
- **Justification**: Legacy application compatibility during migration phase
- **Timeline**: Temporary - to be re-enabled after application configuration
- **Risk Assessment**: Compensating controls in place (enhanced monitoring)
- **Approval**: Security team authorized for testing phase

---

## 📚 Additional Resources

- [Red Hat SELinux User's and Administrator's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/)
- [SELinux Project Wiki](https://selinuxproject.org/page/Main_Page)
- [CentOS SELinux HowTo](https://wiki.centos.org/HowTos/SELinux)
- [SELinux Troubleshooting Guide](https://access.redhat.com/articles/2191331)

**Next:** [Day 6 - Create a Cron Job](../Day-6/)
