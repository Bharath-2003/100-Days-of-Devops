# Day 7: Password-less SSH Key-Based Authentication - Solutions

## 📝 Task Solutions

### Solution 1: Basic SSH Key Setup

**Objective:** Generate SSH keys and set up password-less authentication to a remote server.

**Step-by-Step Solution:**

```bash
# Step 1: Check if SSH keys already exist
ls -la ~/.ssh/

# Step 2: Generate new SSH key pair (Ed25519 - recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# When prompted:
# - File location: Press Enter for default (~/.ssh/id_ed25519)
# - Passphrase: Enter a strong passphrase (or press Enter for no passphrase)
# - Confirm passphrase: Re-enter the same passphrase

# Step 3: Verify keys were created
ls -la ~/.ssh/
# You should see:
# id_ed25519 (private key)
# id_ed25519.pub (public key)

# Step 4: View your public key
cat ~/.ssh/id_ed25519.pub

# Step 5: Copy public key to remote server (easiest method)
ssh-copy-id user@remote-server-ip

# Example:
ssh-copy-id john@192.168.1.100

# Enter your password when prompted (last time!)

# Step 6: Test password-less login
ssh user@remote-server-ip

# You should now be logged in without entering a password!

# Step 7: Verify the setup on remote server
ssh user@remote-server-ip
cat ~/.ssh/authorized_keys
# Your public key should be listed here
exit
```

**Expected Output:**
```
$ ssh-keygen -t ed25519 -C "john@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/john/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/john/.ssh/id_ed25519
Your public key has been saved in /home/john/.ssh/id_ed25519.pub

$ ssh-copy-id john@192.168.1.100
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s)
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed
john@192.168.1.100's password: [enter password]

Number of key(s) added: 1

Now try logging into the machine with: "ssh 'john@192.168.1.100'"

$ ssh john@192.168.1.100
Welcome to Ubuntu 22.04 LTS
Last login: Sat Jan 16 10:30:00 2024
john@remote-server:~$
```

---

### Solution 2: Manual SSH Key Setup (Without ssh-copy-id)

**Objective:** Set up SSH keys manually when ssh-copy-id is not available.

**Step-by-Step Solution:**

```bash
# Step 1: Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "admin@company.com"

# Step 2: Display your public key
cat ~/.ssh/id_rsa.pub

# Step 3: Copy the entire output (starts with ssh-rsa...)

# Step 4: SSH to remote server with password
ssh user@remote-server

# Step 5: On remote server, create .ssh directory if it doesn't exist
mkdir -p ~/.ssh

# Step 6: Set proper permissions on .ssh directory
chmod 700 ~/.ssh

# Step 7: Create or edit authorized_keys file
nano ~/.ssh/authorized_keys

# Step 8: Paste your public key on a new line
# Save and exit (Ctrl+X, then Y, then Enter)

# Step 9: Set proper permissions on authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Step 10: Exit from remote server
exit

# Step 11: Test password-less login
ssh user@remote-server
```

**Alternative One-liner Method:**
```bash
# Copy public key and append to authorized_keys in one command
cat ~/.ssh/id_rsa.pub | ssh user@remote-server "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Enter password when prompted

# Test connection
ssh user@remote-server
```

---

### Solution 3: Multiple SSH Keys for Different Servers

**Objective:** Manage multiple SSH keys for different purposes (work, personal, GitHub, etc.).

**Step-by-Step Solution:**

```bash
# Step 1: Generate different keys for different purposes
# Work key
ssh-keygen -t ed25519 -f ~/.ssh/id_work -C "work@company.com"

# Personal key
ssh-keygen -t ed25519 -f ~/.ssh/id_personal -C "personal@email.com"

# GitHub key
ssh-keygen -t ed25519 -f ~/.ssh/id_github -C "github@email.com"

# Step 2: List all your keys
ls -la ~/.ssh/

# Step 3: Create SSH config file
nano ~/.ssh/config

# Step 4: Add configuration for each server
cat > ~/.ssh/config << 'EOF'
# Work servers
Host work-server work-prod work-dev
    HostName %h.company.com
    User admin
    IdentityFile ~/.ssh/id_work
    Port 22

# Personal server
Host personal
    HostName personal-server.example.com
    User john
    IdentityFile ~/.ssh/id_personal
    Port 2222

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github
    
# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
EOF

# Step 5: Set proper permissions on config file
chmod 600 ~/.ssh/config

# Step 6: Copy keys to respective servers
# Work server
ssh-copy-id -i ~/.ssh/id_work.pub admin@work-server.company.com

# Personal server
ssh-copy-id -i ~/.ssh/id_personal.pub -p 2222 john@personal-server.example.com

# GitHub (add key via web interface)
cat ~/.ssh/id_github.pub
# Copy and paste to GitHub Settings > SSH Keys

# Step 7: Test connections
ssh work-server
ssh personal
ssh -T git@github.com

# Step 8: Verify which key is being used
ssh -v work-server 2>&1 | grep "identity file"
```

**SSH Config Benefits:**
```bash
# Instead of typing:
ssh -i ~/.ssh/id_work -p 22 admin@work-server.company.com

# You can simply type:
ssh work-server
```

---

### Solution 4: SSH Key with Passphrase and SSH Agent

**Objective:** Use SSH keys with passphrases for security while maintaining convenience with SSH agent.

**Step-by-Step Solution:**

```bash
# Step 1: Generate key with passphrase
ssh-keygen -t ed25519 -C "secure@example.com"
# Enter a strong passphrase when prompted

# Step 2: Start SSH agent
eval "$(ssh-agent -s)"
# Output: Agent pid 12345

# Step 3: Add key to SSH agent
ssh-add ~/.ssh/id_ed25519
# Enter your passphrase

# Step 4: Verify key is loaded
ssh-add -l
# Output shows fingerprint of loaded key

# Step 5: Copy key to remote server
ssh-copy-id user@remote-server

# Step 6: Test connection (no passphrase needed this session)
ssh user@remote-server

# Step 7: Make SSH agent persistent (add to ~/.bashrc or ~/.zshrc)
cat >> ~/.bashrc << 'EOF'

# Start SSH agent if not running
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)" > /dev/null
    ssh-add ~/.ssh/id_ed25519 2>/dev/null
fi
EOF

# Step 8: Source the updated bashrc
source ~/.bashrc

# Step 9: Test in new terminal
# Open new terminal and try SSH
ssh user@remote-server
# Should work without entering passphrase
```

**Using keychain (Alternative):**
```bash
# Step 1: Install keychain
sudo apt-get update
sudo apt-get install keychain

# Step 2: Add to ~/.bashrc
cat >> ~/.bashrc << 'EOF'

# Keychain for SSH agent management
eval $(keychain --eval --quiet id_ed25519)
EOF

# Step 3: Source bashrc
source ~/.bashrc

# Step 4: Enter passphrase once
# Keychain will remember it for future sessions
```

---

### Solution 5: Disable Password Authentication (Security Hardening)

**Objective:** Disable password-based SSH login after setting up key authentication.

**Step-by-Step Solution:**

```bash
# IMPORTANT: Only do this AFTER confirming key-based auth works!

# Step 1: Test key-based authentication first
ssh user@remote-server
# Ensure you can login without password

# Step 2: Keep current session open as backup
# Open a NEW terminal for the following steps

# Step 3: SSH to server
ssh user@remote-server

# Step 4: Backup SSH configuration
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Step 5: Edit SSH daemon configuration
sudo nano /etc/ssh/sshd_config

# Step 6: Find and modify these settings:
# Change or add these lines:
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
PermitRootLogin no

# Step 7: Save and exit (Ctrl+X, Y, Enter)

# Step 8: Test configuration for syntax errors
sudo sshd -t

# Step 9: If no errors, restart SSH service
sudo systemctl restart sshd
# or
sudo systemctl restart ssh

# Step 10: In your NEW terminal, test SSH connection
ssh user@remote-server
# Should still work with key

# Step 11: Try password authentication (should fail)
ssh -o PubkeyAuthentication=no user@remote-server
# Should be denied

# Step 12: If everything works, close backup session
```

**Automated Script:**
```bash
#!/bin/bash
# harden-ssh.sh - Disable password authentication

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

echo "SSH Hardening Script"
echo "===================="

# Check if running as root
if [ "$EUID" -ne 0 ]; then 
    echo -e "${RED}Please run as root or with sudo${NC}"
    exit 1
fi

# Backup configuration
echo "Backing up SSH configuration..."
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)

# Modify configuration
echo "Updating SSH configuration..."
sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^#*PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/^#*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config

# Test configuration
echo "Testing SSH configuration..."
if sshd -t; then
    echo -e "${GREEN}Configuration test passed${NC}"
    
    # Restart SSH
    echo "Restarting SSH service..."
    systemctl restart sshd || systemctl restart ssh
    
    echo -e "${GREEN}SSH hardening complete!${NC}"
    echo "Password authentication is now disabled"
    echo "Only key-based authentication is allowed"
else
    echo -e "${RED}Configuration test failed!${NC}"
    echo "Restoring backup..."
    cp /etc/ssh/sshd_config.backup.* /etc/ssh/sshd_config
    exit 1
fi
```

---

### Solution 6: SSH Key for GitHub/GitLab

**Objective:** Set up SSH keys for Git operations.

**Step-by-Step Solution:**

```bash
# Step 1: Generate SSH key for GitHub
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_github

# Step 2: Start SSH agent
eval "$(ssh-agent -s)"

# Step 3: Add key to SSH agent
ssh-add ~/.ssh/id_github

# Step 4: Copy public key to clipboard
# Linux
cat ~/.ssh/id_github.pub | xclip -selection clipboard
# or just display it
cat ~/.ssh/id_github.pub

# macOS
cat ~/.ssh/id_github.pub | pbcopy

# Step 5: Add key to GitHub
# Go to: GitHub.com > Settings > SSH and GPG keys > New SSH key
# Paste your public key and save

# Step 6: Test GitHub connection
ssh -T git@github.com
# Expected output: Hi username! You've successfully authenticated...

# Step 7: Configure Git to use SSH
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"

# Step 8: Clone repository using SSH
git clone git@github.com:username/repository.git

# Step 9: For existing repositories, change remote URL
cd existing-repo
git remote -v
git remote set-url origin git@github.com:username/repository.git
git remote -v

# Step 10: Test push/pull
git pull
git push
```

**Multiple GitHub Accounts:**
```bash
# Step 1: Generate keys for each account
ssh-keygen -t ed25519 -f ~/.ssh/id_github_personal -C "personal@email.com"
ssh-keygen -t ed25519 -f ~/.ssh/id_github_work -C "work@company.com"

# Step 2: Add to SSH config
cat >> ~/.ssh/config << 'EOF'

# Personal GitHub
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github_personal

# Work GitHub
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github_work
EOF

# Step 3: Add keys to respective GitHub accounts

# Step 4: Clone using custom host
git clone git@github-personal:username/personal-repo.git
git clone git@github-work:company/work-repo.git

# Step 5: For existing repos, update remote
git remote set-url origin git@github-personal:username/repo.git
```

---

## 🔍 Verification Steps

### Verify SSH Key Setup
```bash
# 1. Check if keys exist
ls -la ~/.ssh/

# 2. Verify permissions
ls -l ~/.ssh/id_*

# 3. Check public key content
cat ~/.ssh/id_ed25519.pub

# 4. Verify key fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# 5. Test connection with verbose output
ssh -v user@remote-server

# 6. Check which key is being used
ssh -v user@remote-server 2>&1 | grep "identity file"

# 7. Verify authorized_keys on remote server
ssh user@remote-server "cat ~/.ssh/authorized_keys"
```

### Verify SSH Agent
```bash
# Check if agent is running
echo $SSH_AUTH_SOCK

# List loaded keys
ssh-add -l

# Test agent
ssh-add -T ~/.ssh/id_ed25519.pub
```

### Verify Server Configuration
```bash
# On remote server
sudo sshd -t  # Test configuration
sudo grep -E "PubkeyAuthentication|PasswordAuthentication" /etc/ssh/sshd_config
```

---

## 🐛 Troubleshooting Common Issues

### Issue 1: Permission Denied (publickey)

**Problem:** Cannot connect even with correct key.

**Solutions:**
```bash
# 1. Check file permissions
ls -la ~/.ssh/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 600 ~/.ssh/authorized_keys

# 2. Verify key is in authorized_keys
ssh user@remote-server "cat ~/.ssh/authorized_keys"

# 3. Check SSH agent
ssh-add -l
ssh-add ~/.ssh/id_ed25519

# 4. Use verbose mode to debug
ssh -vvv user@remote-server

# 5. Check server logs
ssh user@remote-server "sudo tail -50 /var/log/auth.log"

# 6. Verify SELinux context (if applicable)
ssh user@remote-server "ls -Z ~/.ssh/"
ssh user@remote-server "restorecon -Rv ~/.ssh/"
```

### Issue 2: Too Many Authentication Failures

**Problem:** "Received disconnect: Too many authentication failures"

**Solutions:**
```bash
# 1. Specify key explicitly
ssh -i ~/.ssh/id_ed25519 user@remote-server

# 2. Clear SSH agent and add only needed key
ssh-add -D
ssh-add ~/.ssh/id_ed25519

# 3. Use IdentitiesOnly in config
cat >> ~/.ssh/config << 'EOF'
Host remote-server
    IdentitiesOnly yes
    IdentityFile ~/.ssh/id_ed25519
EOF

# 4. Increase MaxAuthTries on server
# Edit /etc/ssh/sshd_config
# MaxAuthTries 6
```

### Issue 3: Host Key Verification Failed

**Problem:** "Host key verification failed"

**Solutions:**
```bash
# 1. Remove old host key
ssh-keygen -R hostname
ssh-keygen -R ip-address

# 2. Or edit known_hosts manually
nano ~/.ssh/known_hosts
# Remove the line for the problematic host

# 3. Accept new host key
ssh user@remote-server
# Type 'yes' when prompted

# 4. Disable strict host key checking (not recommended for production)
ssh -o StrictHostKeyChecking=no user@remote-server
```

### Issue 4: Connection Timeout

**Problem:** SSH connection times out.

**Solutions:**
```bash
# 1. Check if server is reachable
ping remote-server

# 2. Check if SSH port is open
telnet remote-server 22
nc -zv remote-server 22

# 3. Check firewall rules
sudo ufw status
sudo iptables -L -n

# 4. Check SSH service on server
ssh user@remote-server "sudo systemctl status sshd"

# 5. Try different port
ssh -p 2222 user@remote-server
```

---

## ✅ Success Criteria

- [ ] SSH keys generated successfully
- [ ] Public key copied to remote server
- [ ] Can login without password
- [ ] Proper file permissions set
- [ ] SSH config file created (if using multiple keys)
- [ ] SSH agent configured (if using passphrases)
- [ ] Password authentication disabled (optional, for security)
- [ ] Connection tested and working
- [ ] Backup access method available

---

## 📚 Additional Examples

### Automated Deployment Key Setup
```bash
#!/bin/bash
# setup-deployment-key.sh

DEPLOY_USER="deploy"
SERVERS=("server1.example.com" "server2.example.com" "server3.example.com")
KEY_FILE="$HOME/.ssh/id_deployment"

# Generate deployment key
if [ ! -f "$KEY_FILE" ]; then
    ssh-keygen -t ed25519 -f "$KEY_FILE" -N "" -C "deployment@automation"
fi

# Copy to all servers
for server in "${SERVERS[@]}"; do
    echo "Setting up key on $server..."
    ssh-copy-id -i "${KEY_FILE}.pub" "${DEPLOY_USER}@${server}"
done

echo "Deployment key setup complete!"
```

### SSH Key Rotation Script
```bash
#!/bin/bash
# rotate-ssh-keys.sh

OLD_KEY="$HOME/.ssh/id_old"
NEW_KEY="$HOME/.ssh/id_new"
SERVERS_FILE="$HOME/.ssh/servers.txt"

# Generate new key
ssh-keygen -t ed25519 -f "$NEW_KEY" -C "rotated-$(date +%Y%m%d)"

# Copy new key to all servers
while read server; do
    echo "Updating $server..."
    ssh-copy-id -i "${NEW_KEY}.pub" "$server"
done < "$SERVERS_FILE"

# Test new key
echo "Testing new key..."
while read server; do
    if ssh -i "$NEW_KEY" -o BatchMode=yes "$server" "echo 'OK'"; then
        echo "$server: ✓"
    else
        echo "$server: ✗ FAILED"
    fi
done < "$SERVERS_FILE"

echo "Key rotation complete!"
echo "Old key: $OLD_KEY"
echo "New key: $NEW_KEY"
```

---

**Completion Date:** 2024-01-16
**Status:** ✅ All solutions tested and verified