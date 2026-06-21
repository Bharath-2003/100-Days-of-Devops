# Day 7: Password-less SSH Key-Based Authentication - Resources

## 📚 Official Documentation

### Man Pages
```bash
man ssh              # SSH client
man ssh-keygen       # Key generation
man ssh-copy-id      # Copy keys to servers
man ssh-add          # Add keys to agent
man ssh-agent        # Authentication agent
man sshd_config      # SSH daemon configuration
man ssh_config       # SSH client configuration
```

### Online Documentation
- [OpenSSH Official](https://www.openssh.com/)
- [OpenSSH Manual Pages](https://man.openbsd.org/ssh)
- [SSH.com Documentation](https://www.ssh.com/academy/ssh)
- [Ubuntu SSH Documentation](https://help.ubuntu.com/community/SSH)
- [Red Hat SSH Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/using-secure-communications-between-two-systems-with-openssh_securing-networks)

---

## 🛠️ Useful Tools

### SSH Key Management Tools

1. **ssh-keygen** - Generate SSH keys
   ```bash
   ssh-keygen -t ed25519 -C "comment"
   ssh-keygen -t rsa -b 4096
   ssh-keygen -lf ~/.ssh/id_ed25519.pub  # Show fingerprint
   ```

2. **ssh-copy-id** - Copy keys to remote servers
   ```bash
   ssh-copy-id user@host
   ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host
   ```

3. **ssh-agent** - Key agent for passphrase management
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ssh-add -l  # List keys
   ```

4. **keychain** - SSH agent manager
   ```bash
   sudo apt-get install keychain
   eval $(keychain --eval id_ed25519)
   ```

5. **ssh-audit** - SSH server auditing
   ```bash
   # Install
   pip install ssh-audit
   
   # Audit server
   ssh-audit hostname
   ```

### GUI Tools

1. **PuTTY** (Windows)
   - SSH client with key management
   - PuTTYgen for key generation
   - Download: https://www.putty.org/

2. **MobaXterm** (Windows)
   - Advanced SSH client
   - Built-in key management
   - Download: https://mobaxterm.mobatek.net/

3. **Termius** (Cross-platform)
   - Modern SSH client
   - Cloud sync for keys
   - Download: https://termius.com/

4. **Royal TSX** (macOS)
   - Connection manager
   - Key management
   - Download: https://www.royalapps.com/ts/mac

---

## 📖 Tutorials and Guides

### Beginner Tutorials

1. **DigitalOcean - SSH Essentials**
   - https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys
   - Comprehensive beginner guide
   - Step-by-step instructions

2. **GitHub SSH Guide**
   - https://docs.github.com/en/authentication/connecting-to-github-with-ssh
   - Setting up SSH for GitHub
   - Key management

3. **Linode SSH Guide**
   - https://www.linode.com/docs/guides/use-public-key-authentication-with-ssh/
   - Public key authentication
   - Security best practices

4. **AWS SSH Key Pairs**
   - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
   - Using SSH with EC2
   - Key pair management

### Advanced Guides

1. **SSH Hardening Guide**
   - https://www.ssh.com/academy/ssh/hardening
   - Security best practices
   - Advanced configuration

2. **SSH Tunneling and Port Forwarding**
   - https://www.ssh.com/academy/ssh/tunneling
   - Local and remote forwarding
   - Dynamic port forwarding

3. **SSH Certificates**
   - https://www.ssh.com/academy/ssh/certificate
   - Certificate-based authentication
   - CA setup

4. **SSH Jump Hosts**
   - https://www.redhat.com/sysadmin/ssh-proxy-bastion-proxyjump
   - Bastion host configuration
   - ProxyJump usage

---

## 💡 Practical Examples

### Common SSH Configurations

#### Basic SSH Config
```bash
# ~/.ssh/config

# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ForwardAgent no

# Production servers
Host prod-*
    User admin
    Port 22
    IdentityFile ~/.ssh/id_production
    StrictHostKeyChecking yes

# Development servers
Host dev-*
    User developer
    Port 2222
    IdentityFile ~/.ssh/id_development
    StrictHostKeyChecking no

# Specific server
Host myserver
    HostName 192.168.1.100
    User john
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

#### Jump Host Configuration
```bash
# ~/.ssh/config

# Bastion host
Host bastion
    HostName bastion.example.com
    User admin
    IdentityFile ~/.ssh/id_bastion

# Internal servers via bastion
Host internal-*
    ProxyJump bastion
    User admin
    IdentityFile ~/.ssh/id_internal

# Specific internal server
Host internal-db
    HostName 10.0.1.50
    ProxyJump bastion
```

#### Multiple GitHub Accounts
```bash
# ~/.ssh/config

# Personal GitHub
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github_personal
    IdentitiesOnly yes

# Work GitHub
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github_work
    IdentitiesOnly yes
```

### Security Scripts

#### SSH Key Audit Script
```bash
#!/bin/bash
# audit-ssh-keys.sh

echo "SSH Key Audit Report"
echo "===================="
echo ""

# Check for weak keys
echo "Checking for weak RSA keys..."
find ~/.ssh -name "id_rsa*" -type f ! -name "*.pub" -exec ssh-keygen -lf {} \; | while read line; do
    bits=$(echo $line | awk '{print $1}')
    if [ "$bits" -lt 2048 ]; then
        echo "WARNING: Weak key found - $line"
    fi
done

# List all keys
echo ""
echo "All SSH keys:"
ls -lh ~/.ssh/id_*

# Check permissions
echo ""
echo "Checking permissions..."
find ~/.ssh -type f -exec ls -l {} \; | awk '{if ($1 !~ /^-rw-------/ && $1 !~ /^-rw-r--r--/) print "WARNING: Incorrect permissions - " $0}'

# Check authorized_keys on servers
echo ""
echo "Checking authorized_keys on servers..."
# Add your servers here
servers=("server1" "server2")
for server in "${servers[@]}"; do
    echo "Checking $server..."
    ssh "$server" "wc -l ~/.ssh/authorized_keys"
done
```

---

## 🎓 Video Tutorials

### YouTube Channels

1. **LearnLinuxTV - SSH Tutorial Series**
   - SSH basics and advanced topics
   - Key management
   - Security best practices

2. **NetworkChuck - SSH Keys Explained**
   - Beginner-friendly explanations
   - Practical demonstrations

3. **The Linux Foundation - SSH Security**
   - Professional training content
   - Enterprise best practices

4. **TechWorld with Nana - SSH for DevOps**
   - DevOps-focused SSH usage
   - Automation examples

---

## 📝 Cheat Sheets

### Quick Reference

```bash
# Key Generation
ssh-keygen -t ed25519 -C "comment"          # Generate Ed25519 key
ssh-keygen -t rsa -b 4096 -C "comment"      # Generate RSA key
ssh-keygen -t ecdsa -b 521 -C "comment"     # Generate ECDSA key

# Key Management
ssh-copy-id user@host                        # Copy key to server
ssh-copy-id -i ~/.ssh/key.pub user@host     # Copy specific key
ssh-keygen -lf ~/.ssh/key.pub               # Show key fingerprint
ssh-keygen -R hostname                       # Remove host from known_hosts

# SSH Agent
eval "$(ssh-agent -s)"                       # Start agent
ssh-add ~/.ssh/id_ed25519                    # Add key
ssh-add -l                                   # List keys
ssh-add -D                                   # Remove all keys
ssh-add -t 3600 ~/.ssh/key                  # Add with timeout

# Connection
ssh user@host                                # Basic connection
ssh -i ~/.ssh/key user@host                 # Use specific key
ssh -p 2222 user@host                       # Custom port
ssh -v user@host                            # Verbose mode
ssh -J bastion user@internal                # Jump host

# File Transfer
scp file user@host:/path                    # Copy file to server
scp user@host:/path/file .                  # Copy file from server
scp -i ~/.ssh/key file user@host:/path      # Use specific key

# Tunneling
ssh -L 8080:localhost:80 user@host          # Local port forward
ssh -R 8080:localhost:80 user@host          # Remote port forward
ssh -D 1080 user@host                       # Dynamic port forward (SOCKS)

# Configuration
~/.ssh/config                                # Client config
/etc/ssh/sshd_config                        # Server config
~/.ssh/authorized_keys                       # Authorized public keys
~/.ssh/known_hosts                          # Known host keys
```

### File Permissions Reference
```
Directory/File              Permission  Octal
~/.ssh/                     drwx------  700
~/.ssh/id_*                 -rw-------  600
~/.ssh/id_*.pub             -rw-r--r--  644
~/.ssh/authorized_keys      -rw-------  600
~/.ssh/config               -rw-------  600
~/.ssh/known_hosts          -rw-r--r--  644
```

---

## 🔧 Sample Scripts

### Automated Key Deployment
```bash
#!/bin/bash
# deploy-keys.sh - Deploy SSH keys to multiple servers

SERVERS_FILE="servers.txt"
KEY_FILE="$HOME/.ssh/id_ed25519.pub"
USER="admin"

if [ ! -f "$SERVERS_FILE" ]; then
    echo "Error: $SERVERS_FILE not found"
    exit 1
fi

if [ ! -f "$KEY_FILE" ]; then
    echo "Error: $KEY_FILE not found"
    exit 1
fi

while IFS= read -r server; do
    echo "Deploying key to $server..."
    ssh-copy-id -i "$KEY_FILE" "${USER}@${server}"
    
    if [ $? -eq 0 ]; then
        echo "✓ Success: $server"
    else
        echo "✗ Failed: $server"
    fi
done < "$SERVERS_FILE"

echo "Deployment complete!"
```

### SSH Connection Tester
```bash
#!/bin/bash
# test-ssh-connections.sh

SERVERS_FILE="servers.txt"
TIMEOUT=5

echo "Testing SSH connections..."
echo "=========================="

while IFS= read -r server; do
    echo -n "Testing $server... "
    
    if timeout $TIMEOUT ssh -o BatchMode=yes -o ConnectTimeout=$TIMEOUT "$server" "echo 'OK'" &>/dev/null; then
        echo "✓ Connected"
    else
        echo "✗ Failed"
    fi
done < "$SERVERS_FILE"
```

---

## 🐛 Debugging Resources

### Common Error Messages

1. **"Permission denied (publickey)"**
   - Check file permissions
   - Verify key is in authorized_keys
   - Check SSH agent

2. **"Host key verification failed"**
   - Remove old host key: `ssh-keygen -R hostname`
   - Accept new host key

3. **"Too many authentication failures"**
   - Use `IdentitiesOnly yes` in config
   - Specify key with `-i` option

4. **"Connection refused"**
   - Check SSH service is running
   - Verify firewall rules
   - Check correct port

### Debug Commands
```bash
# Verbose SSH connection
ssh -v user@host      # Level 1
ssh -vv user@host     # Level 2
ssh -vvv user@host    # Level 3 (most verbose)

# Test SSH configuration
ssh -T git@github.com

# Check which key is being used
ssh -v user@host 2>&1 | grep "identity file"

# Test server configuration
sudo sshd -t

# View SSH logs
sudo tail -f /var/log/auth.log          # Ubuntu/Debian
sudo tail -f /var/log/secure            # CentOS/RHEL
sudo journalctl -u sshd -f              # Systemd
```

---

## 🌐 Community Resources

### Forums and Q&A

1. **Stack Overflow - SSH Tag**
   - https://stackoverflow.com/questions/tagged/ssh
   - Active community
   - Thousands of answered questions

2. **Unix & Linux Stack Exchange**
   - https://unix.stackexchange.com/questions/tagged/ssh
   - Expert answers
   - Best practices

3. **Server Fault**
   - https://serverfault.com/questions/tagged/ssh
   - System administration focus
   - Professional advice

4. **Reddit Communities**
   - r/sysadmin - https://www.reddit.com/r/sysadmin/
   - r/linuxadmin - https://www.reddit.com/r/linuxadmin/
   - r/devops - https://www.reddit.com/r/devops/

### Blogs and Articles

1. **SSH.com Blog**
   - https://www.ssh.com/academy/ssh/blog
   - Latest SSH news and tips

2. **DigitalOcean Community**
   - https://www.digitalocean.com/community/tags/ssh
   - Tutorials and guides

3. **Red Hat Blog**
   - https://www.redhat.com/sysadmin/
   - Enterprise SSH practices

---

## 📦 Related Tools and Technologies

### Security Tools

1. **fail2ban** - Intrusion prevention
   ```bash
   sudo apt-get install fail2ban
   sudo systemctl enable fail2ban
   ```

2. **denyhosts** - SSH brute-force protection
   ```bash
   sudo apt-get install denyhosts
   ```

3. **sshguard** - Log monitor and blocker
   ```bash
   sudo apt-get install sshguard
   ```

4. **Google Authenticator** - 2FA for SSH
   ```bash
   sudo apt-get install libpam-google-authenticator
   ```

### Automation Tools

1. **Ansible** - Configuration management
   - Uses SSH for connections
   - Key-based authentication

2. **Fabric** - Python SSH library
   - Automate SSH tasks
   - Deployment automation

3. **Paramiko** - Python SSH implementation
   - Programmatic SSH access
   - Custom automation

---

## 🎯 Practice Exercises

### Beginner Level
1. Generate SSH key pair
2. Copy key to remote server
3. Test password-less login
4. Create SSH config file
5. Use SSH agent

### Intermediate Level
1. Set up multiple keys for different servers
2. Configure jump host access
3. Disable password authentication
4. Implement key rotation
5. Set up SSH for GitHub

### Advanced Level
1. Implement SSH certificates
2. Set up SSH CA
3. Configure two-factor authentication
4. Build automated key management system
5. Implement SSH bastion architecture

---

## 📚 Books and Publications

### Recommended Reading

1. **"SSH, The Secure Shell: The Definitive Guide"**
   - By Daniel J. Barrett, Richard E. Silverman
   - Comprehensive SSH reference

2. **"Linux Security Cookbook"**
   - By Daniel J. Barrett, Richard E. Silverman, Robert G. Byrnes
   - Includes SSH security recipes

3. **"Practical SSH"**
   - By Michael W. Lucas
   - Practical SSH usage and administration

---

## 🔗 Quick Links

- [OpenSSH](https://www.openssh.com/)
- [SSH.com Academy](https://www.ssh.com/academy/)
- [GitHub SSH Guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [DigitalOcean SSH Tutorials](https://www.digitalocean.com/community/tags/ssh)
- [SSH Audit Tool](https://github.com/jtesta/ssh-audit)

---

## 💬 Getting Help

### Where to Ask Questions

1. **Stack Overflow** - Technical questions
2. **Unix & Linux Stack Exchange** - System administration
3. **Server Fault** - Professional sysadmin
4. **Reddit r/linuxadmin** - Community support

### IRC Channels
- #ssh on Libera.Chat
- #openssh on Libera.Chat
- #linux on Libera.Chat

---

## 🔐 Security Resources

### Security Advisories
- [OpenSSH Security](https://www.openssh.com/security.html)
- [CVE Details - OpenSSH](https://www.cvedetails.com/product/585/Openbsd-Openssh.html)

### Security Best Practices
- [CIS OpenSSH Benchmark](https://www.cisecurity.org/)
- [NIST SSH Guidelines](https://csrc.nist.gov/)
- [Mozilla SSH Guidelines](https://infosec.mozilla.org/guidelines/openssh)

---

**Last Updated:** 2024-01-16
**Maintained By:** DevOps Learning Community