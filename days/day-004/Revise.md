# Day 004 - Quick Revision Guide

## 📚 Script Execution Permissions

### Core Concept
**Goal:** Understand and manage file permissions to make scripts executable

**Key Point:** New files are NOT executable by default - you must add execute permission

---

## 🔧 Essential Commands

#### 1. View File Permissions
```bash
ls -l filename
```
**Description:** Shows detailed file information including permissions.

**Example output:**
```bash
$ ls -l script.sh
-rw-r--r-- 1 user group 1234 Jun 18 10:00 script.sh
│││││││││
│││││││└└─ Others: r-- (read only)
││││││└─── Group: r-- (read only)
│││└└└──── Owner: rw- (read, write)
││└──────── Number of links
│└───────── File type (- = file, d = directory)
└────────── Always starts with file type
```

---

#### 2. Make Script Executable (Add Execute Permission)
```bash
chmod +x script.sh
```
**Description:** Adds execute permission for owner, group, and others.

**Example:**
```bash
$ ls -l script.sh
-rw-r--r-- 1 user group 100 Jun 18 10:00 script.sh

$ chmod +x script.sh

$ ls -l script.sh
-rwxr-xr-x 1 user group 100 Jun 18 10:00 script.sh
```

---

#### 3. Set Specific Permissions (Numeric)
```bash
chmod 755 script.sh
```
**Description:** Sets permissions using numeric notation (rwxr-xr-x).

**Common numeric permissions:**
- `755` = rwxr-xr-x (standard script)
- `700` = rwx------ (private script)
- `644` = rw-r--r-- (regular file)
- `600` = rw------- (private file)

---

#### 4. Set Specific Permissions (Symbolic)
```bash
chmod u=rwx,g=rx,o=rx script.sh
```
**Description:** Sets permissions using symbolic notation.

**Symbolic notation:**
- `u` = owner (user)
- `g` = group
- `o` = others
- `a` = all (u+g+o)

**Operations:**
- `+` = add permission
- `-` = remove permission
- `=` = set exact permission

---

#### 5. Run Executable Script
```bash
./script.sh
```
**Description:** Runs script in current directory (requires execute permission).

**Note:** The `./` is required for scripts in current directory.

---

#### 6. Run Script Without Execute Permission
```bash
bash script.sh
```
**Description:** Runs script using interpreter directly (no execute permission needed).

**Alternatives:**
```bash
sh script.sh
python script.py
perl script.pl
```

---

#### 7. Check Numeric Permissions
```bash
stat -c "%a %n" filename
```
**Description:** Shows numeric permission value.

**Example:**
```bash
$ stat -c "%a %n" script.sh
755 script.sh
```

---

#### 8. Remove Execute Permission
```bash
chmod -x script.sh
```
**Description:** Removes execute permission from all users.

**Example:**
```bash
$ chmod -x script.sh
$ ls -l script.sh
-rw-r--r-- 1 user group 100 Jun 18 10:00 script.sh
```

---

#### 9. Add Execute for Owner Only
```bash
chmod u+x script.sh
```
**Description:** Adds execute permission only for the file owner.

**Example:**
```bash
$ chmod u+x script.sh
$ ls -l script.sh
-rwxr--r-- 1 user group 100 Jun 18 10:00 script.sh
```

---

#### 10. Set Permissions Recursively
```bash
chmod -R 755 directory/
```
**Description:** Applies permissions to directory and all contents recursively.

**Example:**
```bash
chmod -R +x scripts/
```

---

## 🎯 Quick Permission Reference

### Permission Notation

| Symbolic | Numeric | Binary | Meaning |
|----------|---------|--------|---------|
| `---` | 0 | 000 | No permissions |
| `--x` | 1 | 001 | Execute only |
| `-w-` | 2 | 010 | Write only |
| `-wx` | 3 | 011 | Write + Execute |
| `r--` | 4 | 100 | Read only |
| `r-x` | 5 | 101 | Read + Execute |
| `rw-` | 6 | 110 | Read + Write |
| `rwx` | 7 | 111 | Read + Write + Execute |

### Numeric Calculation
```
Read (r)    = 4
Write (w)   = 2
Execute (x) = 1

Examples:
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
```

---

## 📋 Common Permission Patterns

### For Scripts
```bash
755  # rwxr-xr-x - Everyone can execute
750  # rwxr-x--- - Owner and group can execute
700  # rwx------ - Only owner can execute
```

### For Regular Files
```bash
644  # rw-r--r-- - Standard file
640  # rw-r----- - Group can read
600  # rw------- - Private file
```

### For Directories
```bash
755  # rwxr-xr-x - Standard directory
750  # rwxr-x--- - Group can access
700  # rwx------ - Private directory
```

---

## 💡 Complete Workflow

### Create and Execute a Script

```bash
# 1. Create script
cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
EOF

# 2. Check permissions (not executable)
ls -l hello.sh
# -rw-r--r-- 1 user group 31 Jun 18 10:00 hello.sh

# 3. Try to run (will fail)
./hello.sh
# bash: ./hello.sh: Permission denied

# 4. Make executable
chmod +x hello.sh

# 5. Verify permissions changed
ls -l hello.sh
# -rwxr-xr-x 1 user group 31 Jun 18 10:00 hello.sh

# 6. Run script
./hello.sh
# Hello, World!
```

---

## 🔍 Quick Command Reference Table

| Command | Purpose | Example |
|---------|---------|---------|
| `ls -l file` | View permissions | `ls -l script.sh` |
| `chmod +x file` | Make executable | `chmod +x script.sh` |
| `chmod 755 file` | Set standard script perms | `chmod 755 script.sh` |
| `chmod 700 file` | Set private script perms | `chmod 700 script.sh` |
| `chmod u+x file` | Add execute for owner | `chmod u+x script.sh` |
| `chmod -x file` | Remove execute | `chmod -x script.sh` |
| `./script.sh` | Run executable script | `./hello.sh` |
| `bash script.sh` | Run without execute perm | `bash hello.sh` |
| `stat -c "%a" file` | Show numeric permissions | `stat -c "%a" script.sh` |

---

## ⚠️ Common Issues & Solutions

### Issue 1: Permission Denied
```bash
# Problem
$ ./script.sh
bash: ./script.sh: Permission denied

# Solution
$ chmod +x script.sh
$ ./script.sh
```

### Issue 2: Command Not Found
```bash
# Problem
$ script.sh
bash: script.sh: command not found

# Solution - Use ./
$ ./script.sh
```

### Issue 3: No Shebang Line
```bash
# Problem - Script doesn't know which interpreter to use

# Solution - Add shebang at top of script
#!/bin/bash
```

---

## 🎓 Permission Examples

### Example 1: Standard Executable Script
```bash
chmod 755 script.sh
# Owner: rwx (read, write, execute)
# Group: r-x (read, execute)
# Others: r-x (read, execute)
```

### Example 2: Private Executable Script
```bash
chmod 700 script.sh
# Owner: rwx (read, write, execute)
# Group: --- (no permissions)
# Others: --- (no permissions)
```

### Example 3: Read-Only File
```bash
chmod 444 file.txt
# Owner: r-- (read only)
# Group: r-- (read only)
# Others: r-- (read only)
```

---

## 🔐 Security Best Practices

1. **Don't use 777:**
   ```bash
   # Bad
   chmod 777 script.sh
   
   # Good
   chmod 755 script.sh
   ```

2. **Use specific permissions:**
   ```bash
   # Be explicit
   chmod u=rwx,g=rx,o=r script.sh
   ```

3. **Private scripts:**
   ```bash
   # Only owner can execute
   chmod 700 sensitive-script.sh
   ```

4. **Check before executing:**
   ```bash
   ls -l script.sh
   ```

---

## 💻 Symbolic vs Numeric

### Symbolic Notation
```bash
# Add permissions
chmod u+x script.sh      # Add execute for owner
chmod g+w script.sh      # Add write for group
chmod o+r script.sh      # Add read for others
chmod a+x script.sh      # Add execute for all

# Remove permissions
chmod u-x script.sh      # Remove execute from owner
chmod g-w script.sh      # Remove write from group
chmod o-r script.sh      # Remove read from others

# Set exact permissions
chmod u=rwx script.sh    # Owner: rwx
chmod g=rx script.sh     # Group: r-x
chmod o=r script.sh      # Others: r--
```

### Numeric Notation
```bash
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 700 private.sh     # rwx------
chmod 600 secret.txt     # rw-------
```

---

## 🚀 Pro Tips

1. **Quick executable:**
   ```bash
   chmod +x *.sh
   ```

2. **Check permissions:**
   ```bash
   ls -l | grep "^-rwx"
   ```

3. **Find executable files:**
   ```bash
   find . -type f -perm -u+x
   ```

4. **Batch permission setting:**
   ```bash
   find . -name "*.sh" -exec chmod +x {} \;
   ```

5. **View octal permissions:**
   ```bash
   stat -c "%a %n" *
   ```

---

## 📊 Permission Breakdown

### Reading ls -l Output
```
-rwxr-xr-x 1 user group 1234 Jun 18 10:00 script.sh
│││││││││  │ │    │     │    │           │
│││││││││  │ │    │     │    │           └─ Filename
│││││││││  │ │    │     │    └─ Modification time
│││││││││  │ │    │     └─ Size in bytes
│││││││││  │ │    └─ Group owner
│││││││││  │ └─ User owner
│││││││││  └─ Number of hard links
│││││││└└─ Others permissions (r-x)
││││││└─── Group permissions (r-x)
│││└└└──── Owner permissions (rwx)
││└──────── File type (- = file, d = directory, l = link)
│└───────── Always present
└────────── File type indicator
```

---

## 🎯 Key Takeaways

1. **New files are NOT executable by default**
2. **Use `chmod +x` to make scripts executable**
3. **`./` is required to run scripts in current directory**
4. **755 is standard for executable scripts**
5. **700 is for private scripts**
6. **Can run without execute using interpreter: `bash script.sh`**
7. **Always include shebang: `#!/bin/bash`**
8. **Check permissions with `ls -l`**
9. **Avoid 777 permissions (security risk)**
10. **Use numeric (755) or symbolic (u=rwx,g=rx,o=rx) notation**

---

**Last Updated:** 2026-06-18