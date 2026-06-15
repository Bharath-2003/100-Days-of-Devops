# Day 001 - Resources & References

**Date:** 2026-06-15  
**Topic:** Linux User Management

## 📚 Course Materials

- **Video Link:** KodeKloud - 100 Days of DevOps - Day 1
- **Duration:** ~15-20 minutes
- **Difficulty:** Beginner

## 🔗 Official Documentation

- [Linux useradd Manual](https://man7.org/linux/man-pages/man8/useradd.8.html) - Complete useradd command reference
- [Linux usermod Manual](https://man7.org/linux/man-pages/man8/usermod.8.html) - User modification command
- [Linux passwd File Format](https://man7.org/linux/man-pages/man5/passwd.5.html) - Understanding /etc/passwd structure
- [nologin Manual](https://man7.org/linux/man-pages/man8/nologin.8.html) - Non-login shell documentation

## 📖 Additional Reading

### Articles
- [Linux User Management Best Practices](https://www.redhat.com/sysadmin/linux-user-management) - Red Hat guide on user management
- [Understanding Linux Users and Groups](https://www.digitalocean.com/community/tutorials/how-to-view-system-users-in-linux-on-ubuntu) - DigitalOcean tutorial
- [Service Account Security](https://www.cyberciti.biz/tips/linux-security.html) - Security best practices for service accounts

### Tutorials
- [Linux User Administration Tutorial](https://www.tutorialspoint.com/unix/unix-user-administration.htm) - Comprehensive user management guide
- [Managing Users in Linux](https://linuxize.com/post/how-to-create-users-in-linux-using-the-useradd-command/) - Step-by-step useradd tutorial

## 🎥 Video Resources

- [Linux User Management Explained](https://www.youtube.com/results?search_query=linux+user+management) - YouTube tutorials
- [Understanding /etc/passwd and /etc/shadow](https://www.youtube.com/results?search_query=linux+passwd+shadow+files) - Deep dive into user files

## 🛠️ Tools & Software

| Tool | Purpose | Installation Link |
|------|---------|------------------|
| useradd | Create new users | Pre-installed on Linux |
| usermod | Modify existing users | Pre-installed on Linux |
| userdel | Delete users | Pre-installed on Linux |
| id | Display user information | Pre-installed on Linux |
| getent | Query system databases | Pre-installed on Linux |

## 📝 Cheat Sheets

### User Management Commands
```bash
# Create user
sudo useradd username
sudo useradd -s /sbin/nologin username  # with non-interactive shell
sudo useradd -m username                # with home directory
sudo useradd -m -s /bin/bash username   # with home and bash shell

# Modify user
sudo usermod -s /bin/bash username      # change shell
sudo usermod -L username                # lock account
sudo usermod -U username                # unlock account

# Delete user
sudo userdel username                   # delete user only
sudo userdel -r username                # delete user and home directory

# View user info
id username
getent passwd username
grep username /etc/passwd

# List all users
cat /etc/passwd
cut -d: -f1 /etc/passwd

# List users with specific shell
grep nologin /etc/passwd
grep bash /etc/passwd
```

### Shell Paths Reference
```bash
# Interactive shells
/bin/bash
/bin/sh
/bin/zsh
/bin/ksh

# Non-interactive shells
/sbin/nologin
/usr/sbin/nologin
/bin/false

# Check available shells
cat /etc/shells
```

## 💻 Code Examples

### Example 1: Create Service Account
```bash
# Create a service account for nginx
sudo useradd -r -s /sbin/nologin -c "Nginx Web Server" nginx

# Explanation:
# -r : Create system account (UID < 1000)
# -s : Specify shell
# -c : Add comment/description
```

### Example 2: Batch User Creation Script
```bash
#!/bin/bash
# Script to create multiple service accounts

USERS=("webapp" "dbuser" "apiuser")

for user in "${USERS[@]}"; do
    if id "$user" &>/dev/null; then
        echo "User $user already exists"
    else
        sudo useradd -s /sbin/nologin "$user"
        echo "Created user: $user"
    fi
done
```

### Example 3: User Audit Script
```bash
#!/bin/bash
# List all users with non-interactive shells

echo "Users with non-interactive shells:"
echo "=================================="
grep -E "nologin|false" /etc/passwd | cut -d: -f1,7 | column -t -s:
```

## 🌐 Community Resources

- **Forums:** 
  - [Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/tagged/user-management)
  - [r/linuxadmin](https://www.reddit.com/r/linuxadmin/)
  - [r/linux4noobs](https://www.reddit.com/r/linux4noobs/)

- **GitHub Repositories:**
  - [Linux System Administration Scripts](https://github.com/topics/linux-administration)
  - [DevOps Scripts Collection](https://github.com/topics/devops-scripts)

## 📊 Practice Platforms

- [KodeKloud Labs](https://kodekloud.com) - Hands-on Linux labs
- [Linux Journey](https://linuxjourney.com/) - Interactive Linux learning
- [OverTheWire - Bandit](https://overthewire.org/wargames/bandit/) - Linux command line practice
- [Katacoda](https://www.katacoda.com/) - Interactive Linux scenarios

## 🎯 Related Topics to Explore

- User groups and group management (`groupadd`, `groupmod`)
- File permissions and ownership (`chmod`, `chown`)
- sudo configuration and sudoers file
- SSH key-based authentication
- PAM (Pluggable Authentication Modules)
- User password policies
- SELinux user contexts
- LDAP and centralized user management

## 📌 Bookmarks

- [Linux Command Line Cheat Sheet](https://www.linuxtrainingacademy.com/linux-commands-cheat-sheet/)
- [Bash Scripting Guide](https://www.gnu.org/software/bash/manual/bash.html)
- [Linux Security Hardening Guide](https://www.cisecurity.org/cis-benchmarks/)

## 🔍 Quick Reference

### Common User Management Scenarios

1. **Create web server user:**
   ```bash
   sudo useradd -r -s /sbin/nologin -d /var/www -c "Web Server" www-data
   ```

2. **Create database user:**
   ```bash
   sudo useradd -r -s /sbin/nologin -d /var/lib/mysql -c "MySQL Server" mysql
   ```

3. **Create application user with home directory:**
   ```bash
   sudo useradd -m -s /sbin/nologin -c "Application User" appuser
   ```

4. **Check user's current shell:**
   ```bash
   getent passwd username | cut -d: -f7
   ```

5. **List all system users (UID < 1000):**
   ```bash
   awk -F: '$3 < 1000 {print $1}' /etc/passwd
   ```

---
**Last Updated:** 2026-06-15