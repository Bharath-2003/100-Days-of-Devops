# Day 004 - Answers & Solutions

**Date:** 2026-06-18  
**Topic:** Script Execution Permissions

## 📋 Lab Exercises

### Exercise 1: Make a Script Executable

**Question:**
```
Create a simple bash script and make it executable.
```

**Answer:**
```bash
# Step 1: Create a script
cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
echo "This script is now executable!"
EOF

# Step 2: Check current permissions
ls -l hello.sh
# Output: -rw-r--r-- 1 user group 65 Jun 18 10:00 hello.sh
# Notice: No execute permission (x)

# Step 3: Try to run (will fail)
./hello.sh
# Output: bash: ./hello.sh: Permission denied

# Step 4: Make it executable
chmod +x hello.sh

# Step 5: Verify permissions changed
ls -l hello.sh
# Output: -rwxr-xr-x 1 user group 65 Jun 18 10:00 hello.sh
# Notice: Now has execute permission (x)

# Step 6: Run the script
./hello.sh
# Output: Hello, World!
#         This script is now executable!
```

**Explanation:**
- `chmod +x` adds execute permission for owner, group, and others
- The `#!/bin/bash` (shebang) tells system which interpreter to use
- `./` is required to run scripts in current directory
- Without execute permission, you get "Permission denied" error

---

### Exercise 2: Understanding Permission Notation

**Question:**
```
What do these permission notations mean?
- rwxr-xr-x
- rw-r--r--
- rwx------
```

**Answer:**
```bash
# rwxr-xr-x (755)
# Owner: rwx (read, write, execute)
# Group: r-x (read, execute)
# Others: r-x (read, execute)
# Use case: Executable scripts that everyone can run

# rw-r--r-- (644)
# Owner: rw- (read, write)
# Group: r-- (read only)
# Others: r-- (read only)
# Use case: Regular files, documents

# rwx------ (700)
# Owner: rwx (read, write, execute)
# Group: --- (no permissions)
# Others: --- (no permissions)
# Use case: Private scripts, sensitive files
```

**Verification:**
```bash
# Create test files with different permissions
touch file1.sh file2.txt file3.sh

# Set different permissions
chmod 755 file1.sh
chmod 644 file2.txt
chmod 700 file3.sh

# View permissions
ls -l file*.{sh,txt}
# -rwxr-xr-x 1 user group 0 Jun 18 10:00 file1.sh
# -rw-r--r-- 1 user group 0 Jun 18 10:00 file2.txt
# -rwx------ 1 user group 0 Jun 18 10:00 file3.sh
```

---

### Exercise 3: Numeric vs Symbolic Permissions

**Question:**
```
Set permissions using both numeric and symbolic notation.
```

**Answer:**
```bash
# Create a test script
echo '#!/bin/bash' > test.sh
echo 'echo "Testing permissions"' >> test.sh

# Method 1: Numeric notation
chmod 755 test.sh
ls -l test.sh
# -rwxr-xr-x 1 user group 42 Jun 18 10:00 test.sh

# Method 2: Symbolic notation (same result)
chmod u=rwx,g=rx,o=rx test.sh
ls -l test.sh
# -rwxr-xr-x 1 user group 42 Jun 18 10:00 test.sh

# Method 3: Adding/removing permissions
chmod u+x test.sh    # Add execute for owner
chmod g-w test.sh    # Remove write for group
chmod o-x test.sh    # Remove execute for others
chmod a+r test.sh    # Add read for all

# Method 4: Setting specific permissions
chmod u=rwx,g=r,o= test.sh
ls -l test.sh
# -rwxr----- 1 user group 42 Jun 18 10:00 test.sh
```

**Numeric Permission Calculation:**
```
Owner: rwx = 4+2+1 = 7
Group: r-x = 4+0+1 = 5
Others: r-x = 4+0+1 = 5
Result: 755
```

---

### Exercise 4: Run Script Without Execute Permission

**Question:**
```
How can you run a script that doesn't have execute permission?
```

**Answer:**
```bash
# Create script without execute permission
cat > nox.sh << 'EOF'
#!/bin/bash
echo "Running without execute permission"
EOF

# Check permissions (no execute)
ls -l nox.sh
# -rw-r--r-- 1 user group 54 Jun 18 10:00 nox.sh

# Method 1: Try to run directly (will fail)
./nox.sh
# bash: ./nox.sh: Permission denied

# Method 2: Use bash interpreter
bash nox.sh
# Output: Running without execute permission

# Method 3: Use sh interpreter
sh nox.sh
# Output: Running without execute permission

# Method 4: Source the script
source nox.sh
# Output: Running without execute permission

# Method 5: Use dot notation
. nox.sh
# Output: Running without execute permission
```

**Explanation:**
- When you call the interpreter directly, it reads the file
- Execute permission is not required for the interpreter to read
- This is why `bash script.sh` works without execute permission
- However, `./script.sh` requires execute permission

---

### Exercise 5: Set Permissions for Different Users

**Question:**
```
Create a script that:
- Owner can read, write, and execute
- Group can read and execute
- Others have no permissions
```

**Answer:**
```bash
# Create the script
cat > private.sh << 'EOF'
#!/bin/bash
echo "This is a private script"
echo "Owner: full access"
echo "Group: read and execute"
echo "Others: no access"
EOF

# Method 1: Using numeric notation
chmod 750 private.sh

# Method 2: Using symbolic notation
chmod u=rwx,g=rx,o= private.sh

# Verify permissions
ls -l private.sh
# -rwxr-x--- 1 user group 120 Jun 18 10:00 private.sh

# Test as owner
./private.sh
# Output: This is a private script...

# Breakdown:
# 7 (owner): 4(r) + 2(w) + 1(x) = rwx
# 5 (group): 4(r) + 0(w) + 1(x) = r-x
# 0 (others): 0(r) + 0(w) + 0(x) = ---
```

---

## 🧪 Practice Problems

### Problem 1: Batch Permission Setting

**Description:**
Set execute permissions for all .sh files in a directory.

**Solution:**
```bash
# Method 1: Using find
find . -name "*.sh" -type f -exec chmod +x {} \;

# Method 2: Using find with xargs
find . -name "*.sh" -type f | xargs chmod +x

# Method 3: Using bash loop
for script in *.sh; do
    chmod +x "$script"
    echo "Made executable: $script"
done

# Method 4: Using chmod with multiple files
chmod +x *.sh

# Verify
ls -l *.sh
```

---

### Problem 2: Permission Audit Script

**Description:**
Create a script to audit file permissions in a directory.

**Solution:**
```bash
#!/bin/bash
# Permission Audit Script

echo "File Permission Audit"
echo "===================="
echo ""

# Check for executable scripts
echo "Executable Scripts:"
find . -type f -perm -u+x -name "*.sh" -exec ls -lh {} \;

echo ""
echo "Non-Executable Scripts:"
find . -type f ! -perm -u+x -name "*.sh" -exec ls -lh {} \;

echo ""
echo "World-Writable Files (Security Risk):"
find . -type f -perm -o+w -exec ls -lh {} \;

echo ""
echo "Files with SUID/SGID (Special Permissions):"
find . -type f \( -perm -4000 -o -perm -2000 \) -exec ls -lh {} \;

echo ""
echo "Summary:"
echo "--------"
echo "Total scripts: $(find . -name "*.sh" | wc -l)"
echo "Executable: $(find . -name "*.sh" -perm -u+x | wc -l)"
echo "Non-executable: $(find . -name "*.sh" ! -perm -u+x | wc -l)"
```

---

### Problem 3: Safe Script Deployment

**Description:**
Create a script to safely deploy scripts with proper permissions.

**Solution:**
```bash
#!/bin/bash
# Safe Script Deployment

SCRIPT_NAME=$1
DEPLOY_DIR="/usr/local/bin"
PERMISSION="755"

if [ -z "$SCRIPT_NAME" ]; then
    echo "Usage: $0 <script-name>"
    exit 1
fi

if [ ! -f "$SCRIPT_NAME" ]; then
    echo "Error: Script $SCRIPT_NAME not found"
    exit 1
fi

echo "Deploying $SCRIPT_NAME..."

# Backup if exists
if [ -f "$DEPLOY_DIR/$SCRIPT_NAME" ]; then
    echo "Backing up existing script..."
    sudo cp "$DEPLOY_DIR/$SCRIPT_NAME" "$DEPLOY_DIR/$SCRIPT_NAME.backup"
fi

# Copy script
echo "Copying script to $DEPLOY_DIR..."
sudo cp "$SCRIPT_NAME" "$DEPLOY_DIR/"

# Set permissions
echo "Setting permissions to $PERMISSION..."
sudo chmod $PERMISSION "$DEPLOY_DIR/$SCRIPT_NAME"

# Set ownership
echo "Setting ownership to root..."
sudo chown root:root "$DEPLOY_DIR/$SCRIPT_NAME"

# Verify
echo ""
echo "Deployment complete!"
ls -l "$DEPLOY_DIR/$SCRIPT_NAME"

# Test
echo ""
echo "Testing script..."
"$DEPLOY_DIR/$SCRIPT_NAME" --version 2>/dev/null || echo "Script deployed successfully"
```

---

## 🐛 Troubleshooting

### Issue 1: Permission Denied When Running Script
**Problem:** `./script.sh` returns "Permission denied"

**Solution:**
```bash
# Check current permissions
ls -l script.sh

# Add execute permission
chmod +x script.sh

# Verify
ls -l script.sh

# Run script
./script.sh
```

**Alternative:** Run with interpreter
```bash
bash script.sh
```

---

### Issue 2: Script Runs But Commands Fail
**Problem:** Script executes but internal commands fail

**Solution:**
```bash
# Check if script has proper shebang
head -1 script.sh
# Should see: #!/bin/bash

# If missing, add shebang
sed -i '1i#!/bin/bash' script.sh

# Or recreate with shebang
cat > script.sh << 'EOF'
#!/bin/bash
# Your script content here
EOF

chmod +x script.sh
```

---

### Issue 3: Cannot Execute Script in PATH
**Problem:** Script in PATH but still need to use `./`

**Solution:**
```bash
# Check if directory is in PATH
echo $PATH

# Add directory to PATH (temporary)
export PATH=$PATH:$(pwd)

# Add to PATH permanently (in ~/.bashrc)
echo 'export PATH=$PATH:~/scripts' >> ~/.bashrc
source ~/.bashrc

# Or move script to existing PATH directory
sudo mv script.sh /usr/local/bin/
sudo chmod +x /usr/local/bin/script.sh

# Now can run without ./
script.sh
```

---

## ✅ Verification Steps

1. **Check file permissions:**
   ```bash
   ls -l script.sh
   ```

2. **Verify execute permission:**
   ```bash
   ls -l script.sh | grep "x"
   ```

3. **Test script execution:**
   ```bash
   ./script.sh
   ```

4. **Check with stat command:**
   ```bash
   stat script.sh
   ```

5. **Verify numeric permissions:**
   ```bash
   stat -c "%a %n" script.sh
   ```

## 📌 Notes

- New files are created with 644 permissions (rw-r--r--)
- New directories are created with 755 permissions (rwxr-xr-x)
- Execute permission on directories allows entering (cd) into them
- Use `chmod +x` for quick execute permission
- Use `chmod 755` for standard script permissions
- Use `chmod 700` for private scripts
- Always include shebang (#!/bin/bash) in scripts
- Test scripts after changing permissions

## 🔐 Security Best Practices

1. **Principle of Least Privilege:**
   ```bash
   # Don't give more permissions than needed
   chmod 700 sensitive-script.sh  # Owner only
   ```

2. **Avoid World-Writable:**
   ```bash
   # Never use 777 unless absolutely necessary
   # Bad: chmod 777 script.sh
   # Good: chmod 755 script.sh
   ```

3. **Check Script Permissions Regularly:**
   ```bash
   find /path/to/scripts -type f -perm -o+w
   ```

4. **Use Specific Permissions:**
   ```bash
   # Be explicit about what you want
   chmod u=rwx,g=rx,o=r script.sh
   ```

5. **Audit Executable Files:**
   ```bash
   find / -type f -perm -u+x 2>/dev/null
   ```

---

**Status:** ✅ Completed