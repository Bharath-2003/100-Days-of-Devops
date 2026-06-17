# Day 003 - Resources & References

**Date:** 2026-06-17  
**Topic:** Secure Root SSH Access

## 📚 Course Materials

- **Video Link:** KodeKloud - 100 Days of DevOps - Day 3
- **Duration:** ~20-25 minutes
- **Difficulty:** Beginner to Intermediate

## 🔗 Official Documentation

- [OpenSSH Manual](https://www.openssh.com/manual.html) - Complete SSH documentation
- [sshd_config Manual](https://man.openbsd.org/sshd_config) - SSH daemon configuration
- [SSH Protocol RFC](https://www.rfc-editor.org/rfc/rfc4253.html) - SSH protocol specification
- [systemctl Manual](https://man7.org/linux/man-pages/man1/systemctl.1.html) - Service management

## 📖 Additional Reading

### Articles
- [SSH Security Best Practices](https://www.ssh.com/academy/ssh/security) - Comprehensive security guide
- [Hardening SSH](https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html) - CyberCiti hardening guide
- [Why Disable Root SSH](https://www.redhat.com/sysadmin/disable-root-login) - Red Hat explanation
- [SSH Attack Prevention](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-20-04) - DigitalOcean tutorial

### Tutorials
- [SSH Configuration Guide](https://www.ssh.com/academy/ssh/sshd_config) - Complete configuration reference
- [Securing SSH](https://www.linode.com/docs/guides/securing-your-server/) - Linode security guide
- [SSH Key Authentication](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2) - Key-based auth tutorial

## 🎥 Video Resources

- [SSH Security Explained](https://www.youtube.com/results?search_query=ssh+security+best+practices) - YouTube tutorials
- [Linux SSH Hardening](https://www.youtube.com/results?search_query=linux+ssh+hardening) - Video guides
- [Disable Root SSH Login](https://www.youtube.com/results?search_query=disable+root+ssh+login) - Step-by-step videos

## 🛠️ Tools & Software

| Tool | Purpose | Installation Link |
|------|---------|------------------|
| OpenSSH | SSH server and client | Pre-installed on most Linux |
| fail2ban | Brute force protection | `apt install fail2ban` |
| ssh-audit | SSH security scanner | [GitHub](https://github.com/jtesta/ssh-audit) |
| lynis | Security auditing tool | `apt install lynis` |
| ufw | Firewall management | `apt install ufw` |

## 📝 Cheat Sheets

### SSH Configuration Commands
```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
sudo vi /etc/ssh/sshd_config

# Test configuration
sudo sshd -t

# View effective configuration
sudo sshd -T

# Restart SSH service
sudo systemctl restart sshd
sudo systemctl restart ssh

# Check service status
sudo systemctl status sshd
sudo systemctl is-active sshd

# View SSH logs
sudo tail -f /var/log/auth.log        # Ubuntu/Debian
sudo tail -f /var/log/secure          # RHEL/CentOS
sudo journalctl -u sshd -f            # systemd

# Backup configuration
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### Key Configuration Directives
```bash
# Disable root login
PermitRootLogin no

# Disable password authentication
PasswordAuthentication no

# Enable public key authentication
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Change default port
Port 2222

# Limit authentication attempts
MaxAuthTries 3

# Set login grace time
LoginGraceTime 60

# Disable X11 forwarding
X11Forwarding no

# Allow specific users only
AllowUsers admin user1 user2

# Deny specific users
DenyUsers baduser

# Allow specific groups
AllowGroups sshusers admins

# Enable strict mode
StrictModes yes
```

### SSH Connection Commands
```bash
# Connect to server
ssh username@hostname
ssh username@192.168.1.100

# Connect with specific port
ssh -p 2222 username@hostname

# Connect with specific key
ssh -i ~/.ssh/id_rsa username@hostname

# Verbose output (debugging)
ssh -v username@hostname
ssh -vv username@hostname
ssh -vvv username@hostname

# Copy files (SCP)
scp file.txt username@hostname:/path/
scp username@hostname:/path/file.txt .

# Copy files (SFTP)
sftp username@hostname
```

## 💻 Code Examples

### Example 1: Complete SSH Hardening Configuration
```bash
# /etc/ssh/sshd_config - Hardened Configuration

# Network
Port 2222
AddressFamily inet
ListenAddress 0.0.0.0

# Authentication
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
UsePAM yes

# Access Control
AllowUsers admin devops
AllowGroups sshusers
MaxAuthTries 3
MaxSessions 2

# Security
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Timeouts
LoginGraceTime 60
ClientAliveInterval 300
ClientAliveCountMax 2

# Features
X11Forwarding no
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
PermitUserEnvironment no
Compression delayed
UseDNS no
PermitTunnel no
AllowAgentForwarding no
AllowTcpForwarding no
GatewayPorts no
PermitUserRC no
```

### Example 2: SSH Hardening Script
```bash
#!/bin/bash
# Comprehensive SSH Hardening Script

set -e

CONFIG="/etc/ssh/sshd_config"
BACKUP="/etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)"

echo "=== SSH Hardening Script ==="
echo ""

# Check if running as root
if [ "$EUID" -ne 0 ]; then 
    echo "Please run as root or with sudo"
    exit 1
fi

# Backup current configuration
echo "Creating backup: $BACKUP"
cp $CONFIG $BACKUP

# Function to set configuration
set_config() {
    local key=$1
    local value=$2
    
    if grep -q "^${key}" $CONFIG; then
        sed -i "s/^${key}.*/${key} ${value}/" $CONFIG
    elif grep -q "^#${key}" $CONFIG; then
        sed -i "s/^#${key}.*/${key} ${value}/" $CONFIG
    else
        echo "${key} ${value}" >> $CONFIG
    fi
    echo "✓ Set ${key} = ${value}"
}

echo ""
echo "Applying security settings..."

# Core security settings
set_config "PermitRootLogin" "no"
set_config "PasswordAuthentication" "no"
set_config "PubkeyAuthentication" "yes"
set_config "PermitEmptyPasswords" "no"
set_config "MaxAuthTries" "3"
set_config "LoginGraceTime" "60"
set_config "X11Forwarding" "no"
set_config "StrictModes" "yes"
set_config "HostbasedAuthentication" "no"
set_config "IgnoreRhosts" "yes"
set_config "Protocol" "2"
set_config "ClientAliveInterval" "300"
set_config "ClientAliveCountMax" "2"
set_config "LogLevel" "VERBOSE"

echo ""
echo "Testing configuration..."
if sshd -t; then
    echo "✓ Configuration is valid"
    
    echo ""
    echo "Restarting SSH service..."
    systemctl restart sshd
    
    if systemctl is-active --quiet sshd; then
        echo "✓ SSH service restarted successfully"
        echo ""
        echo "=== SSH Hardening Complete ==="
        echo "Backup saved to: $BACKUP"
        echo ""
        echo "⚠️  IMPORTANT:"
        echo "1. Ensure you have SSH key authentication set up"
        echo "2. Test SSH login in a new terminal before closing this one"
        echo "3. Keep this terminal open until you verify access"
    else
        echo "✗ Failed to restart SSH service"
        echo "Restoring backup..."
        cp $BACKUP $CONFIG
        systemctl restart sshd
        exit 1
    fi
else
    echo "✗ Configuration has errors"
    echo "Restoring backup..."
    cp $BACKUP $CONFIG
    exit 1
fi
```

### Example 3: SSH Key Setup Script
```bash
#!/bin/bash
# Setup SSH key authentication for user

USERNAME=$1
SERVER=$2

if [ -z "$USERNAME" ] || [ -z "$SERVER" ]; then
    echo "Usage: $0 <username> <server>"
    echo "Example: $0 admin 192.168.1.100"
    exit 1
fi

echo "Setting up SSH key authentication"
echo "User: $USERNAME"
echo "Server: $SERVER"
echo ""

# Generate SSH key if doesn't exist
if [ ! -f ~/.ssh/id_rsa ]; then
    echo "Generating SSH key pair..."
    ssh-keygen -t rsa -b 4096 -C "$USERNAME@$(hostname)" -f ~/.ssh/id_rsa -N ""
    echo "✓ SSH key generated"
else
    echo "✓ SSH key already exists"
fi

# Copy public key to server
echo ""
echo "Copying public key to server..."
ssh-copy-id -i ~/.ssh/id_rsa.pub $USERNAME@$SERVER

if [ $? -eq 0 ]; then
    echo "✓ Public key copied successfully"
    echo ""
    echo "Testing SSH connection..."
    ssh -o BatchMode=yes -o ConnectTimeout=5 $USERNAME@$SERVER "echo '✓ SSH key authentication working!'"
    
    if [ $? -eq 0 ]; then
        echo ""
        echo "=== Setup Complete ==="
        echo "You can now login without password:"
        echo "ssh $USERNAME@$SERVER"
    fi
else
    echo "✗ Failed to copy public key"
    exit 1
fi
```

### Example 4: SSH Login Monitor
```bash
#!/bin/bash
# Monitor SSH login attempts and send alerts

LOG_FILE="/var/log/auth.log"  # Ubuntu/Debian
ALERT_EMAIL="admin@example.com"
THRESHOLD=5
CHECK_INTERVAL=300  # 5 minutes

while true; do
    # Count failed attempts in last 5 minutes
    FAILED=$(grep "Failed password" $LOG_FILE | \
             grep "$(date --date='5 minutes ago' '+%b %d %H:%M')" | \
             wc -l)
    
    if [ $FAILED -ge $THRESHOLD ]; then
        # Get attacking IPs
        IPS=$(grep "Failed password" $LOG_FILE | \
              grep "$(date --date='5 minutes ago' '+%b %d %H:%M')" | \
              awk '{print $(NF-3)}' | sort | uniq -c | sort -rn)
        
        # Send alert
        {
            echo "SSH Security Alert"
            echo "=================="
            echo ""
            echo "Failed login attempts: $FAILED"
            echo "Time: $(date)"
            echo ""
            echo "Attacking IPs:"
            echo "$IPS"
        } | mail -s "SSH Security Alert - $FAILED failed attempts" $ALERT_EMAIL
        
        logger "SSH Security Alert: $FAILED failed login attempts"
    fi
    
    sleep $CHECK_INTERVAL
done
```

### Example 5: Fail2ban Configuration
```bash
# Install fail2ban
sudo apt install fail2ban -y

# Create local configuration
sudo tee /etc/fail2ban/jail.local > /dev/null <<EOF
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
destemail = admin@example.com
sendername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
EOF

# Start and enable fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Check status
sudo fail2ban-client status sshd
```

## 🌐 Community Resources

- **Forums:** 
  - [Unix & Linux Stack Exchange - SSH](https://unix.stackexchange.com/questions/tagged/ssh)
  - [Server Fault - SSH Security](https://serverfault.com/questions/tagged/ssh+security)
  - [r/linuxadmin](https://www.reddit.com/r/linuxadmin/)

- **GitHub Repositories:**
  - [SSH Hardening Scripts](https://github.com/topics/ssh-hardening)
  - [Security Automation Tools](https://github.com/topics/security-automation)
  - [ssh-audit](https://github.com/jtesta/ssh-audit) - SSH security scanner

## 📊 Practice Platforms

- [KodeKloud Labs](https://kodekloud.com) - Hands-on SSH labs
- [TryHackMe](https://tryhackme.com) - SSH security challenges
- [HackTheBox](https://www.hackthebox.com) - Advanced security practice

## 🎯 Related Topics to Explore

- SSH key-based authentication
- SSH certificate authorities
- Two-factor authentication (2FA) for SSH
- SSH bastion hosts and jump servers
- SSH tunneling and port forwarding
- fail2ban and intrusion prevention
- SELinux/AppArmor for SSH
- SSH audit logging and monitoring
- VPN vs SSH for remote access
- Zero Trust security model

## 📌 Bookmarks

- [SSH Security Best Practices](https://www.ssh.com/academy/ssh/security)
- [CIS Benchmarks for SSH](https://www.cisecurity.org/cis-benchmarks/)
- [NIST SSH Guidelines](https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final)
- [Mozilla SSH Guidelines](https://infosec.mozilla.org/guidelines/openssh)

## 🔍 Quick Reference

### Common SSH Security Checks

1. **Check current SSH configuration:**
   ```bash
   sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|pubkeyauthentication'
   ```

2. **Audit SSH security:**
   ```bash
   ssh-audit localhost
   ```

3. **Check active SSH connections:**
   ```bash
   who
   w
   netstat -tnpa | grep 'ESTABLISHED.*sshd'
   ```

4. **View failed login attempts:**
   ```bash
   sudo grep "Failed password" /var/log/auth.log | tail -20
   ```

5. **Check fail2ban status:**
   ```bash
   sudo fail2ban-client status sshd
   ```

## 🔐 Security Hardening Checklist

### Essential (Must Have)
- [ ] Disable root SSH login
- [ ] Use SSH keys instead of passwords
- [ ] Change default SSH port
- [ ] Limit user access with AllowUsers/AllowGroups
- [ ] Set MaxAuthTries to 3 or less
- [ ] Enable fail2ban or similar IPS

### Recommended
- [ ] Disable password authentication completely
- [ ] Use SSH protocol 2 only
- [ ] Set ClientAliveInterval and ClientAliveCountMax
- [ ] Disable X11 forwarding if not needed
- [ ] Enable verbose logging
- [ ] Implement 2FA for SSH
- [ ] Use SSH certificates

### Advanced
- [ ] Implement SSH bastion host
- [ ] Use port knocking
- [ ] Implement geo-blocking
- [ ] Set up centralized logging
- [ ] Use SSH CA for key management
- [ ] Implement session recording
- [ ] Regular security audits

## 📅 Maintenance Schedule

### Daily
- Monitor SSH logs for suspicious activity
- Check fail2ban status and banned IPs

### Weekly
- Review failed login attempts
- Update fail2ban rules if needed
- Check for SSH software updates

### Monthly
- Audit SSH configuration
- Review user access list
- Rotate SSH keys if needed
- Test backup/restore procedures

### Quarterly
- Full security audit
- Review and update security policies
- Test incident response procedures
- Update documentation

---
**Last Updated:** 2026-06-17