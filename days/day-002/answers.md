# Day 002 - Answers & Solutions

**Date:** 2026-06-16  
**Topic:** Temporary User Setup with Expiry

## 📋 Lab Exercises

### Exercise 1: Create Temporary User with Expiry Date

**Question:**
```
Create a temporary user account that expires on a specific date.
```

**Answer:**
```bash
# Method 1: Create user then set expiry
sudo useradd -m -s /bin/bash tempuser
sudo passwd tempuser
sudo chage -E 2026-12-31 tempuser

# Method 2: Create user with expiry in one command
sudo useradd -m -s /bin/bash -e 2026-12-31 tempuser
sudo passwd tempuser

# Method 3: Using usermod after creation
sudo useradd -m -s /bin/bash tempuser
sudo passwd tempuser
sudo usermod -e 2026-12-31 tempuser
```

**Explanation:**
- Step 1: Create user with home directory (`-m`) and bash shell (`-s /bin/bash`)
- Step 2: Set password for the user
- Step 3: Set account expiry date using `chage -E` or `usermod -e`
- Date format: YYYY-MM-DD

**Output:**
```bash
# Verify user creation
$ id tempuser
uid=1002(tempuser) gid=1002(tempuser) groups=1002(tempuser)

# Check expiry date
$ sudo chage -l tempuser
Last password change                                    : Jun 16, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Dec 31, 2026
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

---

### Exercise 2: Verify Account Expiry

**Question:**
```
How do you check if a user account has an expiry date set?
```

**Answer:**
```bash
# Method 1: Using chage command (detailed)
sudo chage -l tempuser

# Method 2: Check /etc/shadow file
sudo grep tempuser /etc/shadow

# Method 3: Using getent
sudo getent shadow tempuser
```

**Explanation:**
- `chage -l` shows human-readable account aging information
- `/etc/shadow` contains expiry date in days since epoch (field 8)
- Empty or 0 in field 8 means no expiry

**Output:**
```bash
$ sudo chage -l tempuser
Last password change                                    : Jun 16, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Dec 31, 2026
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

---

### Exercise 3: Extend Account Expiry

**Question:**
```
A contractor's account is expiring soon. How do you extend the expiry date?
```

**Answer:**
```bash
# Method 1: Using chage
sudo chage -E 2027-06-30 tempuser

# Method 2: Using usermod
sudo usermod -e 2027-06-30 tempuser

# Verify the change
sudo chage -l tempuser
```

**Explanation:**
- Simply set a new expiry date in the future
- User can continue working without interruption
- No need to recreate the account

**Output:**
```bash
$ sudo chage -l tempuser
Account expires                                         : Jun 30, 2027
```

---

### Exercise 4: Remove Account Expiry

**Question:**
```
How do you make a temporary account permanent (remove expiry)?
```

**Answer:**
```bash
# Method 1: Using chage (set to -1 for never expire)
sudo chage -E -1 tempuser

# Method 2: Using usermod (empty string)
sudo usermod -e "" tempuser

# Verify
sudo chage -l tempuser
```

**Explanation:**
- `-E -1` or empty string removes the expiry date
- Account becomes permanent
- User can login indefinitely

**Output:**
```bash
$ sudo chage -l tempuser
Account expires                                         : never
```

---

### Exercise 5: Expire Account Immediately

**Question:**
```
How do you immediately expire an account without deleting it?
```

**Answer:**
```bash
# Method 1: Set expiry to today or past date
sudo chage -E 0 tempuser

# Method 2: Set to yesterday's date
sudo chage -E $(date -d "yesterday" +%Y-%m-%d) tempuser

# Method 3: Lock the account (alternative)
sudo usermod -L tempuser

# Verify
sudo chage -l tempuser
```

**Explanation:**
- Setting expiry to 0 or past date immediately expires the account
- User cannot login but files remain
- Can be reactivated by setting future date

**Output:**
```bash
$ sudo chage -l tempuser
Account expires                                         : Jan 01, 1970

# When user tries to login:
$ su - tempuser
Your account has expired; please contact your system administrator
```

---

## 🧪 Practice Problems

### Problem 1: Contractor Onboarding

**Description:**
Create a contractor account that expires in 90 days from today.

**Solution:**
```bash
# Calculate expiry date (90 days from now)
EXPIRY_DATE=$(date -d "+90 days" +%Y-%m-%d)

# Create user with expiry
sudo useradd -m -s /bin/bash -e $EXPIRY_DATE contractor
sudo passwd contractor

# Verify
sudo chage -l contractor
echo "Account expires on: $EXPIRY_DATE"
```

**Why this works:**
- `date -d "+90 days"` calculates future date
- Variable stores the date for reuse
- Account automatically expires after 90 days

---

### Problem 2: Bulk Temporary Users

**Description:**
Create multiple temporary users with the same expiry date.

**Solution:**
```bash
#!/bin/bash
# Script to create multiple temporary users

EXPIRY_DATE="2026-12-31"
USERS=("intern1" "intern2" "intern3")

for user in "${USERS[@]}"; do
    sudo useradd -m -s /bin/bash -e $EXPIRY_DATE "$user"
    echo "Created user: $user with expiry: $EXPIRY_DATE"
    
    # Set temporary password
    echo "$user:TempPass123!" | sudo chpasswd
    
    # Force password change on first login
    sudo chage -d 0 "$user"
done

# Verify all users
for user in "${USERS[@]}"; do
    echo "=== $user ==="
    sudo chage -l "$user" | grep "Account expires"
done
```

---

### Problem 3: Audit Expiring Accounts

**Description:**
Find all accounts that will expire in the next 30 days.

**Solution:**
```bash
#!/bin/bash
# Script to find accounts expiring soon

echo "Accounts expiring in the next 30 days:"
echo "======================================"

# Get current date in days since epoch
CURRENT_DATE=$(date +%s)
THIRTY_DAYS=$((30 * 24 * 60 * 60))
THRESHOLD=$((CURRENT_DATE + THIRTY_DAYS))

# Check each user
while IFS=: read -r username _ _ _ _ _ _ _ expiry _; do
    if [ -n "$expiry" ] && [ "$expiry" != "0" ] && [ "$expiry" != "" ]; then
        # Convert expiry from days since epoch to seconds
        expiry_seconds=$((expiry * 24 * 60 * 60))
        
        if [ "$expiry_seconds" -le "$THRESHOLD" ] && [ "$expiry_seconds" -ge "$CURRENT_DATE" ]; then
            expiry_date=$(date -d "@$expiry_seconds" +%Y-%m-%d)
            echo "User: $username - Expires: $expiry_date"
        fi
    fi
done < /etc/shadow
```

---

## 🐛 Troubleshooting

### Issue 1: Account Expired Unexpectedly
**Problem:** User reports they cannot login, account shows as expired

**Solution:**
```bash
# Check account status
sudo chage -l username

# Extend expiry date
sudo chage -E 2027-12-31 username

# Or remove expiry
sudo chage -E -1 username
```

**Lesson Learned:** Always set expiry dates with buffer time and notify users in advance

---

### Issue 2: Cannot Set Expiry Date
**Problem:** "chage: invalid date" error

**Solution:**
```bash
# Correct format: YYYY-MM-DD
sudo chage -E 2026-12-31 username

# NOT: MM/DD/YYYY or DD-MM-YYYY
# NOT: 12/31/2026 ❌
# NOT: 31-12-2026 ❌
```

**Lesson Learned:** Always use ISO date format (YYYY-MM-DD)

---

### Issue 3: Expired Account Still Logging In
**Problem:** Account shows expired but user can still login

**Solution:**
```bash
# Check if expiry is actually set
sudo chage -l username

# Verify /etc/shadow
sudo grep username /etc/shadow

# Force immediate expiry
sudo chage -E 0 username

# Or lock the account
sudo usermod -L username
```

**Lesson Learned:** Verify expiry settings after making changes

---

## ✅ Verification Steps

1. **Verify user creation:**
   ```bash
   id tempuser
   ```

2. **Check expiry date:**
   ```bash
   sudo chage -l tempuser | grep "Account expires"
   ```

3. **Test login before expiry:**
   ```bash
   su - tempuser
   # Should succeed
   ```

4. **Simulate expired account:**
   ```bash
   sudo chage -E 0 tempuser
   su - tempuser
   # Should fail with "account expired" message
   ```

5. **Verify files remain after expiry:**
   ```bash
   sudo ls -la /home/tempuser
   # Files should still exist
   ```

## 📌 Notes

- Account expiry is different from password expiry
- Expired accounts cannot login but files remain intact
- Use `chage -E -1` to remove expiry (never expire)
- Date format must be YYYY-MM-DD
- Account expires at midnight on the specified date
- Root can always access expired account files
- Consider setting up email notifications for expiring accounts

## 🔐 Security Best Practices

1. **Set expiry for all temporary accounts**
   ```bash
   sudo useradd -m -s /bin/bash -e 2026-12-31 contractor
   ```

2. **Regular audits of expiring accounts**
   ```bash
   # List all accounts with expiry
   sudo awk -F: '$8 != "" && $8 != "0" {print $1}' /etc/shadow
   ```

3. **Notify users before expiry**
   - Send email 7 days before expiry
   - Send email 1 day before expiry
   - Provide extension process

4. **Document expiry dates**
   - Keep spreadsheet of temporary accounts
   - Note reason for access
   - Track extensions

5. **Clean up expired accounts**
   ```bash
   # After confirming no longer needed
   sudo userdel -r expired_user
   ```

---

**Status:** ✅ Completed