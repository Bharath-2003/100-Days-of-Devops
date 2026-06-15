# Day 001 - Linux User Management

**Date:** 2026-06-15  
**Topic:** Creating Users with Non-Interactive Shell

## 📝 Key Concepts

### User Management in Linux
- Linux systems support multiple users with different privileges
- Each user has a unique username and user ID (UID)
- Users can have different shells assigned to them

### Shell Types
- **Interactive Shell**: Allows user to login and execute commands (e.g., `/bin/bash`, `/bin/zsh`)
- **Non-Interactive Shell**: Prevents user from logging in interactively (e.g., `/sbin/nologin`, `/bin/false`)

### Why Non-Interactive Shells?
- Security: Prevents unauthorized login attempts
- Service Accounts: Used for running services/daemons without login capability
- System Users: For processes that need file ownership but not login access

## 💡 Important Points

- The `/sbin/nologin` shell displays a message and exits when login is attempted
- The `/bin/false` shell simply exits without any message
- Non-interactive shells are commonly used for service accounts (e.g., www-data, mysql)
- Users with non-interactive shells can still own files and run processes

## 🔧 Commands/Tools Used

```bash
# Create user with non-interactive shell
sudo useradd -s /sbin/nologin john

# Alternative method
sudo useradd john -s /sbin/nologin

# Verify user creation
id john
grep john /etc/passwd

# Check user's shell
grep john /etc/passwd | cut -d: -f7

# View all users with nologin shell
grep nologin /etc/passwd
```

## 📊 User Account Structure

```
/etc/passwd format:
username:x:UID:GID:comment:home_directory:shell

Example:
john:x:1001:1001::/home/john:/sbin/nologin
```

## ❓ Questions & Clarifications

- Q: What's the difference between `/sbin/nologin` and `/bin/false`?
  A: `/sbin/nologin` displays a polite message before denying access, while `/bin/false` just exits silently.

- Q: Can a user with `/sbin/nologin` still run processes?
  A: Yes, the shell restriction only prevents interactive login. The user can still own files and processes.

## 🎯 Key Takeaways

1. Non-interactive shells enhance system security by preventing unauthorized logins
2. The `useradd` command with `-s` flag specifies the user's shell
3. Service accounts should always use non-interactive shells
4. User information is stored in `/etc/passwd` file

## 🔗 Related Topics

- User permissions and groups
- SSH key-based authentication
- Service account management
- PAM (Pluggable Authentication Modules)

---
**Next Steps:** Learn about user groups and permissions management