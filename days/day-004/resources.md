# Day 004 - Resources & References

**Date:** 2026-06-18  
**Topic:** Script Execution Permissions

## 📚 Course Materials

- **Video Link:** KodeKloud - 100 Days of DevOps - Day 4
- **Duration:** ~15-20 minutes
- **Difficulty:** Beginner

## 🔗 Official Documentation

- [chmod Manual](https://man7.org/linux/man-pages/man1/chmod.1.html) - Change file permissions
- [ls Manual](https://man7.org/linux/man-pages/man1/ls.1.html) - List directory contents
- [File Permissions Guide](https://www.gnu.org/software/coreutils/manual/html_node/File-permissions.html) - GNU Coreutils
- [Bash Scripting Guide](https://www.gnu.org/software/bash/manual/bash.html) - Official Bash manual

## 📖 Additional Reading

### Articles
- [Understanding Linux File Permissions](https://www.redhat.com/sysadmin/linux-file-permissions-explained) - Red Hat guide
- [Linux Permissions Explained](https://www.linux.com/training-tutorials/understanding-linux-file-permissions/) - Linux.com tutorial
- [chmod Command Examples](https://www.cyberciti.biz/faq/unix-linux-bsd-chmod-numeric-permissions-notation-command/) - CyberCiti guide

### Tutorials
- [File Permissions Tutorial](https://www.tutorialspoint.com/unix/unix-file-permission.htm) - Comprehensive tutorial
- [chmod Command Guide](https://linuxize.com/post/chmod-command-in-linux/) - Linuxize guide
- [Bash Scripting Basics](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-basics) - DigitalOcean

## 🎥 Video Resources

- [Linux File Permissions](https://www.youtube.com/results?search_query=linux+file+permissions+explained) - YouTube tutorials
- [chmod Command Tutorial](https://www.youtube.com/results?search_query=chmod+command+tutorial) - Video guides
- [Bash Scripting](https://www.youtube.com/results?search_query=bash+scripting+tutorial) - Script tutorials

## 🛠️ Tools & Software

| Tool | Purpose | Installation |
|------|---------|--------------|
| chmod | Change permissions | Pre-installed |
| ls | List files with permissions | Pre-installed |
| stat | Display file status | Pre-installed |
| find | Find files by permissions | Pre-installed |
| umask | Set default permissions | Pre-installed |

## 📝 Cheat Sheets

### Permission Commands
```bash
# View permissions
ls -l filename
ls -lh filename          # Human-readable sizes
stat filename            # Detailed file info
stat -c "%a %n" filename # Numeric permissions

# Change permissions - Symbolic
chmod +x script.sh       # Add execute for all
chmod -x script.sh       # Remove execute for all
chmod u+x script.sh      # Add execute for owner
chmod g+x script.sh      # Add execute for group
chmod o+x script.sh      # Add execute for others
chmod a+x script.sh      # Add execute for all (same as +x)

chmod u+rwx script.sh    # Add read, write, execute for owner
chmod g-w script.sh      # Remove write for group
chmod o-rwx script.sh    # Remove all for others

chmod u=rwx,g=rx,o=r script.sh  # Set specific permissions

# Change permissions - Numeric
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 700 private.sh     # rwx------
chmod 600 secret.txt     # rw-------
chmod 777 public.sh      # rwxrwxrwx (avoid!)

# Recursive permissions
chmod -R 755 directory/  # Apply to directory and contents
chmod -R +x scripts/     # Make all files executable

# Find files by permission
find . -perm 755         # Exact match
find . -perm -755        # At least these permissions
find . -perm /755        # Any of these permissions
```

### Permission Reference Table
```
Symbolic  Numeric  Binary   Meaning
--------  -------  ------   -------
---       0        000      No permissions
--x       1        001      Execute only
-w-       2        010      Write only
-wx       3        011      Write and execute
r--       4        100      Read only
r-x       5        101      Read and execute
rw-       6        110      Read and write
rwx       7        111      Read, write, and execute
```

### Common Permission Patterns
```bash
# Scripts
755  # rwxr-xr-x - Standard executable script
750  # rwxr-x--- - Group executable, others none
700  # rwx------ - Owner only executable

# Regular files
644  # rw-r--r-- - Standard file
640  # rw-r----- - Group readable, others none
600  # rw------- - Owner only

# Directories
755  # rwxr-xr-x - Standard directory
750  # rwxr-x--- - Group accessible, others none
700  # rwx------ - Owner only directory
```

## 💻 Code Examples

### Example 1: Create and Execute Script
```bash
#!/bin/bash
# create-and-run.sh - Demo script creation and execution

# Create a simple script
cat > demo.sh << 'EOF'
#!/bin/bash
echo "Hello from demo script!"
echo "Current user: $(whoami)"
echo "Current directory: $(pwd)"
EOF

echo "Script created: demo.sh"
ls -l demo.sh

echo ""
echo "Attempting to run without execute permission..."
./demo.sh 2>&1 || echo "Failed: Permission denied"

echo ""
echo "Running with bash interpreter..."
bash demo.sh

echo ""
echo "Adding execute permission..."
chmod +x demo.sh
ls -l demo.sh

echo ""
echo "Running with execute permission..."
./demo.sh

echo ""
echo "Cleanup..."
rm demo.sh
```

### Example 2: Permission Checker Script
```bash
#!/bin/bash
# check-permissions.sh - Check and report file permissions

check_file() {
    local file=$1
    
    if [ ! -e "$file" ]; then
        echo "Error: File '$file' does not exist"
        return 1
    fi
    
    echo "File: $file"
    echo "===================="
    
    # Get permissions
    perms=$(ls -l "$file" | awk '{print $1}')
    numeric=$(stat -c "%a" "$file")
    
    echo "Symbolic: $perms"
    echo "Numeric: $numeric"
    
    # Check owner permissions
    if [ -r "$file" ]; then echo "✓ Readable by you"; else echo "✗ Not readable by you"; fi
    if [ -w "$file" ]; then echo "✓ Writable by you"; else echo "✗ Not writable by you"; fi
    if [ -x "$file" ]; then echo "✓ Executable by you"; else echo "✗ Not executable by you"; fi
    
    echo ""
}

# Check multiple files
for file in "$@"; do
    check_file "$file"
done
```

### Example 3: Batch Permission Setter
```bash
#!/bin/bash
# set-script-permissions.sh - Set permissions for all scripts

SCRIPT_DIR=${1:-.}
PERMISSION=${2:-755}

echo "Setting permissions for scripts in: $SCRIPT_DIR"
echo "Permission: $PERMISSION"
echo ""

count=0
for script in "$SCRIPT_DIR"/*.sh; do
    if [ -f "$script" ]; then
        echo "Processing: $script"
        chmod "$PERMISSION" "$script"
        ls -l "$script"
        ((count++))
    fi
done

echo ""
echo "Total scripts processed: $count"
```

### Example 4: Permission Audit Script
```bash
#!/bin/bash
# audit-permissions.sh - Audit file permissions

echo "Permission Audit Report"
echo "======================"
echo "Date: $(date)"
echo ""

# Find world-writable files
echo "World-Writable Files (Security Risk):"
find . -type f -perm -o+w 2>/dev/null | head -10
echo ""

# Find executable files
echo "Executable Files:"
find . -type f -perm -u+x 2>/dev/null | head -10
echo ""

# Find files with unusual permissions
echo "Files with 777 permissions (Security Risk):"
find . -type f -perm 777 2>/dev/null
echo ""

# Summary
echo "Summary:"
echo "--------"
echo "Total files: $(find . -type f 2>/dev/null | wc -l)"
echo "Executable files: $(find . -type f -perm -u+x 2>/dev/null | wc -l)"
echo "World-writable files: $(find . -type f -perm -o+w 2>/dev/null | wc -l)"
```

### Example 5: Safe Script Installer
```bash
#!/bin/bash
# install-script.sh - Safely install script with proper permissions

SOURCE_SCRIPT=$1
INSTALL_DIR=${2:-/usr/local/bin}
PERMISSION=755

if [ -z "$SOURCE_SCRIPT" ]; then
    echo "Usage: $0 <script> [install-dir]"
    exit 1
fi

if [ ! -f "$SOURCE_SCRIPT" ]; then
    echo "Error: Script not found: $SOURCE_SCRIPT"
    exit 1
fi

SCRIPT_NAME=$(basename "$SOURCE_SCRIPT")

echo "Installing script: $SCRIPT_NAME"
echo "Destination: $INSTALL_DIR"
echo ""

# Check if destination exists
if [ -f "$INSTALL_DIR/$SCRIPT_NAME" ]; then
    echo "Warning: Script already exists"
    read -p "Overwrite? (y/n) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        echo "Installation cancelled"
        exit 0
    fi
    
    # Backup existing
    sudo cp "$INSTALL_DIR/$SCRIPT_NAME" "$INSTALL_DIR/$SCRIPT_NAME.backup"
    echo "Backup created: $SCRIPT_NAME.backup"
fi

# Copy script
sudo cp "$SOURCE_SCRIPT" "$INSTALL_DIR/"
echo "✓ Script copied"

# Set permissions
sudo chmod $PERMISSION "$INSTALL_DIR/$SCRIPT_NAME"
echo "✓ Permissions set to $PERMISSION"

# Set ownership
sudo chown root:root "$INSTALL_DIR/$SCRIPT_NAME"
echo "✓ Ownership set to root:root"

# Verify
echo ""
echo "Installation complete!"
ls -l "$INSTALL_DIR/$SCRIPT_NAME"

# Test
echo ""
echo "Testing script..."
if command -v "$SCRIPT_NAME" &> /dev/null; then
    echo "✓ Script is in PATH and executable"
else
    echo "⚠ Script may not be in PATH"
fi
```

## 🌐 Community Resources

- **Forums:** 
  - [Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/tagged/permissions)
  - [r/linuxquestions](https://www.reddit.com/r/linuxquestions/)
  - [Linux.org Forums](https://www.linux.org/forums/)

- **GitHub Repositories:**
  - [Bash Scripts Collection](https://github.com/topics/bash-scripts)
  - [Linux Administration Scripts](https://github.com/topics/linux-administration)

## 📊 Practice Platforms

- [KodeKloud Labs](https://kodekloud.com) - Hands-on Linux labs
- [Linux Journey](https://linuxjourney.com/) - Interactive learning
- [OverTheWire - Bandit](https://overthewire.org/wargames/bandit/) - Permission challenges

## 🎯 Related Topics to Explore

- File ownership (chown, chgrp)
- Special permissions (setuid, setgid, sticky bit)
- Access Control Lists (ACLs)
- umask and default permissions
- SELinux and AppArmor contexts
- File attributes (chattr, lsattr)
- Symbolic and hard links
- Directory permissions
- Process permissions and capabilities

## 📌 Bookmarks

- [Linux Permissions Calculator](https://chmod-calculator.com/) - Visual permission calculator
- [Explain Shell](https://explainshell.com/) - Command explanation tool
- [ShellCheck](https://www.shellcheck.net/) - Shell script analyzer

## 🔍 Quick Reference

### Permission Calculation
```
Read (r)    = 4
Write (w)   = 2
Execute (x) = 1

Examples:
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

### Common Scenarios

1. **Make script executable:**
   ```bash
   chmod +x script.sh
   ```

2. **Standard script permissions:**
   ```bash
   chmod 755 script.sh
   ```

3. **Private script:**
   ```bash
   chmod 700 script.sh
   ```

4. **Read-only file:**
   ```bash
   chmod 444 file.txt
   ```

5. **Shared directory:**
   ```bash
   chmod 775 shared/
   ```

## 🔐 Security Considerations

### Dangerous Permissions
```bash
# Avoid these unless absolutely necessary
chmod 777 file  # Everyone can do everything
chmod 666 file  # Everyone can read and write
chmod -R 777 /  # NEVER do this!
```

### Safe Permissions
```bash
# Use these instead
chmod 755 script.sh   # Standard executable
chmod 644 file.txt    # Standard file
chmod 700 private.sh  # Private executable
chmod 600 secret.txt  # Private file
```

### Permission Audit
```bash
# Find potentially dangerous permissions
find / -type f -perm -o+w 2>/dev/null  # World-writable
find / -type f -perm -4000 2>/dev/null # SUID files
find / -type f -perm -2000 2>/dev/null # SGID files
```

---
**Last Updated:** 2026-06-18