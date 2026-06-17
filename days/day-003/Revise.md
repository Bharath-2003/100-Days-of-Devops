# Day 003 - Quick Revision Guide

## 📚 SSH Security - Disable Root Login

### Core Concept
**Goal:** Prevent direct root SSH login to improve server security

**Why:** Root login is a major security risk - attackers know the username (root) and only need to guess the password.

---

## 🔧 Essential Commands

#### 1. Edit SSH Configuration
```bash
sudo nano /etc/ssh/sshd_config
```
**Description:** Opens SSH server configuration file for editing.

**Alternative editors:**
```bash
sudo vi /etc/ssh/sshd_config
sudo vim /etc/ssh/sshd_config
```

---

#### 2. Disable Root Login (Configuration Change)
```bash
# In /etc/ssh/sshd_config, find and change:
PermitRootLogin yes

# To:
PermitRootLogin no
```
**Description:** This is the key configuration change that disables direct root SSH login.

**Other PermitRootLogin options:**
- `yes` - Root can login with password (❌ Insecure)
- `no` - Root cannot login via SSH (✅ Most Secure)
- `prohibit-password` - Root can login with SSH keys only (⚠️ Moderate)
- `forced-commands-only` - Root can only run specific commands (⚠️ Moderate)

---

#### 3. Test SSH Configuration
```bash
sudo sshd -t
```
**Description:** Tests SSH configuration for syntax errors BEFORE restarting the service. No output means configuration is valid.

**Example:**
```bash
$ sudo sshd -t
# No output = configuration is valid

$ sudo sshd -t
/etc/ssh/sshd_config line 42: Bad configuration option: InvalidOption
# Error found = fix before restarting
```

---

#### 4. Restart SSH Service
```bash
# Ubuntu/Debian
sudo systemctl restart sshd
# or
sudo systemctl restart ssh

# RHEL/CentOS/Fedora
sudo systemctl restart sshd
```
**Description:** Restarts SSH service to apply configuration changes. Use `restart` not `reload` for configuration changes.

---

#### 5. Check SSH Service Status
```bash
sudo systemctl status sshd
```
**Description:** Verifies SSH service is running properly after restart.

**Example output:**
```bash
● sshd.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
     Active: active (running) since Tue 2026-06-17 10:30:00 UTC
```

---

#### 6. View Effective SSH Configuration
```bash
sudo sshd -T
```
**Description:** Shows the effective SSH configuration (what's actually being used).

**Check specific setting:**
```bash
sudo sshd -T | grep permitrootlogin
# Output: permitrootlogin no
```

---

#### 7. Backup SSH Configuration
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```
**Description:** Creates a backup of SSH configuration before making changes. Always do this first!

**With timestamp:**
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d)
```

---

#### 8. View SSH Logs
```bash
# Ubuntu/Debian
sudo tail -f /var/log/auth.log

# RHEL/CentOS
sudo tail -f /var/log/secure

# Using journalctl (systemd)
sudo journalctl -u sshd -f
```
**Description:** Monitors SSH authentication logs in real-time. Useful for troubleshooting and security monitoring.

---

#### 9. Check Current SSH Configuration
```bash
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
```
**Description:** Quickly check if root login is enabled or disabled.

**Example:**
```bash
$ sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
PermitRootLogin no
```

---

#### 10. Test SSH Connection
```bash
# Test root login (should fail after disabling)
ssh root@server_ip

# Test regular user login (should work)
ssh username@server_ip
```
**Description:** Verify that root SSH is actually disabled and regular users can still login.

---

## 🎯 Complete Workflow

### Step-by-Step: Disable Root SSH Login

```bash
# 1. Backup configuration
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# 2. Edit configuration
sudo nano /etc/ssh/sshd_config

# 3. Find and change this line:
#    PermitRootLogin yes
#    to:
#    PermitRootLogin no

# 4. Save and exit (Ctrl+X, Y, Enter in nano)

# 5. Test configuration
sudo sshd -t

# 6. If no errors, restart SSH
sudo systemctl restart sshd

# 7. Verify service is running
sudo systemctl status sshd

# 8. Check effective configuration
sudo sshd -T | grep permitrootlogin

# 9. Test in NEW terminal (keep current one open!)
ssh root@localhost
# Should see: Permission denied
```

---

## 📋 Quick Command Reference Table

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `sudo nano /etc/ssh/sshd_config` | Edit SSH config | Making changes |
| `sudo sshd -t` | Test config syntax | Before restarting |
| `sudo systemctl restart sshd` | Restart SSH service | Apply changes |
| `sudo systemctl status sshd` | Check service status | Verify running |
| `sudo sshd -T \| grep permitrootlogin` | Check effective config | Verify setting |
| `sudo grep "^PermitRootLogin" /etc/ssh/sshd_config` | Check config file | Quick check |
| `sudo tail -f /var/log/auth.log` | View SSH logs | Troubleshooting |
| `ssh root@server` | Test root login | Verify disabled |

---

## ⚠️ Critical Safety Steps

### BEFORE Disabling Root SSH:

1. **Create admin user with sudo access:**
   ```bash
   sudo useradd -m -s /bin/bash admin
   sudo passwd admin
   sudo usermod -aG sudo admin    # Ubuntu/Debian
   # or
   sudo usermod -aG wheel admin   # RHEL/CentOS
   ```

2. **Test admin user can login and use sudo:**
   ```bash
   # In new terminal
   ssh admin@server_ip
   sudo whoami    # Should return: root
   ```

3. **Keep one SSH session open:**
   - Don't close your current SSH session
   - Test changes in a NEW terminal
   - Only close original session after confirming access works

---

## 🔍 Verification Checklist

```bash
# ✅ 1. Configuration file updated
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
# Expected: PermitRootLogin no

# ✅ 2. Configuration syntax valid
sudo sshd -t
# Expected: No output (means valid)

# ✅ 3. Effective configuration correct
sudo sshd -T | grep permitrootlogin
# Expected: permitrootlogin no

# ✅ 4. SSH service running
sudo systemctl is-active sshd
# Expected: active

# ✅ 5. Root SSH login blocked
ssh root@localhost
# Expected: Permission denied

# ✅ 6. Regular user can login
ssh admin@localhost
# Expected: Successful login
```

---

## 🔐 Additional Security Settings

### Recommended SSH Hardening

```bash
# Edit /etc/ssh/sshd_config and add/modify:

# 1. Disable root login
PermitRootLogin no

# 2. Disable password authentication (use keys)
PasswordAuthentication no
PubkeyAuthentication yes

# 3. Disable empty passwords
PermitEmptyPasswords no

# 4. Limit authentication attempts
MaxAuthTries 3

# 5. Set login grace time
LoginGraceTime 60

# 6. Disable X11 forwarding
X11Forwarding no

# 7. Allow only specific users
AllowUsers admin user1 user2

# 8. Change default port (optional)
Port 2222
```

---

## 🐛 Common Issues & Solutions

### Issue 1: Locked Out After Disabling Root
**Solution:** Use console access (physical or cloud provider console)
```bash
# Via console, login as root
# Re-enable root SSH temporarily
sudo sed -i 's/^PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### Issue 2: SSH Service Won't Start
**Solution:** Check configuration and restore backup
```bash
# Test configuration
sudo sshd -t

# View errors
sudo journalctl -xeu sshd

# Restore backup
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### Issue 3: Changes Not Taking Effect
**Solution:** Use restart, not reload
```bash
# Wrong
sudo systemctl reload sshd

# Correct
sudo systemctl restart sshd
```

---

## 💡 Pro Tips

1. **Always backup first:**
   ```bash
   sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
   ```

2. **Test before restarting:**
   ```bash
   sudo sshd -t
   ```

3. **Keep one session open:**
   - Test changes in NEW terminal
   - Don't close original until verified

4. **Use effective config check:**
   ```bash
   sudo sshd -T | grep permitrootlogin
   ```

5. **Monitor logs during testing:**
   ```bash
   sudo tail -f /var/log/auth.log
   ```

---

## 📊 Configuration File Location

| Distribution | SSH Config File | Log File |
|--------------|----------------|----------|
| Ubuntu/Debian | `/etc/ssh/sshd_config` | `/var/log/auth.log` |
| RHEL/CentOS | `/etc/ssh/sshd_config` | `/var/log/secure` |
| Fedora | `/etc/ssh/sshd_config` | `/var/log/secure` |
| All (systemd) | `/etc/ssh/sshd_config` | `journalctl -u sshd` |

---

## 🎓 Key Takeaways

1. **Edit:** `/etc/ssh/sshd_config`
2. **Change:** `PermitRootLogin no`
3. **Test:** `sudo sshd -t`
4. **Restart:** `sudo systemctl restart sshd`
5. **Verify:** `sudo sshd -T | grep permitrootlogin`
6. **Safety:** Keep one SSH session open while testing
7. **Prerequisite:** Create admin user with sudo access first

---

## 🔄 Quick One-Liner Commands

```bash
# Check if root login is disabled
sudo sshd -T | grep permitrootlogin

# Disable root login (backup, edit, restart)
sudo cp /etc/ssh/sshd_config{,.backup} && \
sudo sed -i 's/^PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config && \
sudo sshd -t && \
sudo systemctl restart sshd

# View last 20 failed SSH attempts
sudo grep "Failed password" /var/log/auth.log | tail -20

# Count failed root login attempts today
sudo grep "Failed password for root" /var/log/auth.log | \
grep "$(date '+%b %d')" | wc -l
```

---

## 🛡️ Security Best Practices

1. ✅ Disable root SSH login
2. ✅ Use SSH keys instead of passwords
3. ✅ Change default SSH port
4. ✅ Limit user access with AllowUsers
5. ✅ Set MaxAuthTries to 3
6. ✅ Install and configure fail2ban
7. ✅ Enable SSH logging
8. ✅ Regular security audits
9. ✅ Keep SSH software updated
10. ✅ Use strong passwords/keys

---

**Last Updated:** 2026-06-17