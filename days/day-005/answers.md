# Day 5: SELinux Installation and Configuration - Answers & Solutions

## Task Overview
Install and configure SELinux on a Linux system, understand its modes, and learn how to manage SELinux policies and contexts.

---

## Solution Steps

### Step 1: Check if SELinux is Installed

**Command:**
```bash
sestatus
```

**Expected Output (if installed):**
```
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

**If SELinux is not installed:**
```
-bash: sestatus: command not found
```

---

### Step 2: Install SELinux (if not present)

#### On RHEL/CentOS/Rocky Linux 8/9:

```bash
# Install SELinux packages
sudo yum install -y selinux-policy selinux-policy-targeted

# Install SELinux utilities
sudo yum install -y policycoreutils policycoreutils-python-utils

# Install troubleshooting tools
sudo yum install -y setroubleshoot-server setools-console
```

#### On Fedora:

```bash
# Install SELinux packages
sudo dnf install -y selinux-policy selinux-policy-targeted

# Install SELinux utilities
sudo dnf install -y policycoreutils policycoreutils-python-utils

# Install troubleshooting tools
sudo dnf install -y setroubleshoot-server setools-console
```

#### On Debian/Ubuntu:

```bash
# Update package list
sudo apt-get update

# Install SELinux
sudo apt-get install -y selinux-basics selinux-policy-default auditd

# Activate SELinux
sudo selinux-activate

# Reboot system
sudo reboot
```

**Verification:**
```bash
# Check installation
rpm -qa | grep selinux

# Expected output (RHEL/CentOS):
# selinux-policy-3.14.3-80.el8.noarch
# selinux-policy-targeted-3.14.3-80.el8.noarch
# libselinux-2.9-5.el8.x86_64
# libselinux-utils-2.9-5.el8.x86_64
```

---

### Step 3: Configure SELinux Mode

#### Check Current Configuration

```bash
# View current mode
getenforce

# View detailed status
sestatus

# View configuration file
cat /etc/selinux/config
```

#### Set SELinux to Enforcing Mode

**Method 1: Temporary (until reboot)**
```bash
# Set to enforcing
sudo setenforce 1

# Verify
getenforce
# Output: Enforcing
```

**Method 2: Permanent (survives reboot)**
```bash
# Edit configuration file
sudo nano /etc/selinux/config

# Change the following line:
SELINUX=enforcing

# Save and exit (Ctrl+X, Y, Enter)

# Reboot for changes to take effect
sudo reboot
```

**Complete Configuration File Example:**
```bash
# /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing

# SELINUXTYPE= can take one of these values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

---

### Step 4: Working with SELinux Contexts

#### View File Contexts

```bash
# View context of a specific file
ls -Z /etc/passwd

# Expected output:
# system_u:object_r:passwd_file_t:s0 /etc/passwd

# View context of directory contents
ls -Z /var/www/html/

# View context recursively
ls -lZR /var/www/html/
```

#### View Process Contexts

```bash
# View all process contexts
ps -eZ

# View specific service context
ps -eZ | grep httpd

# Expected output:
# system_u:system_r:httpd_t:s0    1234 ?        00:00:00 httpd
```

#### Change File Context (Temporary)

```bash
# Change file type
sudo chcon -t httpd_sys_content_t /var/www/html/index.html

# Verify change
ls -Z /var/www/html/index.html

# Change recursively
sudo chcon -R -t httpd_sys_content_t /var/www/html/
```

#### Change File Context (Permanent)

```bash
# Add permanent context rule
sudo semanage fcontext -a -t httpd_sys_content_t "/web/html(/.*)?"

# Apply the context
sudo restorecon -Rv /web/html

# Verify
ls -Z /web/html/
```

#### Restore Default Context

```bash
# Restore single file
sudo restorecon -v /var/www/html/index.html

# Restore directory recursively
sudo restorecon -Rv /var/www/html/

# Expected output:
# Relabeled /var/www/html/index.html from unconfined_u:object_r:user_home_t:s0 to system_u:object_r:httpd_sys_content_t:s0
```

---

### Step 5: Managing SELinux Booleans

#### List All Booleans

```bash
# List all booleans
getsebool -a

# List booleans with grep
getsebool -a | grep httpd

# Expected output:
# httpd_anon_write --> off
# httpd_builtin_scripting --> on
# httpd_can_network_connect --> off
# httpd_can_network_connect_db --> off
```

#### Check Specific Boolean

```bash
# Check single boolean
getsebool httpd_can_network_connect

# Output: httpd_can_network_connect --> off
```

#### Enable Boolean (Temporary)

```bash
# Enable boolean
sudo setsebool httpd_can_network_connect on

# Verify
getsebool httpd_can_network_connect

# Output: httpd_can_network_connect --> on

# Note: This change will be lost after reboot
```

#### Enable Boolean (Permanent)

```bash
# Enable boolean permanently
sudo setsebool -P httpd_can_network_connect on

# Verify
getsebool httpd_can_network_connect

# Output: httpd_can_network_connect --> on

# This change persists across reboots
```

---

### Step 6: Troubleshooting SELinux Issues

#### View SELinux Denials

```bash
# View recent denials
sudo ausearch -m avc -ts recent

# View denials from today
sudo ausearch -m avc -ts today

# View all denials
sudo ausearch -m avc

# View denials for specific service
sudo ausearch -m avc -c httpd
```

**Example Denial:**
```
type=AVC msg=audit(1624089600.123:456): avc:  denied  { write } for  pid=1234 comm="httpd" name="index.html" dev="sda1" ino=67890 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
```

#### Analyze Denials with sealert

```bash
# Install troubleshooting tools (if not installed)
sudo yum install -y setroubleshoot-server

# Analyze audit log
sudo sealert -a /var/log/audit/audit.log

# View specific alert
sudo sealert -l <alert_id>
```

**Example sealert Output:**
```
SELinux is preventing httpd from write access on the file index.html.

*****  Plugin catchall_boolean (47.5 confidence) suggests   ******************

If you want to allow httpd to write to user home directories
Then you must tell SELinux about this by enabling the 'httpd_enable_homedirs' boolean.

Do
setsebool -P httpd_enable_homedirs 1

*****  Plugin catchall (6.38 confidence) suggests   **************************

If you believe that httpd should be allowed write access on the index.html file by default.
Then you should report this as a bug.

You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# ausearch -c 'httpd' --raw | audit2allow -M my-httpd
# semodule -i my-httpd.pp
```

#### Create Custom Policy Module

```bash
# Generate policy from denials
sudo ausearch -c 'httpd' --raw | audit2allow -M my-httpd

# This creates two files:
# my-httpd.te (Type Enforcement file)
# my-httpd.pp (Policy Package)

# Install the policy module
sudo semodule -i my-httpd.pp

# Verify installation
sudo semodule -l | grep my-httpd
```

---

### Step 7: Common SELinux Scenarios

#### Scenario 1: Web Server Cannot Access Custom Document Root

**Problem:**
```bash
# Apache cannot serve files from /web/html/
sudo systemctl status httpd
# Shows permission denied errors
```

**Solution:**
```bash
# Set correct SELinux context
sudo semanage fcontext -a -t httpd_sys_content_t "/web/html(/.*)?"
sudo restorecon -Rv /web/html/

# Verify
ls -Z /web/html/
```

#### Scenario 2: Web Server Cannot Connect to Database

**Problem:**
```bash
# Application cannot connect to MySQL/PostgreSQL
# Check logs: sudo tail -f /var/log/httpd/error_log
```

**Solution:**
```bash
# Enable database connection boolean
sudo setsebool -P httpd_can_network_connect_db on

# Verify
getsebool httpd_can_network_connect_db
```

#### Scenario 3: Custom Port for Service

**Problem:**
```bash
# Service fails to bind to custom port 8080
# SELinux denies port binding
```

**Solution:**
```bash
# Add custom port to SELinux policy
sudo semanage port -a -t http_port_t -p tcp 8080

# Verify
sudo semanage port -l | grep http_port_t

# Expected output includes:
# http_port_t                    tcp      8080, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

#### Scenario 4: NFS Home Directories

**Problem:**
```bash
# Users cannot access NFS-mounted home directories
```

**Solution:**
```bash
# Enable NFS home directories
sudo setsebool -P use_nfs_home_dirs on

# Set correct context for NFS mount
sudo semanage fcontext -a -t nfs_t "/mnt/nfs(/.*)?"
sudo restorecon -Rv /mnt/nfs/
```

---

### Step 8: SELinux Best Practices Implementation

#### 1. Never Disable SELinux in Production

```bash
# WRONG: Disabling SELinux
sudo setenforce 0  # Temporary
# or
SELINUX=disabled   # In /etc/selinux/config

# RIGHT: Use permissive mode for troubleshooting
sudo setenforce 0  # Temporarily for testing
# Fix the issue
sudo setenforce 1  # Re-enable enforcing
```

#### 2. Regular Monitoring

```bash
# Create monitoring script
cat > /usr/local/bin/selinux-monitor.sh << 'EOF'
#!/bin/bash
# SELinux Monitoring Script

echo "=== SELinux Status ==="
sestatus

echo -e "\n=== Recent Denials ==="
ausearch -m avc -ts today | tail -20

echo -e "\n=== Current Mode ==="
getenforce

echo -e "\n=== Active Booleans ==="
getsebool -a | grep " --> on"
EOF

# Make executable
sudo chmod +x /usr/local/bin/selinux-monitor.sh

# Run monitoring
sudo /usr/local/bin/selinux-monitor.sh
```

#### 3. Backup SELinux Configuration

```bash
# Backup SELinux configuration
sudo tar -czf selinux-backup-$(date +%Y%m%d).tar.gz \
    /etc/selinux/config \
    /etc/selinux/targeted/

# List custom modules
sudo semodule -l > selinux-modules-$(date +%Y%m%d).txt

# List custom file contexts
sudo semanage fcontext -l > selinux-contexts-$(date +%Y%m%d).txt

# List custom ports
sudo semanage port -l > selinux-ports-$(date +%Y%m%d).txt

# List boolean settings
getsebool -a > selinux-booleans-$(date +%Y%m%d).txt
```

---

## Verification Checklist

- [ ] SELinux is installed and enabled
- [ ] Current mode is set to Enforcing
- [ ] Configuration file is properly configured
- [ ] Can view file and process contexts
- [ ] Can change contexts temporarily and permanently
- [ ] Can manage SELinux booleans
- [ ] Can troubleshoot SELinux denials
- [ ] Understand how to create custom policies
- [ ] Know common SELinux scenarios and solutions
- [ ] Have monitoring and backup procedures in place

---

## Common Commands Reference

```bash
# Status and Mode
sestatus                          # Detailed SELinux status
getenforce                        # Current mode
sudo setenforce 1                 # Set enforcing (temporary)
sudo setenforce 0                 # Set permissive (temporary)

# Contexts
ls -Z file                        # View file context
ps -eZ                            # View process contexts
sudo chcon -t type file           # Change context (temporary)
sudo restorecon -Rv /path         # Restore default contexts

# Permanent Context Changes
sudo semanage fcontext -a -t type "/path(/.*)?"
sudo restorecon -Rv /path

# Booleans
getsebool -a                      # List all booleans
getsebool boolean_name            # Check specific boolean
sudo setsebool boolean_name on    # Enable (temporary)
sudo setsebool -P boolean_name on # Enable (permanent)

# Troubleshooting
sudo ausearch -m avc -ts recent   # View recent denials
sudo sealert -a /var/log/audit/audit.log  # Analyze denials
sudo audit2allow -a               # Generate policy from denials

# Port Management
sudo semanage port -l             # List port contexts
sudo semanage port -a -t type -p tcp port  # Add port

# Module Management
sudo semodule -l                  # List modules
sudo semodule -i module.pp        # Install module
sudo semodule -r module_name      # Remove module
```

---

## Troubleshooting Tips

### Issue: Command not found

**Problem:**
```bash
sestatus
-bash: sestatus: command not found
```

**Solution:**
```bash
# Install SELinux utilities
sudo yum install -y policycoreutils
```

### Issue: Cannot change to enforcing mode

**Problem:**
```bash
sudo setenforce 1
setenforce: SELinux is disabled
```

**Solution:**
```bash
# Edit config file
sudo nano /etc/selinux/config
# Set SELINUX=enforcing
# Reboot system
sudo reboot
```

### Issue: Service fails after enabling SELinux

**Problem:**
Service works in permissive mode but fails in enforcing mode.

**Solution:**
```bash
# Set to permissive temporarily
sudo setenforce 0

# Start service and test
sudo systemctl start service_name

# Check for denials
sudo ausearch -m avc -ts recent

# Fix denials (restore contexts, enable booleans, etc.)
# Test again in enforcing mode
sudo setenforce 1
```

---

## Summary

SELinux installation and configuration involves:

1. **Installation**: Installing SELinux packages and utilities
2. **Configuration**: Setting the appropriate mode (Enforcing/Permissive/Disabled)
3. **Context Management**: Understanding and managing security contexts
4. **Boolean Management**: Enabling/disabling SELinux booleans
5. **Troubleshooting**: Analyzing and fixing SELinux denials
6. **Best Practices**: Regular monitoring and proper configuration

**Key Points:**
- Always use Enforcing mode in production
- Use Permissive mode only for troubleshooting
- Understand security contexts before making changes
- Use booleans when available instead of custom policies
- Monitor SELinux logs regularly
- Never disable SELinux without understanding security implications