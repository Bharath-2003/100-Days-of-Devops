# Day 001 - Answers & Solutions

**Date:** 2026-06-15  
**Topic:** Linux User Management - Creating Non-Interactive User

## 📋 Lab Exercises

### Exercise 1: Create User with Non-Interactive Shell

**Question:**
```
Create a user named john with a non-interactive shell on App Server 1.
```

**Answer:**
```bash
# Method 1: Using useradd with -s flag
sudo useradd -s /sbin/nologin john

# Method 2: Alternative syntax
sudo useradd john -s /sbin/nologin

# Method 3: Create user first, then modify shell
sudo useradd john
sudo usermod -s /sbin/nologin john
```

**Explanation:**
- Step 1: Use `sudo` to execute with root privileges (required for user management)
- Step 2: `useradd` command creates a new user account
- Step 3: `-s /sbin/nologin` flag specifies the shell as non-interactive
- Step 4: `john` is the username to be created

**Output:**
```bash
# Verify user creation
$ id john
uid=1001(john) gid=1001(john) groups=1001(john)

# Check user entry in /etc/passwd
$ grep john /etc/passwd
john:x:1001:1001::/home/john:/sbin/nologin

# Verify the shell
$ grep john /etc/passwd | cut -d: -f7
/sbin/nologin
```

---

### Exercise 2: Verify Non-Interactive Shell Behavior

**Question:**
```
Verify that the user john cannot login interactively.
```

**Answer:**
```bash
# Try to switch to john user
su - john

# Or try SSH login (if SSH is configured)
ssh john@localhost
```

**Explanation:**
- Step 1: Attempt to switch to the john user account
- Step 2: The system will deny access with a message from `/sbin/nologin`

**Output:**
```
This account is currently not available.
```

---

## 🧪 Practice Problems

### Problem 1: Understanding Shell Options

**Description:**
What are the different non-interactive shell options available in Linux?

**Solution:**
```bash
# Common non-interactive shells:

# 1. /sbin/nologin - Displays message and exits
# Most commonly used for service accounts

# 2. /bin/false - Simply exits with false status
# More restrictive, no message displayed

# 3. /usr/sbin/nologin - Alternative path on some systems
# Same functionality as /sbin/nologin
```

**Why this works:**
- `/sbin/nologin` is preferred for user-facing accounts as it provides feedback
- `/bin/false` is more suitable for system accounts that should never attempt login
- Both prevent interactive shell access while allowing the user to own files and processes

---

### Problem 2: Bulk User Creation

**Description:**
Create multiple users with non-interactive shells.

**Solution:**
```bash
# Create multiple users at once
for user in john jane bob; do
    sudo useradd -s /sbin/nologin $user
    echo "Created user: $user"
done

# Verify all users
grep -E "john|jane|bob" /etc/passwd
```

---

## 🐛 Troubleshooting

### Issue 1: Permission Denied
**Problem:** "useradd: Permission denied" error when creating user

**Solution:** 
```bash
# Use sudo for administrative privileges
sudo useradd -s /sbin/nologin john
```

**Lesson Learned:** User management commands require root/sudo privileges

---

### Issue 2: Shell Path Not Found
**Problem:** Invalid shell path specified

**Solution:** 
```bash
# Check available shells
cat /etc/shells

# Verify nologin path
which nologin
# or
ls -l /sbin/nologin /usr/sbin/nologin
```

**Lesson Learned:** Always verify shell paths exist on your system before assigning them

---

### Issue 3: User Already Exists
**Problem:** "useradd: user 'john' already exists"

**Solution:** 
```bash
# Check if user exists
id john

# Delete existing user if needed
sudo userdel john

# Or modify existing user's shell
sudo usermod -s /sbin/nologin john
```

**Lesson Learned:** Check for existing users before creation, or use `usermod` to modify existing accounts

---

## ✅ Verification Steps

1. **Verify user creation:**
   ```bash
   id john
   ```

2. **Check user entry in passwd file:**
   ```bash
   grep john /etc/passwd
   ```

3. **Confirm non-interactive shell:**
   ```bash
   grep john /etc/passwd | cut -d: -f7
   ```

4. **Test login prevention:**
   ```bash
   su - john
   # Should display: "This account is currently not available."
   ```

5. **Check user's home directory (if created):**
   ```bash
   ls -ld /home/john
   ```

## 📌 Notes

- The `-s` flag in `useradd` specifies the login shell
- `/sbin/nologin` is the standard path for non-interactive shell
- User accounts with non-interactive shells can still own files and run processes
- This is a security best practice for service accounts
- Always verify user creation with `id` and `grep` commands

## 🔐 Security Best Practices

1. Use non-interactive shells for all service accounts
2. Regularly audit user accounts: `cat /etc/passwd`
3. Remove unused accounts promptly
4. Document the purpose of each service account
5. Use strong naming conventions for service accounts

---
**Status:** ✅ Completed