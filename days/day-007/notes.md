# Day 7: Password-less SSH Key-Based Authentication

## 📋 Task Description
Learn how to set up secure, password-less SSH authentication using public-private key pairs. This is essential for automated deployments, secure server access, and DevOps workflows.

## 🎯 Learning Objectives
- Understand SSH key-based authentication
- Generate SSH key pairs
- Configure password-less authentication
- Manage SSH keys securely
- Implement SSH best practices
- Troubleshoot SSH connection issues

---

## 📚 Comprehensive Notes

### What is SSH Key-Based Authentication?

**SSH (Secure Shell)** key-based authentication uses cryptographic key pairs instead of passwords to authenticate users. This method is more secure and convenient for automated processes.

**Key Components:**
- **Private Key**: Kept secret on the client machine (like a password)
- **Public Key**: Shared with servers you want to access (like a username)
- **Key Pair**: The combination of private and public keys

**How It Works:**
```
Client Machine                    Remote Server
┌─────────────────┐              ┌─────────────────┐
│  Private Key    │              │   Public Key    │
│  (id_rsa)       │              │ (authorized_keys)│
│                 │              │                 │
│  1. Request     │─────────────>│                 │
│     Connection  │              │  2. Challenge   │
│                 │<─────────────│     (encrypted) │
│  3. Sign with   │              │                 │
│     Private Key │─────────────>│  4. Verify with │
│                 │              │     Public Key  │
│                 │<─────────────│  5. Grant Access│
└─────────────────┘              └─────────────────┘
```

### Benefits of SSH Key Authentication

1. **Enhanced Security**
   - No password transmission over network
   - Resistant to brute-force attacks
   - Can use passphrase for additional security

2. **Convenience**
   - No need to remember passwords
   - Enables automation (scripts, CI/CD)
   - Single key for multiple servers

3. **Auditability**
   - Easy to track which keys have access
   - Simple to revoke access
   - Can add comments to identify keys

---

### SSH Key Types

**Common Key Algorithms:**

1. **RSA (Rivest-Shamir-Adleman)**
   - Most widely supported
   - Key size: 2048, 3072, or 4096 bits
   - Default and recommended: 4096 bits
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **Ed25519 (Edwards-curve)**
   - Modern, fast, and secure
   - Fixed key size: 256 bits
   - Recommended for new keys
   ```bash
   ssh-keygen -t ed25519
   ```

3. **ECDSA (Elliptic Curve)**
   - Good security with smaller keys
   - Key sizes: 256, 384, or 521 bits
   ```bash
   ssh-keygen -t ecdsa -b 521
   ```

4. **DSA (Digital Signature Algorithm)**
   - Deprecated, not recommended
   - Limited to 1024 bits
   - Avoid using

**Comparison:**
```
Algorithm | Security | Speed | Key Size | Compatibility
----------|----------|-------|----------|---------------
RSA       | Good     | Slow  | Large    | Excellent
Ed25519   | Excellent| Fast  | Small    | Good (modern)
ECDSA     | Good     | Fast  | Medium   | Good
DSA       | Weak     | Slow  | Small    | Legacy only
```

---

### Generating SSH Keys

**Basic Key Generation:**
```bash
# Generate RSA key (4096 bits)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate with custom filename
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_custom_key

# Generate without passphrase (for automation)
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/automation_key
```

**Interactive Process:**
```
$ ssh-keygen -t ed25519 -C "user@example.com"

Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519): [Press Enter]
Enter passphrase (empty for no passphrase): [Type passphrase or Press Enter]
Enter same passphrase again: [Type passphrase again or Press Enter]

Your identification has been saved in /home/user/.ssh/id_ed25519
Your public key has been saved in /home/user/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx user@example.com
The key's randomart image is:
+--[ED25519 256]--+
|                 |
|                 |
|      . .        |
|     . o .       |
|    . . S        |
|   . . o         |
|    . .          |
|                 |
|                 |
+----[SHA256]-----+
```

**Key Files Created:**
- `~/.ssh/id_ed25519` - Private key (keep secret!)
- `~/.ssh/id_ed25519.pub` - Public key (share this)

---

### Copying Public Key to Remote Server

**Method 1: Using ssh-copy-id (Recommended)**
```bash
# Copy key to remote server
ssh-copy-id user@remote-server

# Copy specific key
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@remote-server

# Copy to specific port
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 user@remote-server
```

**Method 2: Manual Copy**
```bash
# Display public key
cat ~/.ssh/id_ed25519.pub

# Copy and paste to remote server
ssh user@remote-server
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "your-public-key-content" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
exit
```

**Method 3: Using SCP**
```bash
# Copy public key file
scp ~/.ssh/id_ed25519.pub user@remote-server:~/

# SSH to server and append to authorized_keys
ssh user@remote-server
cat ~/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm ~/id_ed25519.pub
exit
```

**Method 4: One-liner**
```bash
cat ~/.ssh/id_ed25519.pub | ssh user@remote-server "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

---

### SSH Configuration

**Client Configuration (~/.ssh/config):**
```bash
# Global settings
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes

# Specific server configuration
Host myserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    
# Multiple servers with same key
Host server1 server2 server3
    User deploy
    IdentityFile ~/.ssh/deployment_key
    
# Jump host (bastion)
Host internal-server
    HostName 10.0.1.50
    User admin
    ProxyJump bastion-host
    
# Using wildcards
Host *.example.com
    User admin
    IdentityFile ~/.ssh/company_key
```

**Usage with config:**
```bash
# Instead of: ssh admin@192.168.1.100
ssh myserver

# Config makes it simple
ssh server1
ssh internal-server
```

**Server Configuration (/etc/ssh/sshd_config):**
```bash
# Enable public key authentication
PubkeyAuthentication yes

# Specify authorized keys file
AuthorizedKeysFile .ssh/authorized_keys

# Disable password authentication (after key setup)
PasswordAuthentication no

# Disable root login
PermitRootLogin no

# Disable empty passwords
PermitEmptyPasswords no

# Enable strict mode (check file permissions)
StrictModes yes

# Limit authentication attempts
MaxAuthTries 3

# Set login grace time
LoginGraceTime 60

# Specify allowed users
AllowUsers user1 user2 admin

# Or allowed groups
AllowGroups sshusers admins
```

**Apply SSH configuration changes:**
```bash
# Test configuration
sudo sshd -t

# Restart SSH service
sudo systemctl restart sshd
# or
sudo systemctl restart ssh
```

---

### File Permissions

**Critical Permissions:**
```bash
# SSH directory
chmod 700 ~/.ssh

# Private keys
chmod 600 ~/.ssh/id_*
chmod 600 ~/.ssh/id_*.pem

# Public keys
chmod 644 ~/.ssh/id_*.pub

# Authorized keys
chmod 600 ~/.ssh/authorized_keys

# Config file
chmod 600 ~/.ssh/config

# Known hosts
chmod 644 ~/.ssh/known_hosts
```

**Permission Summary:**
```
File/Directory              Permission  Octal
~/.ssh/                     drwx------  700
~/.ssh/id_rsa               -rw-------  600
~/.ssh/id_rsa.pub           -rw-r--r--  644
~/.ssh/authorized_keys      -rw-------  600
~/.ssh/config               -rw-------  600
~/.ssh/known_hosts          -rw-r--r--  644
```

**Fix permissions script:**
```bash
#!/bin/bash
# fix-ssh-permissions.sh

SSH_DIR="$HOME/.ssh"

# Create .ssh directory if it doesn't exist
mkdir -p "$SSH_DIR"

# Set directory permissions
chmod 700 "$SSH_DIR"

# Set private key permissions
find "$SSH_DIR" -type f -name "id_*" ! -name "*.pub" -exec chmod 600 {} \;

# Set public key permissions
find "$SSH_DIR" -type f -name "*.pub" -exec chmod 644 {} \;

# Set authorized_keys permissions
[ -f "$SSH_DIR/authorized_keys" ] && chmod 600 "$SSH_DIR/authorized_keys"

# Set config permissions
[ -f "$SSH_DIR/config" ] && chmod 600 "$SSH_DIR/config"

# Set known_hosts permissions
[ -f "$SSH_DIR/known_hosts" ] && chmod 644 "$SSH_DIR/known_hosts"

echo "SSH permissions fixed!"
```

---

### SSH Agent

**SSH Agent** stores private keys in memory, so you don't need to enter passphrases repeatedly.

**Start SSH Agent:**
```bash
# Start agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Add key with specific lifetime (1 hour)
ssh-add -t 3600 ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# Remove specific key
ssh-add -d ~/.ssh/id_ed25519

# Remove all keys
ssh-add -D
```

**Auto-start SSH Agent (add to ~/.bashrc or ~/.zshrc):**
```bash
# Start SSH agent if not running
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519 2>/dev/null
fi
```

**Using keychain (alternative):**
```bash
# Install keychain
sudo apt-get install keychain

# Add to ~/.bashrc
eval $(keychain --eval --quiet id_ed25519)
```

---

### SSH Key Management

**Multiple Keys:**
```bash
# Generate different keys for different purposes
ssh-keygen -t ed25519 -f ~/.ssh/id_work -C "work@company.com"
ssh-keygen -t ed25519 -f ~/.ssh/id_personal -C "personal@email.com"
ssh-keygen -t ed25519 -f ~/.ssh/id_github -C "github@email.com"

# Use specific key
ssh -i ~/.ssh/id_work user@work-server
ssh -i ~/.ssh/id_personal user@personal-server

# Configure in ~/.ssh/config
Host work-server
    IdentityFile ~/.ssh/id_work
    
Host personal-server
    IdentityFile ~/.ssh/id_personal
```

**Key Rotation:**
```bash
# 1. Generate new key
ssh-keygen -t ed25519 -f ~/.ssh/id_new

# 2. Copy new key to servers
ssh-copy-id -i ~/.ssh/id_new.pub user@server

# 3. Test new key
ssh -i ~/.ssh/id_new user@server

# 4. Remove old key from servers
ssh user@server
sed -i '/old-key-comment/d' ~/.ssh/authorized_keys

# 5. Delete old key locally
rm ~/.ssh/id_old ~/.ssh/id_old.pub
```

**Revoking Access:**
```bash
# On remote server, edit authorized_keys
ssh user@server
nano ~/.ssh/authorized_keys

# Remove the specific public key line
# Or comment it out with #

# Verify
cat ~/.ssh/authorized_keys
```

---

### Security Best Practices

1. **Use Strong Keys**
   - RSA: minimum 4096 bits
   - Ed25519: recommended for new keys
   - Avoid DSA and weak RSA keys

2. **Protect Private Keys**
   - Never share private keys
   - Use passphrases for private keys
   - Store keys securely (encrypted disk)
   - Use SSH agent for convenience

3. **Regular Key Rotation**
   - Rotate keys periodically (annually)
   - Remove unused keys
   - Audit authorized_keys regularly

4. **Limit Access**
   - Use different keys for different purposes
   - Implement principle of least privilege
   - Use AllowUsers/AllowGroups in sshd_config

5. **Monitor and Audit**
   - Log SSH access attempts
   - Monitor authorized_keys changes
   - Review SSH logs regularly
   ```bash
   # View SSH logs
   sudo tail -f /var/log/auth.log
   sudo journalctl -u sshd -f
   ```

6. **Additional Security**
   - Disable password authentication
   - Use non-standard SSH port
   - Implement fail2ban
   - Use two-factor authentication (2FA)
   - Restrict SSH to specific IPs

---

### Troubleshooting

**Common Issues:**

1. **Permission Denied (publickey)**
   ```bash
   # Check permissions
   ls -la ~/.ssh/
   
   # Fix permissions
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_*
   chmod 600 ~/.ssh/authorized_keys
   
   # Verify key is loaded
   ssh-add -l
   
   # Add key if needed
   ssh-add ~/.ssh/id_ed25519
   ```

2. **Connection Refused**
   ```bash
   # Check SSH service
   sudo systemctl status sshd
   
   # Check port
   sudo netstat -tlnp | grep ssh
   
   # Check firewall
   sudo ufw status
   sudo firewall-cmd --list-all
   ```

3. **Host Key Verification Failed**
   ```bash
   # Remove old host key
   ssh-keygen -R hostname
   
   # Or edit known_hosts
   nano ~/.ssh/known_hosts
   ```

4. **Too Many Authentication Failures**
   ```bash
   # Specify key explicitly
   ssh -i ~/.ssh/specific_key user@server
   
   # Or limit keys in ssh-agent
   ssh-add -D  # Remove all
   ssh-add ~/.ssh/specific_key  # Add only needed key
   ```

**Debug Mode:**
```bash
# Verbose output (levels: -v, -vv, -vvv)
ssh -v user@server
ssh -vv user@server
ssh -vvv user@server

# Test configuration
ssh -T git@github.com

# Check which key is being used
ssh -v user@server 2>&1 | grep "identity file"
```

---

## 🔑 Key Takeaways

1. SSH key authentication is more secure than passwords
2. Always protect private keys with passphrases
3. Use Ed25519 or RSA 4096-bit keys
4. Proper file permissions are critical
5. Use SSH config for convenience
6. Regularly rotate and audit keys
7. Never share private keys
8. Use SSH agent for convenience with passphrases

---

## 📖 Additional Resources

- Man pages: `man ssh`, `man ssh-keygen`, `man ssh-copy-id`, `man sshd_config`
- SSH documentation: `/usr/share/doc/openssh-client/`
- GitHub SSH guide: https://docs.github.com/en/authentication/connecting-to-github-with-ssh
- DigitalOcean SSH tutorial: https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys

---

## 🔗 Related Topics
- SSH tunneling and port forwarding
- SSH jump hosts (bastion servers)
- SSH certificates
- Two-factor authentication (2FA) with SSH
- SSH hardening and security

---

**Date:** 2024-01-16
**Status:** ✅ Completed
**Difficulty:** ⭐⭐ Intermediate