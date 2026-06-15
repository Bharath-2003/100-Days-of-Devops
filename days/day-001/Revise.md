# Day 001 - Quick Revision Guide

## 📚 Linux User Management Commands

### User Creation Commands

#### 1. Create Basic User
```bash
sudo useradd <username>
```
**Description:** Creates a basic user account without home directory or specific shell. User cannot login until password is set.

**Example:**
```bash
sudo useradd testuser
```

---

#### 2. Create Normal User (with Home Directory and Shell)
```bash
sudo useradd -m -s /bin/bash <username>
```
**Description:** Creates a normal user with:
- `-m` : Creates home directory at `/home/username`
- `-s /bin/bash` : Sets bash as the login shell (allows interactive login)

**Example:**
```bash
sudo useradd -m -s /bin/bash alice
```

---

#### 3. Create Service User (Non-Interactive)
```bash
sudo useradd -s /sbin/nologin <username>
```
**Description:** Creates a user with non-interactive shell. User cannot login but can own files and run processes. Used for service accounts.

**Example:**
```bash
sudo useradd -s /sbin/nologin john
```

---

### User Information Commands

#### 4. Check User ID and Groups
```bash
id <username>
```
**Description:** Displays user ID (UID), group ID (GID), and all groups the user belongs to.

**Example:**
```bash
id alice
# Output: uid=1001(alice) gid=1001(alice) groups=1001(alice)
```

---

#### 5. View User Entry in passwd File
```bash
grep <username> /etc/passwd
```
**Description:** Shows the user's entry in `/etc/passwd` file, displaying username, UID, GID, home directory, and shell.

**Example:**
```bash
grep alice /etc/passwd
# Output: alice:x:1001:1001::/home/alice:/bin/bash
```

---

### User Modification Commands

#### 6. Add User to Sudo Group
```bash
sudo usermod -aG sudo <username>
```
**Description:** Adds user to the sudo group, granting administrative privileges. User can execute commands with sudo.
- `-a` : Append (don't remove from other groups)
- `-G` : Supplementary groups

**Example:**
```bash
sudo usermod -aG sudo alice
```

**Note:** On RHEL/CentOS, use `wheel` instead of `sudo`:
```bash
sudo usermod -aG wheel alice
```

---

### User Deletion Commands

#### 7. Delete User (Keep Home Directory)
```bash
sudo deluser <username>
```
**Description:** Removes the user account but keeps the home directory and files.

**Example:**
```bash
sudo deluser alice
```

**Alternative command:**
```bash
sudo userdel alice
```

---

#### 8. Remove User from Sudo Group
```bash
sudo deluser <username> sudo
```
**Description:** Removes user from the sudo group, revoking administrative privileges.

**Example:**
```bash
sudo deluser alice sudo
```

**Alternative command:**
```bash
sudo gpasswd -d alice sudo
```

---

### User Query Commands

#### 9. Check User's Groups
```bash
groups <username>
```
**Description:** Lists all groups that the user belongs to.

**Example:**
```bash
groups alice
# Output: alice : alice sudo
```

---

### User Switching Commands

#### 10. Switch to Another User
```bash
su - <username>
```
**Description:** Switches to another user account with their environment. Requires the target user's password.
- `su -` : Loads user's environment (recommended)
- `su` : Switches user but keeps current environment

**Example:**
```bash
su - alice
# Enter alice's password
```

**To exit back:**
```bash
exit
# or press Ctrl+D
```

---

#### 11. Set User Password
```bash
sudo passwd <username>
```
**Description:** Sets or changes the password for a user. Required before user can login.

**Example:**
```bash
sudo passwd alice
# Enter new password twice
```

**Note:** The command in your list shows `su passwd <username>` but the correct command is `sudo passwd <username>` or just `passwd <username>` when logged in as that user.

---

### Identity Check Commands

#### 12. Check Current User
```bash
whoami
```
**Description:** Displays the username of the currently logged-in user.

**Example:**
```bash
whoami
# Output: alice
```

---

#### 13. Check Effective User (with Sudo)
```bash
sudo whoami
```
**Description:** Shows which user you become when using sudo (usually root).

**Example:**
```bash
sudo whoami
# Output: root
```

---

## 🎯 Quick Command Reference Table

| Command | Purpose | Requires Sudo |
|---------|---------|---------------|
| `sudo useradd <username>` | Create basic user | Yes |
| `sudo useradd -m -s /bin/bash <username>` | Create normal user with home & shell | Yes |
| `sudo useradd -s /sbin/nologin <username>` | Create service user (no login) | Yes |
| `id <username>` | Show user ID and groups | No |
| `grep <username> /etc/passwd` | View user details | No |
| `sudo usermod -aG sudo <username>` | Grant sudo access | Yes |
| `sudo deluser <username>` | Delete user | Yes |
| `sudo deluser <username> sudo` | Remove sudo access | Yes |
| `groups <username>` | List user's groups | No |
| `su - <username>` | Switch to user | No (needs user's password) |
| `sudo passwd <username>` | Set user password | Yes |
| `whoami` | Show current user | No |
| `sudo whoami` | Show effective user with sudo | Yes |

---

## 💡 Common Workflows

### Workflow 1: Create Normal User with Sudo Access
```bash
# 1. Create user with home directory and bash shell
sudo useradd -m -s /bin/bash alice

# 2. Set password
sudo passwd alice

# 3. Grant sudo access
sudo usermod -aG sudo alice

# 4. Verify
id alice
groups alice
```

---

### Workflow 2: Create Service Account
```bash
# 1. Create user with non-interactive shell
sudo useradd -s /sbin/nologin webapp

# 2. Verify (no password needed for service accounts)
id webapp
grep webapp /etc/passwd
```

---

### Workflow 3: Switch Users and Test Access
```bash
# 1. Check current user
whoami

# 2. Switch to another user
su - alice

# 3. Test sudo access
sudo whoami

# 4. Exit back
exit
```

---

## 🔍 Verification Commands

```bash
# Check if user exists
id username

# View user details
grep username /etc/passwd

# Check user's groups
groups username

# List all users
cat /etc/passwd | cut -d: -f1

# List users with sudo access
getent group sudo

# Check user's shell
grep username /etc/passwd | cut -d: -f7
```

---

## ⚠️ Important Notes

1. **Always use `sudo`** for user management commands (useradd, usermod, deluser, passwd)
2. **Use `-m` flag** when creating normal users to create home directory
3. **Use `/sbin/nologin`** for service accounts that shouldn't login
4. **Use `su -`** (with dash) to properly load user's environment
5. **Set password** with `sudo passwd username` before user can login
6. **Verify changes** with `id`, `groups`, or `grep /etc/passwd`

---

## 🎓 Key Takeaways

- **Basic user**: `sudo useradd username` (no home, no shell)
- **Normal user**: `sudo useradd -m -s /bin/bash username` (can login)
- **Service user**: `sudo useradd -s /sbin/nologin username` (cannot login)
- **Grant sudo**: `sudo usermod -aG sudo username`
- **Remove sudo**: `sudo deluser username sudo`
- **Switch user**: `su - username`
- **Check identity**: `whoami` or `id`

---

**Last Updated:** 2026-06-15