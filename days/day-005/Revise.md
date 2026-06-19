# Day 5: SELinux - Quick Revision Guide

## 🎯 Core Concept
**SELinux (Security-Enhanced Linux)** - Mandatory Access Control (MAC) security module that enforces security policies at the kernel level.

---

## 📋 Three SELinux Modes

| Mode | Description | Behavior |
|------|-------------|----------|
| **Enforcing** | Policy enforced | Blocks & logs violations |
| **Permissive** | Policy not enforced | Only logs violations |
| **Disabled** | SELinux off | No protection/logging |

---

## ⚡ Essential Commands

### Status & Mode
```bash
sestatus                    # Detailed status
getenforce                  # Current mode
sudo setenforce 1          # Enforcing (temp)
sudo setenforce 0          # Permissive (temp)
```

### File Contexts
```bash
ls -Z file                 # View file context
ls -Z /var/www/html/       # View directory contexts
ps -eZ                     # View process contexts
id -Z                      # View user context
```

### Change Contexts
```bash
# Temporary change
sudo chcon -t httpd_sys_content_t file

# Permanent change
sudo semanage fcontext -a -t httpd_sys_content_t "/path(/.*)?"
sudo restorecon -Rv /path

# Restore default
sudo restorecon -v file
sudo restorecon -Rv /directory
```

### Booleans
```bash
getsebool -a                        # List all
getsebool httpd_can_network_connect # Check one
sudo setsebool boolean_name on      # Enable (temp)
sudo setsebool -P boolean_name on   # Enable (permanent)
```

### Troubleshooting
```bash
sudo ausearch -m avc -ts recent     # Recent denials
sudo ausearch -m avc -ts today      # Today's denials
sudo sealert -a /var/log/audit/audit.log  # Analyze
sudo audit2allow -a                 # Generate policy
```

---

## 🔧 Configuration File

**Location:** `/etc/selinux/config`

```bash
SELINUX=enforcing    # or permissive or disabled
SELINUXTYPE=targeted # or mls or minimum
```

**Apply changes:** Reboot required for mode changes in config file

---

## 🎨 Security Context Format

```
user:role:type:level
```

**Example:**
```
system_u:object_r:httpd_sys_content_t:s0
```

- **user**: SELinux user (system_u, unconfined_u)
- **role**: SELinux role (object_r, system_r)
- **type**: Most important for policy decisions
- **level**: MLS level (s0)

---

## 🚀 Common Scenarios

### 1. Web Server Custom Document Root
```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/web/html(/.*)?"
sudo restorecon -Rv /web/html/
```

### 2. Allow Web Server Database Connection
```bash
sudo setsebool -P httpd_can_network_connect_db on
```

### 3. Allow Web Server Network Connection
```bash
sudo setsebool -P httpd_can_network_connect on
```

### 4. Custom Port for Service
```bash
sudo semanage port -a -t http_port_t -p tcp 8080
sudo semanage port -l | grep http_port_t
```

### 5. NFS Home Directories
```bash
sudo setsebool -P use_nfs_home_dirs on
```

---

## 📦 Installation Commands

### RHEL/CentOS/Rocky Linux
```bash
sudo yum install -y selinux-policy selinux-policy-targeted
sudo yum install -y policycoreutils policycoreutils-python-utils
sudo yum install -y setroubleshoot-server setools-console
```

### Debian/Ubuntu
```bash
sudo apt-get install -y selinux-basics selinux-policy-default auditd
sudo selinux-activate
sudo reboot
```

---

## 🔍 Troubleshooting Workflow

1. **Check if SELinux is the issue**
   ```bash
   sudo setenforce 0  # Set permissive
   # Test application
   sudo setenforce 1  # Set enforcing
   ```

2. **Find denials**
   ```bash
   sudo ausearch -m avc -ts recent
   ```

3. **Analyze denials**
   ```bash
   sudo sealert -a /var/log/audit/audit.log
   ```

4. **Fix the issue**
   - Restore contexts: `sudo restorecon -Rv /path`
   - Enable boolean: `sudo setsebool -P boolean on`
   - Create policy: `sudo audit2allow -a -M mypolicy && sudo semodule -i mypolicy.pp`

---

## 📊 Port Management

```bash
# List port contexts
sudo semanage port -l

# Add custom port
sudo semanage port -a -t http_port_t -p tcp 8080

# Delete port
sudo semanage port -d -t http_port_t -p tcp 8080

# Modify port
sudo semanage port -m -t http_port_t -p tcp 8080
```

---

## 🔐 Policy Module Management

```bash
# List modules
sudo semodule -l

# Install module
sudo semodule -i module.pp

# Remove module
sudo semodule -r module_name

# Enable module
sudo semodule -e module_name

# Disable module
sudo semodule -d module_name
```

---

## 📝 Log Files

| File | Description |
|------|-------------|
| `/var/log/audit/audit.log` | SELinux audit log |
| `/var/log/messages` | System messages (setroubleshoot) |
| `/etc/selinux/config` | SELinux configuration |
| `/etc/selinux/targeted/` | Policy files |

---

## ⚠️ Common Mistakes to Avoid

❌ **DON'T:**
- Disable SELinux in production
- Use `chcon` for permanent changes
- Ignore SELinux denials
- Create custom policies without checking booleans first

✅ **DO:**
- Use Enforcing mode in production
- Use `semanage fcontext` + `restorecon` for permanent changes
- Monitor logs regularly
- Check available booleans before creating policies
- Test in Permissive mode first

---

## 🎓 Quick Decision Tree

```
Application not working?
├─ Set to Permissive mode
├─ Test application
│  ├─ Works? → SELinux issue
│  │  ├─ Check denials: ausearch -m avc -ts recent
│  │  ├─ Analyze: sealert -a /var/log/audit/audit.log
│  │  └─ Fix:
│  │     ├─ Wrong context? → restorecon
│  │     ├─ Need permission? → setsebool
│  │     └─ Custom policy? → audit2allow
│  └─ Still fails? → Not SELinux issue
└─ Set back to Enforcing mode
```

---

## 🔑 Key Takeaways

1. **Three Modes**: Enforcing (production), Permissive (testing), Disabled (avoid)
2. **Context Format**: user:role:type:level
3. **Permanent Changes**: Use `semanage fcontext` + `restorecon`
4. **Temporary Changes**: Use `chcon` (lost on relabel)
5. **Booleans**: Runtime policy modifications
6. **Troubleshooting**: ausearch → sealert → fix
7. **Never Disable**: Use Permissive for troubleshooting only

---

## 📚 Most Used Commands (Top 10)

```bash
1.  sestatus                              # Check status
2.  getenforce                            # Check mode
3.  sudo setenforce 0                     # Permissive mode
4.  ls -Z /path                           # View contexts
5.  sudo restorecon -Rv /path             # Restore contexts
6.  getsebool -a | grep service           # Check booleans
7.  sudo setsebool -P boolean on          # Enable boolean
8.  sudo ausearch -m avc -ts recent       # Check denials
9.  sudo sealert -a /var/log/audit/audit.log  # Analyze
10. sudo semanage fcontext -a -t type "/path(/.*)?"  # Add context
```

---

## 🎯 Exam/Interview Quick Points

**What is SELinux?**
- Mandatory Access Control (MAC) security module
- Developed by NSA
- Enforces security policies at kernel level
- Provides fine-grained access control

**Three Modes:**
- Enforcing: Blocks and logs violations
- Permissive: Only logs violations
- Disabled: No SELinux protection

**Security Context:**
- Format: user:role:type:level
- Type is most important for policy decisions
- Every file and process has a context

**Common Tasks:**
- Change mode: `setenforce`
- View contexts: `ls -Z`, `ps -eZ`
- Restore contexts: `restorecon`
- Manage booleans: `getsebool`, `setsebool`
- Troubleshoot: `ausearch`, `sealert`, `audit2allow`

**Best Practices:**
- Always use Enforcing in production
- Use Permissive only for troubleshooting
- Check booleans before creating custom policies
- Monitor logs regularly
- Never disable SELinux without understanding implications

---

## 🔗 Quick Reference Links

- **Man Pages**: `man selinux`, `man sestatus`, `man semanage`
- **Config**: `/etc/selinux/config`
- **Logs**: `/var/log/audit/audit.log`
- **Docs**: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/

---

## ⏱️ 5-Minute Revision Checklist

- [ ] Know three SELinux modes
- [ ] Understand security context format
- [ ] Can check SELinux status (`sestatus`, `getenforce`)
- [ ] Can view contexts (`ls -Z`, `ps -eZ`)
- [ ] Can restore contexts (`restorecon`)
- [ ] Can manage booleans (`getsebool`, `setsebool`)
- [ ] Can troubleshoot denials (`ausearch`, `sealert`)
- [ ] Know when to use Enforcing vs Permissive
- [ ] Understand difference between `chcon` and `semanage fcontext`
- [ ] Know common scenarios (web server, custom ports, NFS)

---

**Last Updated:** June 19, 2026
**Topic:** SELinux Installation and Configuration
**Difficulty:** Intermediate
**Time to Master:** 2-3 hours practice