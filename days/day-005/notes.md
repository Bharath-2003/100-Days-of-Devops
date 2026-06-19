# Day 5: SELinux Installation and Configuration

## Date
June 19, 2026

## Topic
SELinux (Security-Enhanced Linux) Installation and Configuration

---

## What is SELinux?

**SELinux (Security-Enhanced Linux)** is a Linux kernel security module that provides a mechanism for supporting access control security policies. It was developed by the NSA (National Security Agency) and provides Mandatory Access Control (MAC) as opposed to the traditional Discretionary Access Control (DAC).

### Key Features
- **Mandatory Access Control (MAC)**: Enforces security policies that cannot be overridden by users
- **Fine-grained Access Control**: Controls access at the process and file level
- **Security Contexts**: Every process and file has a security context
- **Policy-based**: Uses security policies to determine what actions are allowed

---

## SELinux Modes

SELinux operates in three modes:

### 1. Enforcing Mode
- **Description**: SELinux policy is enforced
- **Behavior**: Security policy violations are blocked and logged
- **Use Case**: Production environments requiring maximum security

### 2. Permissive Mode
- **Description**: SELinux policy is not enforced
- **Behavior**: Security policy violations are logged but not blocked
- **Use Case**: Testing and troubleshooting SELinux policies

### 3. Disabled Mode
- **Description**: SELinux is completely disabled
- **Behavior**: No SELinux protection or logging
- **Use Case**: Systems where SELinux is not required (not recommended)

---

## SELinux Components

### 1. Security Contexts
Every file, process, and port has a security context with four fields:
```
user:role:type:level
```

**Example:**
```
system_u:object_r:httpd_sys_content_t:s0
```

- **user**: SELinux user (system_u, unconfined_u, etc.)
- **role**: SELinux role (object_r, system_r, etc.)
- **type**: SELinux type (most important for policy decisions)
- **level**: Multi-Level Security (MLS) level

### 2. SELinux Policies
- **Targeted Policy**: Default policy, protects specific system services
- **MLS Policy**: Multi-Level Security policy for classified environments
- **Minimum Policy**: Minimal protection for specific applications

### 3. Booleans
SELinux booleans allow runtime modification of SELinux policy without recompiling:
```bash
# List all booleans
getsebool -a

# Check specific boolean
getsebool httpd_can_network_connect

# Set boolean temporarily
setsebool httpd_can_network_connect on

# Set boolean permanently
setsebool -P httpd_can_network_connect on
```

---

## Installation

### Check if SELinux is Installed
```bash
# Check SELinux status
sestatus

# Check if SELinux packages are installed
rpm -qa | grep selinux
```

### Install SELinux (if not present)

**On RHEL/CentOS/Rocky Linux:**
```bash
sudo yum install selinux-policy selinux-policy-targeted
sudo yum install policycoreutils policycoreutils-python-utils
sudo yum install setroubleshoot-server setools-console
```

**On Fedora:**
```bash
sudo dnf install selinux-policy selinux-policy-targeted
sudo dnf install policycoreutils policycoreutils-python-utils
sudo dnf install setroubleshoot-server setools-console
```

**On Debian/Ubuntu:**
```bash
sudo apt-get update
sudo apt-get install selinux-basics selinux-policy-default auditd
sudo selinux-activate
```

---

## Configuration

### 1. SELinux Configuration File
Location: `/etc/selinux/config`

```bash
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

### 2. Checking Current Mode
```bash
# Get current SELinux mode
getenforce

# Get detailed SELinux status
sestatus

# Check SELinux mode from config file
cat /etc/selinux/config | grep ^SELINUX=
```

### 3. Changing SELinux Mode

**Temporarily (until reboot):**
```bash
# Set to Enforcing
sudo setenforce 1

# Set to Permissive
sudo setenforce 0

# Note: Cannot enable/disable SELinux temporarily
```

**Permanently (survives reboot):**
```bash
# Edit configuration file
sudo nano /etc/selinux/config

# Change SELINUX= to desired mode:
# SELINUX=enforcing
# SELINUX=permissive
# SELINUX=disabled

# Reboot system for changes to take effect
sudo reboot
```

---

## Common SELinux Commands

### Status and Mode Commands
```bash
# Check SELinux status
sestatus

# Get current mode
getenforce

# Set mode temporarily
sudo setenforce 1    # Enforcing
sudo setenforce 0    # Permissive
```

### Context Management
```bash
# View file context
ls -Z /path/to/file

# View process context
ps -eZ

# View current user context
id -Z

# Restore default context
sudo restorecon -v /path/to/file

# Restore context recursively
sudo restorecon -Rv /path/to/directory
```

### Policy Management
```bash
# List all booleans
getsebool -a

# Check specific boolean
getsebool httpd_can_network_connect

# Set boolean temporarily
sudo setsebool httpd_can_network_connect on

# Set boolean permanently
sudo setsebool -P httpd_can_network_connect on
```

### Troubleshooting Commands
```bash
# View SELinux denials
sudo ausearch -m avc -ts recent

# View SELinux alerts
sudo sealert -a /var/log/audit/audit.log

# Generate policy module from denials
sudo audit2allow -a

# Create and install policy module
sudo audit2allow -a -M mypolicy
sudo semodule -i mypolicy.pp
```

---

## SELinux Context Management

### Viewing Contexts
```bash
# File contexts
ls -Z /var/www/html/

# Process contexts
ps auxZ | grep httpd

# Port contexts
sudo semanage port -l | grep http

# User contexts
sudo semanage login -l
```

### Changing Contexts
```bash
# Change file type temporarily
sudo chcon -t httpd_sys_content_t /var/www/html/index.html

# Change file type permanently
sudo semanage fcontext -a -t httpd_sys_content_t "/web/html(/.*)?"
sudo restorecon -Rv /web/html

# Restore default context
sudo restorecon -v /var/www/html/index.html
```

---

## Troubleshooting SELinux Issues

### 1. Check SELinux Denials
```bash
# View recent denials
sudo ausearch -m avc -ts recent

# View all denials
sudo ausearch -m avc

# View denials for specific service
sudo ausearch -m avc -c httpd
```

### 2. Analyze Denials
```bash
# Install troubleshooting tools
sudo yum install setroubleshoot-server

# Analyze audit log
sudo sealert -a /var/log/audit/audit.log

# Get suggestions for specific denial
sudo sealert -l <alert_id>
```

### 3. Common Solutions

**Solution 1: Fix File Contexts**
```bash
sudo restorecon -Rv /path/to/directory
```

**Solution 2: Enable Boolean**
```bash
sudo setsebool -P boolean_name on
```

**Solution 3: Create Custom Policy**
```bash
sudo audit2allow -a -M mypolicy
sudo semodule -i mypolicy.pp
```

**Solution 4: Temporarily Disable (for testing only)**
```bash
sudo setenforce 0
# Test your application
# If it works, the issue is SELinux-related
sudo setenforce 1
```

---

## Best Practices

### 1. Never Disable SELinux in Production
- Use Permissive mode for troubleshooting
- Fix the underlying issue instead of disabling SELinux

### 2. Use Targeted Policy
- Default targeted policy is suitable for most use cases
- Protects critical system services

### 3. Restore Contexts After File Operations
```bash
# After copying files
sudo restorecon -Rv /destination/

# After moving files
sudo restorecon -Rv /destination/
```

### 4. Use Booleans Instead of Custom Policies
- Check if a boolean exists for your use case
- Booleans are easier to manage than custom policies

### 5. Monitor SELinux Logs
```bash
# Regular monitoring
sudo ausearch -m avc -ts today

# Set up log rotation
# Configure /etc/logrotate.d/audit
```

### 6. Test Changes in Permissive Mode
```bash
# Set to permissive
sudo setenforce 0

# Test your changes
# Monitor logs for denials

# Fix issues and set back to enforcing
sudo setenforce 1
```

---

## Common Use Cases

### 1. Web Server (Apache/Nginx)
```bash
# Allow httpd to connect to network
sudo setsebool -P httpd_can_network_connect on

# Allow httpd to connect to database
sudo setsebool -P httpd_can_network_connect_db on

# Set correct context for web content
sudo semanage fcontext -a -t httpd_sys_content_t "/web/html(/.*)?"
sudo restorecon -Rv /web/html
```

### 2. Custom Application Port
```bash
# Add custom port for httpd
sudo semanage port -a -t http_port_t -p tcp 8080

# Verify
sudo semanage port -l | grep http_port_t
```

### 3. NFS Shares
```bash
# Enable NFS home directories
sudo setsebool -P use_nfs_home_dirs on

# Set context for NFS mount
sudo semanage fcontext -a -t nfs_t "/mnt/nfs(/.*)?"
sudo restorecon -Rv /mnt/nfs
```

---

## Security Implications

### Advantages
- **Enhanced Security**: Provides additional layer of security
- **Mandatory Access Control**: Prevents privilege escalation
- **Process Isolation**: Limits damage from compromised processes
- **Compliance**: Required for many security standards (PCI-DSS, HIPAA)

### Considerations
- **Learning Curve**: Requires understanding of SELinux concepts
- **Troubleshooting**: Can complicate debugging
- **Performance**: Minimal overhead in most cases
- **Compatibility**: Some applications may require policy adjustments

---

## Summary

SELinux is a powerful security feature that provides mandatory access control for Linux systems. While it can be challenging to configure initially, it significantly enhances system security by:

1. Enforcing security policies at the kernel level
2. Limiting the impact of security breaches
3. Providing fine-grained access control
4. Logging security policy violations

**Key Takeaways:**
- Always use Enforcing mode in production
- Use Permissive mode for troubleshooting
- Understand security contexts and policies
- Monitor SELinux logs regularly
- Never disable SELinux without understanding the security implications

---

## Additional Resources
- Red Hat SELinux Documentation
- SELinux Project Wiki
- CentOS SELinux HowTo
- Fedora SELinux FAQ