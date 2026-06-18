# Day 004 - Script Execution Permissions

**Date:** 2026-06-18  
**Topic:** Linux File Permissions and Script Execution

## 📝 Key Concepts

### Linux File Permissions
- Every file and directory in Linux has permissions
- Permissions control who can read, write, or execute files
- Three types of users: Owner, Group, Others
- Three types of permissions: Read (r), Write (w), Execute (x)

### Permission Types

| Permission | Symbol | Numeric | File | Directory |
|------------|--------|---------|------|-----------|
| **Read** | r | 4 | View file contents | List directory contents |
| **Write** | w | 2 | Modify file | Create/delete files in directory |
| **Execute** | x | 1 | Run as program/script | Enter directory (cd) |

### User Categories

| Category | Description | Example |
|----------|-------------|---------|
| **Owner (u)** | User who owns the file | Creator of the file |
| **Group (g)** | Users in the file's group | Team members |
| **Others (o)** | Everyone else | All other users |

### Why Script Execution Permissions Matter
- Scripts need execute permission to run directly
- Without execute permission, you must use interpreter explicitly
- Security: Prevents accidental execution of malicious scripts
- Best practice: Only make scripts executable when needed

## 💡 Important Points

- New files are NOT executable by default (security feature)
- Execute permission is required to run scripts directly
- Can still run scripts without execute permission using interpreter
- Permissions are displayed in `ls -l` output
- Numeric permissions: Read(4) + Write(2) + Execute(1)

## 🔧 Commands/Tools Used

### View File Permissions
```bash
# Long listing format (shows permissions)
ls -l filename

# Example output:
# -rw-r--r-- 1 user group 1234 Jun 18 10:00 script.sh
# │││││││││
# │││││││└└─ Others permissions (r--)
# ││││││└─── Group permissions (r--)
# │││└└└──── Owner permissions (rw-)
# ││└──────── Number of hard links
# │└───────── File type (- = regular file, d = directory)
```

### Make Script Executable
```bash
# Add execute permission for owner
chmod +x script.sh

# Add execute permission for everyone
chmod a+x script.sh

# Add execute permission for owner only
chmod u+x script.sh

# Using numeric notation (755 = rwxr-xr-x)
chmod 755 script.sh

# Using numeric notation (700 = rwx------)
chmod 700 script.sh
```

### Remove Execute Permission
```bash
# Remove execute permission
chmod -x script.sh

# Remove execute for everyone
chmod a-x script.sh

# Using numeric notation (644 = rw-r--r--)
chmod 644 script.sh
```

### Run Scripts
```bash
# With execute permission
./script.sh

# Without execute permission (using interpreter)
bash script.sh
sh script.sh
python script.py
```

## 📊 Permission Notation

### Symbolic Notation
```
-rwxr-xr-x
│││││││││
│││││││└└─ Others: r-x (read, execute)
││││││└─── Group: r-x (read, execute)
│││└└└──── Owner: rwx (read, write, execute)
││└──────── Number of links
│└───────── File type
└────────── Always starts with file type
```

### Numeric (Octal) Notation
```
Permission  Binary  Octal
---------   ------  -----
---         000     0
--x         001     1
-w-         010     2
-wx         011     3
r--         100     4
r-x         101     5
rw-         110     6
rwx         111     7
```

### Common Permission Combinations
```bash
# Scripts
755 (rwxr-xr-x)  # Owner can modify, everyone can execute
700 (rwx------)  # Only owner can read, write, execute
750 (rwxr-x---)  # Owner full, group execute, others none

# Regular files
644 (rw-r--r--)  # Owner can modify, others can read
600 (rw-------)  # Only owner can read/write
640 (rw-r-----)  # Owner can modify, group can read

# Directories
755 (rwxr-xr-x)  # Standard directory permissions
700 (rwx------)  # Private directory
750 (rwxr-x---)  # Owner full, group can enter
```

## ❓ Questions & Clarifications

- Q: Why can't I run my script even though I created it?
  A: New files don't have execute permission by default. Use `chmod +x script.sh`.

- Q: What's the difference between `chmod +x` and `chmod 755`?
  A: `+x` adds execute to existing permissions, `755` sets specific permissions (rwxr-xr-x).

- Q: Can I run a script without execute permission?
  A: Yes, by calling the interpreter directly: `bash script.sh` or `python script.py`.

- Q: What does `./` mean before script name?
  A: It means "current directory". Required to run scripts not in PATH.

## 🎯 Key Takeaways

1. Use `chmod +x script.sh` to make scripts executable
2. Use `ls -l` to view current permissions
3. Execute permission is required to run scripts directly with `./`
4. Can still run without execute permission using interpreter
5. Common script permission: 755 (rwxr-xr-x)
6. Security: Only make scripts executable when needed
7. Use `chmod 700` for private scripts (owner only)

## 🔗 Related Topics

- File ownership (chown, chgrp)
- Special permissions (setuid, setgid, sticky bit)
- umask (default permissions for new files)
- Access Control Lists (ACLs)
- SELinux and AppArmor contexts
- Script shebang lines (#!/bin/bash)

---
**Next Steps:** Learn about file ownership and special permissions