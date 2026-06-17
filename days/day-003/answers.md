# Day 003 - Answers & Solutions

**Date:** 2026-06-17  
**Topic:** Secure Root SSH Access - Disable Root Login

## 📋 Lab Exercises

### Exercise 1: Disable Root SSH Login

**Question:**
```
Disable direct root SSH login to improve server security.
```

**Answer:**
```bash
# Step 1: Backup the SSH configuration file
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Step 2: Edit the SSH configuration
sudo nano /etc/ssh/sshd_config

# Step 3: Find and modify the PermitRootLogin directive
# Change from:
#PermitRootLogin yes
# To:
PermitRootLogin no

# Step 4: Test the configuration
sudo sshd -t

# Step 5: Restart SSH service
sudo systemctl restart sshd

# Step 6: Verify service is running
sudo systemctl status sshd
```

**Explanation:**
- Step 1: Always backup configuration files before making changes
- Step 2: Edit `/etc/ssh/sshd_config` with your preferred editor
- Step 3: Set `PermitRootLogin no` to disable root SSH login
- Step 4: Test configuration syntax to catch errors before restart
- Step 5: Restart SSH service to apply changes
- Step 6: Verify SSH service is running properly

**Output:**
```bash
# After testing configuration
$ sudo sshd -t
# No output means configuration is valid

# After restarting service
$ sudo systemctl status sshd
● sshd.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled)
     Active: active (running) since Tue 2026-06-17 10:30:00 UTC
```

---

### Exercise 2: Verify Root SSH is Disabled

**Question:**
```
How do you verify that root SSH login is actually disabled?
```

**Answer:**
```bash
# Method 1: Check the configuration file
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config

# Method 2: Test SSH connection as root (from another machine)
ssh root@server_ip
# Should see: Permission denied

# Method 3: Check SSH logs
sudo tail -f /var/log/auth.log | grep root    # Ubuntu/Debian
sudo tail -f /var/log/secure | grep root      # RHEL/CentOS

# Method 4: Use sshd test mode
sudo sshd -T | grep permitrootlogin
```

**Explanation:**
- Method 1 shows the actual configuration setting
- Method 2 tests real-world SSH connection attempt
- Method 3 monitors authentication logs for root login attempts
- Method 4 displays the effective configuration

**Output:**
```bash
$ sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
PermitRootLogin no

$ ssh root@192.168.1.100
root@192.168.1.100's password: 
Permission denied, please try again.

$ sudo sshd -T | grep permitrootlogin
permitrootlogin no
```

---

### Exercise 3: Create Admin User Before Disabling Root

**Question:**
```
What's the proper workflow to safely disable root SSH access?
```

**Answer:**
```bash
# Step 1: Create admin user
sudo useradd -m -s /bin/bash admin

# Step 2: Set strong password
sudo passwd admin

# Step 3: Grant sudo privileges
sudo usermod -aG sudo admin    # Ubuntu/Debian
# or
sudo usermod -aG wheel admin   # RHEL/CentOS

# Step 4: Test sudo access
su - admin
sudo whoami    # Should return: root

# Step 5: Test SSH login as admin (from another terminal)
ssh admin@server_ip

# Step 6: Once confirmed admin can login and use sudo, disable root SSH
sudo nano /etc/ssh/sshd_config
# Set: PermitRootLogin no

# Step 7: Test configuration and restart
sudo sshd -t
sudo systemctl restart sshd

# Step 8: Verify root SSH is blocked
ssh root@server_ip    # Should fail
```

**Explanation:**
- Always create and test admin user BEFORE disabling root SSH
- Verify admin can login and use sudo
- Keep one SSH session open while testing
- Only disable root SSH after confirming admin access works

---

### Exercise 4: Restore Root SSH Access (Emergency)

**Question:**
```
You accidentally locked yourself out. How do you restore root SSH access?
```

**Answer:**
```bash
# Option 1: Using console access (physical or cloud console)
# Login via console as root or admin user

# Restore from backup
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart sshd

# Option 2: Edit configuration via console
sudo nano /etc/ssh/sshd_config
# Change: PermitRootLogin no
# To: PermitRootLogin yes

sudo systemctl restart sshd

# Option 3: Using existing SSH session (if still connected)
# If you have an active SSH session, use it to fix the config
sudo nano /etc/ssh/sshd_config
# Make necessary changes
sudo systemctl restart sshd
```

**Lesson Learned:** Always keep one SSH session open when making SSH configuration changes!

---

### Exercise 5: Enhanced SSH Security Configuration

**Question:**
```
What additional SSH security measures should be implemented?
```

**Answer:**
```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Recommended security settings:

# 1. Disable root login
PermitRootLogin no

# 2. Disable password authentication (use keys only)
PasswordAuthentication no
PubkeyAuthentication yes

# 3. Disable empty passwords
PermitEmptyPasswords no

# 4. Change default port (optional)
Port 2222

# 5. Limit login attempts
MaxAuthTries 3

# 6. Set login grace time
LoginGraceTime 60

# 7. Disable X11 forwarding if not needed
X11Forwarding no

# 8. Allow only specific users
AllowUsers admin user1 user2

# 9. Enable strict mode
StrictModes yes

# 10. Disable host-based authentication
HostbasedAuthentication no

# Test and restart
sudo sshd -t
sudo systemctl restart sshd
```

**Explanation:**
- Each setting adds a layer of security
- Combine multiple measures for defense in depth
- Always test configuration before restarting service

---

## 🧪 Practice Problems

### Problem 1: Automated SSH Hardening Script

**Description:**
Create a script to automatically harden SSH configuration.

**Solution:**
```bash
#!/bin/bash
# SSH Hardening Script

CONFIG_FILE="/etc/ssh/sshd_config"
BACKUP_FILE="/etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)"

echo "SSH Hardening Script"
echo "===================="

# Backup current configuration
echo "Creating backup: $BACKUP_FILE"
sudo cp $CONFIG_FILE $BACKUP_FILE

# Function to update or add configuration
update_config() {
    local key=$1
    local value=$2
    
    if sudo grep -q "^${key}" $CONFIG_FILE; then
        sudo sed -i "s/^${key}.*/${key} ${value}/" $CONFIG_FILE
        echo "Updated: ${key} ${value}"
    else
        echo "${key} ${value}" | sudo tee -a $CONFIG_FILE > /dev/null
        echo "Added: ${key} ${value}"
    fi
}

# Apply security settings
update_config "PermitRootLogin" "no"
update_config "PasswordAuthentication" "no"
update_config "PubkeyAuthentication" "yes"
update_config "PermitEmptyPasswords" "no"
update_config "MaxAuthTries" "3"
update_config "LoginGraceTime" "60"
update_config "X11Forwarding" "no"
update_config "StrictModes" "yes"

# Test configuration
echo ""
echo "Testing configuration..."
if sudo sshd -t; then
    echo "✓ Configuration is valid"
    
    # Restart SSH service
    echo "Restarting SSH service..."
    sudo systemctl restart sshd
    
    if sudo systemctl is-active --quiet sshd; then
        echo "✓ SSH service restarted successfully"
        echo ""
        echo "SSH hardening complete!"
        echo "Backup saved to: $BACKUP_FILE"
    else
        echo "✗ Failed to restart SSH service"
        echo "Restoring backup..."
        sudo cp $BACKUP_FILE $CONFIG_FILE
        sudo systemctl restart sshd
    fi
else
    echo "✗ Configuration has errors"
    echo "Restoring backup..."
    sudo cp $BACKUP_FILE $CONFIG_FILE
fi
```

---

### Problem 2: SSH Login Monitoring Script

**Description:**
Monitor and alert on failed root SSH login attempts.

**Solution:**
```bash
#!/bin/bash
# Monitor failed root SSH login attempts

LOG_FILE="/var/log/auth.log"  # Ubuntu/Debian
# LOG_FILE="/var/log/secure"  # RHEL/CentOS

ALERT_EMAIL="admin@example.com"
THRESHOLD=5

echo "Monitoring failed root SSH login attempts..."

# Count failed root login attempts in last hour
FAILED_ATTEMPTS=$(sudo grep "Failed password for root" $LOG_FILE | \
                  grep "$(date '+%b %d %H')" | wc -l)

echo "Failed root login attempts in last hour: $FAILED_ATTEMPTS"

if [ $FAILED_ATTEMPTS -ge $THRESHOLD ]; then
    echo "⚠️  ALERT: $FAILED_ATTEMPTS failed root login attempts detected!"
    
    # Get IP addresses
    IPS=$(sudo grep "Failed password for root" $LOG_FILE | \
          grep "$(date '+%b %d %H')" | \
          awk '{print $(NF-3)}' | sort | uniq -c | sort -rn)
    
    # Send email alert
    echo "Failed root SSH login attempts: $FAILED_ATTEMPTS" | \
        mail -s "SSH Security Alert" $ALERT_EMAIL
    
    echo "Alert sent to $ALERT_EMAIL"
    echo "Attacking IPs:"
    echo "$IPS"
else
    echo "✓ No suspicious activity detected"
fi
```

---

### Problem 3: SSH Configuration Audit Script

**Description:**
Audit SSH configuration for security best practices.

**Solution:**
```bash
#!/bin/bash
# SSH Security Audit Script

CONFIG_FILE="/etc/ssh/sshd_config"
SCORE=0
MAX_SCORE=10

echo "SSH Security Audit"
echo "=================="
echo ""

# Function to check configuration
check_config() {
    local key=$1
    local expected=$2
    local description=$3
    
    actual=$(sudo sshd -T | grep "^${key}" | awk '{print $2}')
    
    if [ "$actual" == "$expected" ]; then
        echo "✓ $description"
        ((SCORE++))
    else
        echo "✗ $description (Current: $actual, Expected: $expected)"
    fi
}

# Perform checks
check_config "permitrootlogin" "no" "Root login disabled"
check_config "passwordauthentication" "no" "Password auth disabled"
check_config "pubkeyauthentication" "yes" "Public key auth enabled"
check_config "permitemptypasswords" "no" "Empty passwords disabled"
check_config "maxauthtries" "3" "Max auth tries limited"
check_config "x11forwarding" "no" "X11 forwarding disabled"
check_config "strictmodes" "yes" "Strict modes enabled"
check_config "hostbasedauthentication" "no" "Host-based auth disabled"

# Check if default port is changed
PORT=$(sudo sshd -T | grep "^port" | awk '{print $2}')
if [ "$PORT" != "22" ]; then
    echo "✓ Default port changed"
    ((SCORE++))
else
    echo "⚠ Default port (22) in use"
fi

# Check if AllowUsers is set
if sudo grep -q "^AllowUsers" $CONFIG_FILE; then
    echo "✓ User access restricted"
    ((SCORE++))
else
    echo "⚠ No user access restrictions"
fi

echo ""
echo "Security Score: $SCORE/$MAX_SCORE"

if [ $SCORE -eq $MAX_SCORE ]; then
    echo "🎉 Excellent! SSH is properly secured."
elif [ $SCORE -ge 7 ]; then
    echo "👍 Good security posture, minor improvements needed."
elif [ $SCORE -ge 5 ]; then
    echo "⚠️  Moderate security, several improvements recommended."
else
    echo "❌ Poor security! Immediate action required."
fi
```

---

## 🐛 Troubleshooting

### Issue 1: Locked Out After Disabling Root SSH
**Problem:** Cannot SSH to server after disabling root login

**Solution:**
```bash
# Use console access (physical or cloud provider console)
# Login as root via console

# Check SSH configuration
sudo grep "PermitRootLogin" /etc/ssh/sshd_config

# Temporarily enable root SSH
sudo sed -i 's/^PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Create admin user with sudo access
sudo useradd -m -s /bin/bash admin
sudo passwd admin
sudo usermod -aG sudo admin

# Test admin access, then disable root SSH again
```

**Lesson Learned:** Always create and test admin user before disabling root SSH

---

### Issue 2: SSH Service Won't Start After Configuration Change
**Problem:** SSH service fails to start after editing sshd_config

**Solution:**
```bash
# Check service status
sudo systemctl status sshd

# Test configuration for syntax errors
sudo sshd -t

# View detailed error messages
sudo journalctl -xeu sshd

# Restore from backup
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart sshd
```

**Lesson Learned:** Always test configuration with `sshd -t` before restarting service

---

### Issue 3: Changes Not Taking Effect
**Problem:** Modified sshd_config but changes don't apply

**Solution:**
```bash
# Ensure you edited the correct file
ls -l /etc/ssh/sshd_config

# Check for syntax errors
sudo sshd -t

# Restart service (not reload)
sudo systemctl restart sshd

# Verify effective configuration
sudo sshd -T | grep permitrootlogin

# Check if service restarted successfully
sudo systemctl status sshd
```

**Lesson Learned:** Use `restart` not `reload`, and verify with `sshd -T`

---

## ✅ Verification Steps

1. **Verify configuration change:**
   ```bash
   sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
   ```

2. **Test configuration syntax:**
   ```bash
   sudo sshd -t
   ```

3. **Check effective configuration:**
   ```bash
   sudo sshd -T | grep permitrootlogin
   ```

4. **Verify service is running:**
   ```bash
   sudo systemctl status sshd
   ```

5. **Test root SSH login (should fail):**
   ```bash
   ssh root@server_ip
   # Should see: Permission denied
   ```

6. **Test admin user SSH login (should work):**
   ```bash
   ssh admin@server_ip
   # Should succeed
   ```

7. **Monitor SSH logs:**
   ```bash
   sudo tail -f /var/log/auth.log    # Ubuntu/Debian
   sudo tail -f /var/log/secure      # RHEL/CentOS
   ```

## 📌 Notes

- Always backup `/etc/ssh/sshd_config` before making changes
- Test configuration with `sudo sshd -t` before restarting
- Keep one SSH session open while testing changes
- Create admin user with sudo access before disabling root SSH
- Use `systemctl restart sshd` not `reload` for configuration changes
- Monitor `/var/log/auth.log` for failed login attempts
- Consider implementing fail2ban for additional protection

## 🔐 Security Best Practices

1. **Before disabling root SSH:**
   - Create admin user with sudo access
   - Test admin user can login and use sudo
   - Keep one SSH session open

2. **SSH hardening checklist:**
   - ✅ Disable root login
   - ✅ Disable password authentication
   - ✅ Use SSH keys only
   - ✅ Change default port
   - ✅ Limit user access
   - ✅ Set max auth tries
   - ✅ Enable fail2ban

3. **Regular maintenance:**
   - Review SSH logs weekly
   - Update SSH server regularly
   - Audit user access quarterly
   - Test backup/restore procedures

---

**Status:** ✅ Completed