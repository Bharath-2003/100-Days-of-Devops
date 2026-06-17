# Day 003 - Secure Root SSH Access

**Date:** 2026-06-17  
**Topic:** SSH Security - Disabling Root Login

## 📝 Key Concepts

### SSH (Secure Shell)
- Protocol for secure remote access to Linux servers
- Encrypted communication between client and server
- Default port: 22
- Configuration file: `/etc/ssh/sshd_config`

### Root User Security Risk
- Root user has unlimited privileges
- Direct root login is a major security vulnerability
- Attackers commonly target root account
- Brute force attacks often target root@server

### Why Disable Root SSH Login?

**Security Risks of Root SSH:**
1. **Single Point of Failure:** If root password is compromised, attacker has full control
2. **No Accountability:** Cannot track which admin performed actions
3. **Brute Force Target:** Root is a known username, only password needs guessing
4. **No Audit Trail:** Actions performed as root are not individually traceable

**Best Practice Approach:**
1. Create individual admin users
2. Grant sudo access to admin users
3. Disable direct root SSH login
4. Use SSH keys instead of passwords
5. All actions logged with individual user identity

## 💡 Important Points

- Disabling root SSH login does NOT disable the root account
- Root can still be accessed via `sudo` or `su` from regular users
- Changes require SSH service restart to take effect
- Always test with a new session before closing current SSH connection
- Keep at least one SSH session open while testing changes

## 🔧 Commands/Tools Used

### Primary Configuration File
```bash
# SSH server configuration
/etc/ssh/sshd_config
```

### Key Configuration Directive
```bash
# In /etc/ssh/sshd_config
PermitRootLogin no
```

### Edit SSH Configuration
```bash
# Edit SSH config file
sudo nano /etc/ssh/sshd_config
# or
sudo vi /etc/ssh/sshd_config

# Find and modify:
PermitRootLogin yes    # Change to
PermitRootLogin no     # This disables root SSH login
```

### Restart SSH Service
```bash
# Ubuntu/Debian
sudo systemctl restart sshd
# or
sudo systemctl restart ssh

# RHEL/CentOS
sudo systemctl restart sshd

# Verify service status
sudo systemctl status sshd
```

### Test SSH Configuration
```bash
# Test config syntax before restarting
sudo sshd -t

# Check SSH service status
sudo systemctl status sshd

# View SSH logs
sudo tail -f /var/log/auth.log        # Ubuntu/Debian
sudo tail -f /var/log/secure          # RHEL/CentOS
```

## 📊 SSH Configuration Options

### PermitRootLogin Values

| Value | Description | Security Level |
|-------|-------------|----------------|
| `yes` | Root can login with password | ❌ Least Secure |
| `prohibit-password` | Root can login with SSH keys only | ⚠️ Moderate |
| `forced-commands-only` | Root can only run specific commands | ⚠️ Moderate |
| `no` | Root cannot login via SSH at all | ✅ Most Secure |

### Recommended Configuration
```bash
# /etc/ssh/sshd_config

# Disable root login
PermitRootLogin no

# Disable password authentication (use keys)
PasswordAuthentication no

# Enable public key authentication
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Change default port (optional)
Port 2222

# Limit user access
AllowUsers admin user1 user2
```

## ❓ Questions & Clarifications

- Q: If I disable root SSH, how do I perform admin tasks?
  A: Login as regular user with sudo privileges, then use `sudo` for admin commands.

- Q: What if I lock myself out?
  A: Always keep one SSH session open while testing. Use console access if locked out.

- Q: Does this affect local root login?
  A: No, you can still login as root locally or use `su -` from another user.

- Q: Can I still use `sudo su -` to become root?
  A: Yes, disabling root SSH only prevents direct SSH login as root.

## 🎯 Key Takeaways

1. Edit `/etc/ssh/sshd_config` and set `PermitRootLogin no`
2. Always test configuration with `sudo sshd -t` before restarting
3. Restart SSH service: `sudo systemctl restart sshd`
4. Keep one SSH session open while testing changes
5. Create admin users with sudo access before disabling root SSH
6. Use SSH keys instead of passwords for better security

## 🔗 Related Topics

- SSH key-based authentication
- SSH port forwarding and tunneling
- Fail2ban for brute force protection
- Two-factor authentication (2FA) for SSH
- SSH bastion hosts and jump servers
- SELinux/AppArmor for SSH hardening

---
**Next Steps:** Learn about SSH key-based authentication and fail2ban