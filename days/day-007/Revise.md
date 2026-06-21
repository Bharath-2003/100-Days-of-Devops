# Day 7: Password-less SSH Key-Based Authentication - Quick Revision Guide

## 🎯 Quick Summary

SSH key-based authentication provides secure, password-less access to remote servers using cryptographic key pairs. This method is more secure than password authentication and is essential for automation and DevOps workflows.

**Key Concepts:**
- Public/Private key pairs for authentication
- Ed25519 recommended for new keys (RSA 4096-bit also acceptable)
- Proper file permissions are critical (600 for private keys)
- SSH agent manages passphrases for convenience
- SSH config file simplifies connection management

---

## ⚡ Essential Commands

### Key Generation
```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate RSA key (4096-bit)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate key with custom filename
ssh-keygen -t ed25519 -f ~/.ssh/id_custom -C "comment"
```

### Key Deployment
```bash
# Copy key to server (easiest method)
ssh-copy-id user@hostname

# Copy specific key
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@hostname

# Manual copy (if ssh-copy-id unavailable)
cat ~/.ssh/id_ed25519.pub | ssh user@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### SSH Agent
```bash
# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# Remove all keys from agent
ssh-add -D

# Add key with timeout (1 hour)
ssh-add -t 3600 ~/.ssh/id_ed25519
```

### Connection Commands
```bash
# Basic connection
ssh user@hostname

# Use specific key
ssh -i ~/.ssh/id_custom user@hostname

# Custom port
ssh -p 2222 user@hostname

# Verbose mode (debugging)
ssh -v user@hostname
ssh -vv user@hostname  # More verbose
ssh -vvv user@hostname # Most verbose

# Jump host
ssh -J bastion_user@bastion internal_user@internal_host
```

### Key Management
```bash
# View key fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# Change key passphrase
ssh-keygen -p -f ~/.ssh/id_ed25519

# Remove host from known_hosts
ssh-keygen -R hostname

# Test GitHub connection
ssh -T git@github.com
```

---

## 📋 File Permissions Checklist

```
✓ ~/.ssh/                    700 (drwx------)
✓ ~/.ssh/id_*                600 (-rw-------)
✓ ~/.ssh/id_*.pub            644 (-rw-r--r--)
✓ ~/.ssh/authorized_keys     600 (-rw-------)
✓ ~/.ssh/config              600 (-rw-------)
✓ ~/.ssh/known_hosts         644 (-rw-r--r--)
```

**Fix permissions:**
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/id_*.pub
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config
```

---

## 🔧 SSH Config File Template

```bash
# ~/.ssh/config

# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes

# Specific server
Host myserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# Multiple servers with pattern
Host prod-*
    User admin
    Port 22
    IdentityFile ~/.ssh/id_production
    StrictHostKeyChecking yes

# Jump host configuration
Host internal-server
    HostName 10.0.1.50
    User admin
    ProxyJump bastion.example.com
    IdentityFile ~/.ssh/id_internal

# GitHub personal
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github_personal
    IdentitiesOnly yes
```

---

## 🚀 Quick Setup Steps

### 1. Generate Key
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
# Press Enter for default location
# Enter passphrase (recommended)
```

### 2. Copy to Server
```bash
ssh-copy-id user@hostname
# Enter password one last time
```

### 3. Test Connection
```bash
ssh user@hostname
# Should connect without password
```

### 4. Disable Password Auth (Optional)
```bash
# On server
sudo nano /etc/ssh/sshd_config

# Set these values:
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no

# Restart SSH
sudo systemctl restart sshd
```

---

## 🔍 Troubleshooting Quick Reference

### Permission Denied (publickey)
```bash
# Check file permissions
ls -la ~/.ssh/

# Fix permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/id_*.pub

# Verify key is on server
ssh user@hostname "cat ~/.ssh/authorized_keys"

# Check SSH agent
ssh-add -l

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Debug connection
ssh -vvv user@hostname
```

### Connection Refused
```bash
# Check SSH service
sudo systemctl status sshd

# Check firewall
sudo ufw status
sudo firewall-cmd --list-all

# Verify port
sudo netstat -tlnp | grep ssh
```

### Host Key Verification Failed
```bash
# Remove old host key
ssh-keygen -R hostname

# Or edit known_hosts
nano ~/.ssh/known_hosts
```

### Too Many Authentication Failures
```bash
# Use specific key
ssh -i ~/.ssh/id_specific user@hostname

# Or add to config
Host hostname
    IdentitiesOnly yes
    IdentityFile ~/.ssh/id_specific
```

---

## 📊 Key Types Comparison

| Type | Key Size | Speed | Security | Recommendation |
|------|----------|-------|----------|----------------|
| Ed25519 | 256-bit | ⚡⚡⚡ | 🔒🔒🔒 | ✅ Best choice |
| RSA | 4096-bit | ⚡⚡ | 🔒🔒🔒 | ✅ Good (legacy) |
| ECDSA | 521-bit | ⚡⚡⚡ | 🔒🔒 | ⚠️ Use with caution |
| DSA | 1024-bit | ⚡ | 🔒 | ❌ Deprecated |

**Recommendation:** Use Ed25519 for new keys, RSA 4096-bit for compatibility.

---

## 🎓 Common Use Cases

### 1. Basic Server Access
```bash
# Generate key
ssh-keygen -t ed25519

# Copy to server
ssh-copy-id user@server

# Connect
ssh user@server
```

### 2. GitHub/GitLab
```bash
# Generate key
ssh-keygen -t ed25519 -C "github@email.com" -f ~/.ssh/id_github

# Copy public key
cat ~/.ssh/id_github.pub

# Add to GitHub: Settings → SSH Keys → New SSH Key

# Test
ssh -T git@github.com
```

### 3. Multiple Servers
```bash
# Create config
cat > ~/.ssh/config << 'EOF'
Host server1
    HostName 192.168.1.10
    User admin
    IdentityFile ~/.ssh/id_server1

Host server2
    HostName 192.168.1.20
    User admin
    IdentityFile ~/.ssh/id_server2
EOF

# Connect
ssh server1
ssh server2
```

### 4. Jump Host (Bastion)
```bash
# Config method
Host bastion
    HostName bastion.example.com
    User admin

Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump bastion

# Connect
ssh internal

# Or command line
ssh -J admin@bastion admin@10.0.1.50
```

---

## 🔐 Security Best Practices

### ✅ Do's
- ✓ Use Ed25519 or RSA 4096-bit keys
- ✓ Always use passphrases on private keys
- ✓ Set correct file permissions (700/600/644)
- ✓ Use SSH agent for convenience
- ✓ Rotate keys periodically (annually)
- ✓ Use different keys for different purposes
- ✓ Disable password authentication
- ✓ Use SSH config for organization
- ✓ Keep private keys secure (never share)
- ✓ Use jump hosts for internal servers

### ❌ Don'ts
- ✗ Don't use DSA keys (deprecated)
- ✗ Don't use RSA keys < 2048 bits
- ✗ Don't skip passphrases
- ✗ Don't share private keys
- ✗ Don't commit keys to version control
- ✗ Don't use same key everywhere
- ✗ Don't ignore file permissions
- ✗ Don't disable StrictHostKeyChecking globally
- ✗ Don't store keys in cloud sync folders
- ✗ Don't forget to backup keys securely

---

## 💡 Pro Tips

1. **Use SSH Config**
   - Simplifies connections
   - Manages multiple keys
   - Sets default options

2. **SSH Agent**
   - Enter passphrase once per session
   - Automatic key management
   - Works with git operations

3. **Key Comments**
   - Use `-C` flag for meaningful comments
   - Helps identify keys later
   - Example: `ssh-keygen -t ed25519 -C "work-laptop-2024"`

4. **Backup Keys**
   - Keep encrypted backups
   - Store in secure location
   - Document key purposes

5. **Key Rotation**
   - Generate new keys annually
   - Remove old keys from servers
   - Update documentation

---

## 📝 Pre-Exam Checklist

- [ ] Can generate SSH key pairs
- [ ] Know difference between public and private keys
- [ ] Can copy keys to remote servers
- [ ] Understand file permissions (700, 600, 644)
- [ ] Can use SSH agent
- [ ] Know how to create SSH config
- [ ] Can troubleshoot connection issues
- [ ] Understand key types (Ed25519, RSA, etc.)
- [ ] Can disable password authentication
- [ ] Know how to use jump hosts
- [ ] Can manage multiple keys
- [ ] Understand security best practices

---

## 🎯 Interview Questions

### Basic Level

**Q1: What is SSH key-based authentication?**
A: A method of logging into SSH servers using cryptographic key pairs (public/private) instead of passwords. More secure and enables automation.

**Q2: What are the two parts of an SSH key pair?**
A: Private key (kept secret on client) and public key (shared with servers).

**Q3: What is the recommended key type for new SSH keys?**
A: Ed25519 is recommended for its security and performance. RSA 4096-bit is also acceptable.

**Q4: What command generates an SSH key?**
A: `ssh-keygen -t ed25519 -C "comment"`

**Q5: How do you copy an SSH key to a server?**
A: `ssh-copy-id user@hostname`

### Intermediate Level

**Q6: What are the correct permissions for SSH files?**
A:
- ~/.ssh/ directory: 700
- Private keys: 600
- Public keys: 644
- authorized_keys: 600

**Q7: What is the purpose of SSH agent?**
A: SSH agent stores decrypted private keys in memory, allowing you to use passphrase-protected keys without entering the passphrase repeatedly.

**Q8: How do you use a specific SSH key for a connection?**
A: `ssh -i ~/.ssh/specific_key user@hostname` or configure in ~/.ssh/config

**Q9: What is the authorized_keys file?**
A: A file on the server (~/.ssh/authorized_keys) containing public keys that are allowed to authenticate.

**Q10: How do you debug SSH connection issues?**
A: Use verbose mode: `ssh -vvv user@hostname`

### Advanced Level

**Q11: How do you set up SSH jump host (bastion)?**
A: Use ProxyJump in config or `-J` flag:
```bash
ssh -J bastion_user@bastion internal_user@internal
```

**Q12: How do you manage multiple SSH keys for different services?**
A: Use ~/.ssh/config with different IdentityFile entries for each host.

**Q13: What security measures should be taken on SSH server?**
A:
- Disable password authentication
- Disable root login
- Use non-standard port
- Implement fail2ban
- Keep SSH updated
- Use strong key types

**Q14: How do you rotate SSH keys?**
A:
1. Generate new key pair
2. Copy new public key to servers
3. Test new key works
4. Remove old public key from servers
5. Delete old private key

**Q15: What is the difference between IdentityFile and IdentitiesOnly?**
A:
- IdentityFile: Specifies which key file to use
- IdentitiesOnly: When set to yes, only uses keys specified in config (prevents trying all keys in agent)

---

## 🔄 Common Workflows

### Daily Development
```bash
# Start SSH agent (once per session)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Connect to servers
ssh dev-server
ssh prod-server

# Git operations (automatic with agent)
git push origin main
```

### Server Provisioning
```bash
# Generate key
ssh-keygen -t ed25519 -f ~/.ssh/id_newserver

# Copy to server
ssh-copy-id -i ~/.ssh/id_newserver.pub user@newserver

# Add to config
cat >> ~/.ssh/config << EOF
Host newserver
    HostName 192.168.1.100
    User admin
    IdentityFile ~/.ssh/id_newserver
EOF

# Test
ssh newserver
```

### Key Rotation
```bash
# Generate new key
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_new

# Copy to all servers
for server in server1 server2 server3; do
    ssh-copy-id -i ~/.ssh/id_ed25519_new.pub $server
done

# Test new key
ssh -i ~/.ssh/id_ed25519_new server1

# Remove old key from servers
for server in server1 server2 server3; do
    ssh $server "sed -i '/old_key_fingerprint/d' ~/.ssh/authorized_keys"
done

# Rename new key
mv ~/.ssh/id_ed25519_new ~/.ssh/id_ed25519
mv ~/.ssh/id_ed25519_new.pub ~/.ssh/id_ed25519.pub
```

---

## 📚 Additional Resources

- **Man Pages:** `man ssh`, `man ssh-keygen`, `man ssh-copy-id`, `man ssh_config`
- **Online:** https://www.ssh.com/academy/ssh
- **GitHub Guide:** https://docs.github.com/en/authentication/connecting-to-github-with-ssh
- **DigitalOcean:** https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys

---

## ✅ Quick Verification

After setup, verify everything works:

```bash
# 1. Check key exists
ls -la ~/.ssh/id_*

# 2. Check permissions
ls -la ~/.ssh/

# 3. Test connection
ssh user@hostname

# 4. Verify no password prompt
# Should connect without asking for password

# 5. Check server authorized_keys
ssh user@hostname "cat ~/.ssh/authorized_keys"
```

---

**Remember:** 
- Private key = Secret (never share)
- Public key = Shareable (copy to servers)
- Passphrase = Extra security layer
- SSH agent = Convenience tool
- Permissions = Critical for security

**Last Updated:** 2024-01-16