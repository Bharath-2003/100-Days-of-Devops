# Day 002 - Quick Revision Guide

## 📚 Account Expiry Management Commands

### Core Commands

#### 1. Set Account Expiry Date
```bash
sudo chage -E YYYY-MM-DD <username>
```
**Description:** Sets the date when the user account will expire. Account cannot login after this date.

**Example:**
```bash
sudo chage -E 2026-12-31 tempuser
```

---

#### 2. Set Account Expiry (Alternative)
```bash
sudo usermod -e YYYY-MM-DD <username>
```
**Description:** Alternative command to set account expiry date using usermod.

**Example:**
```bash
sudo usermod -e 2026-12-31 tempuser
```

---

#### 3. View Account Aging Information
```bash
sudo chage -l <username>
```
**Description:** Displays detailed account aging information including expiry date, password change dates, and aging policies.

**Example:**
```bash
sudo chage -l tempuser
# Output shows: Account expires: Dec 31, 2026
```

---

#### 4. Remove Account Expiry (Never Expire)
```bash
sudo chage -E -1 <username>
```
**Description:** Removes the expiry date, making the account permanent (never expires).

**Example:**
```bash
sudo chage -E -1 tempuser
```

**Alternative:**
```bash
sudo usermod -e "" tempuser
```

---

#### 5. Expire Account Immediately
```bash
sudo chage -E 0 <username>
```
**Description:** Sets account to expire immediately. User cannot login but files remain.

**Example:**
```bash
sudo chage -E 0 tempuser
```

---

#### 6. Create User with Expiry Date
```bash
sudo useradd -m -s /bin/bash -e YYYY-MM-DD <username>
```
**Description:** Creates a new user with home directory, bash shell, and expiry date in one command.

**Example:**
```bash
sudo useradd -m -s /bin/bash -e 2026-12-31 tempuser
sudo passwd tempuser
```

---

### Date Calculation Commands

#### 7. Get Date N Days from Now
```bash
date -d "+N days" +%Y-%m-%d
```
**Description:** Calculates a future date N days from today in YYYY-MM-DD format.

**Examples:**
```bash
# 30 days from now
date -d "+30 days" +%Y-%m-%d

# 90 days from now
date -d "+90 days" +%Y-%m-%d

# 1 year from now
date -d "+1 year" +%Y-%m-%d
```

---

#### 8. Get Current Date in ISO Format
```bash
date +%Y-%m-%d
```
**Description:** Displays current date in YYYY-MM-DD format (ISO 8601).

**Example:**
```bash
date +%Y-%m-%d
# Output: 2026-06-16
```

---

#### 9. Get Yesterday's Date
```bash
date -d "yesterday" +%Y-%m-%d
```
**Description:** Gets yesterday's date, useful for immediate expiry.

**Example:**
```bash
sudo chage -E $(date -d "yesterday" +%Y-%m-%d) tempuser
```

---

### Verification Commands

#### 10. Check Account Expiry Status
```bash
sudo chage -l <username> | grep "Account expires"
```
**Description:** Quickly check just the account expiry date.

**Example:**
```bash
sudo chage -l tempuser | grep "Account expires"
# Output: Account expires: Dec 31, 2026
```

---

#### 11. View Shadow File Entry
```bash
sudo grep <username> /etc/shadow
```
**Description:** Shows the user's entry in /etc/shadow file (field 8 contains expiry in days since epoch).

**Example:**
```bash
sudo grep tempuser /etc/shadow
```

---

#### 12. List All Users with Expiry
```bash
sudo awk -F: '$8 != "" && $8 != "0" {print $1, $8}' /etc/shadow
```
**Description:** Lists all users that have an expiry date set.

**Example:**
```bash
sudo awk -F: '$8 != "" && $8 != "0" {print $1, $8}' /etc/shadow
# Output: tempuser 20818
```

---

## 🎯 Quick Command Reference Table

| Command | Purpose | Requires Sudo |
|---------|---------|---------------|
| `sudo chage -E YYYY-MM-DD user` | Set account expiry | Yes |
| `sudo usermod -e YYYY-MM-DD user` | Set account expiry (alt) | Yes |
| `sudo chage -l user` | View account info | Yes |
| `sudo chage -E -1 user` | Remove expiry (never expire) | Yes |
| `sudo chage -E 0 user` | Expire immediately | Yes |
| `sudo useradd -m -s /bin/bash -e DATE user` | Create user with expiry | Yes |
| `date -d "+30 days" +%Y-%m-%d` | Calculate future date | No |
| `date +%Y-%m-%d` | Current date ISO format | No |
| `sudo chage -l user \| grep "Account expires"` | Check expiry quickly | Yes |
| `sudo grep user /etc/shadow` | View shadow entry | Yes |

---

## 💡 Common Workflows

### Workflow 1: Create Temporary User (30 days)
```bash
# Calculate expiry date
EXPIRY=$(date -d "+30 days" +%Y-%m-%d)

# Create user with expiry
sudo useradd -m -s /bin/bash -e $EXPIRY tempuser

# Set password
sudo passwd tempuser

# Verify
sudo chage -l tempuser
```

---

### Workflow 2: Create Contractor (90 days)
```bash
# Create with 90-day expiry
sudo useradd -m -s /bin/bash -e $(date -d "+90 days" +%Y-%m-%d) contractor

# Set password
sudo passwd contractor

# Add to required groups
sudo usermod -aG developers contractor

# Verify
sudo chage -l contractor
```

---

### Workflow 3: Extend Expiring Account
```bash
# Check current expiry
sudo chage -l tempuser | grep "Account expires"

# Extend by 30 days from now
sudo chage -E $(date -d "+30 days" +%Y-%m-%d) tempuser

# Verify new expiry
sudo chage -l tempuser
```

---

### Workflow 4: Convert Temporary to Permanent
```bash
# Remove expiry
sudo chage -E -1 tempuser

# Verify
sudo chage -l tempuser | grep "Account expires"
# Should show: never
```

---

### Workflow 5: Immediately Disable Account
```bash
# Expire account now
sudo chage -E 0 tempuser

# Verify
sudo chage -l tempuser

# Test (should fail)
su - tempuser
# Output: Your account has expired
```

---

## 🔍 Verification Commands

```bash
# Check if user exists
id tempuser

# View complete account info
sudo chage -l tempuser

# Check expiry date only
sudo chage -l tempuser | grep "Account expires"

# View shadow file entry
sudo grep tempuser /etc/shadow | cut -d: -f1,8

# Test login (before expiry)
su - tempuser

# List all temporary accounts
sudo awk -F: '$8 != "" && $8 != "0" {print $1}' /etc/shadow
```

---

## ⚠️ Important Notes

1. **Date Format:** Always use YYYY-MM-DD (ISO 8601 format)
   - ✅ Correct: `2026-12-31`
   - ❌ Wrong: `12/31/2026` or `31-12-2026`

2. **Expiry Time:** Account expires at **midnight** on the specified date

3. **Files Remain:** Expired accounts cannot login but files stay in `/home/username/`

4. **Reactivation:** Can reactivate by setting new future date

5. **Root Access:** Root can always access expired account files

6. **Never Expire:** Use `-E -1` or `-e ""` to remove expiry

---

## 🎓 Key Takeaways

- **Set expiry:** `sudo chage -E YYYY-MM-DD username`
- **View expiry:** `sudo chage -l username`
- **Remove expiry:** `sudo chage -E -1 username`
- **Expire now:** `sudo chage -E 0 username`
- **Calculate dates:** `date -d "+N days" +%Y-%m-%d`
- **Create with expiry:** `sudo useradd -m -s /bin/bash -e DATE username`

---

## 🔄 Comparison: Account Expiry vs Password Expiry

| Feature | Account Expiry | Password Expiry |
|---------|---------------|-----------------|
| **Command** | `chage -E` | `chage -M` |
| **What expires** | Entire account | Only password |
| **Can login?** | ❌ No | ✅ Yes (after password change) |
| **Files affected** | ❌ No | ❌ No |
| **Use case** | Temporary users | Security policy |
| **Reactivation** | Set new date | Change password |

---

## 📅 Date Format Examples

```bash
# Correct formats
2026-12-31
2027-01-15
2026-06-30

# Calculate dates
date -d "+30 days" +%Y-%m-%d    # 30 days from now
date -d "+3 months" +%Y-%m-%d   # 3 months from now
date -d "+1 year" +%Y-%m-%d     # 1 year from now
date -d "2026-12-31" +%Y-%m-%d  # Specific date

# Get components
date +%Y    # Year: 2026
date +%m    # Month: 06
date +%d    # Day: 16
```

---

## 🚀 Pro Tips

1. **Use variables for dates:**
   ```bash
   EXPIRY=$(date -d "+90 days" +%Y-%m-%d)
   sudo useradd -m -s /bin/bash -e $EXPIRY contractor
   ```

2. **Bulk operations:**
   ```bash
   for user in user1 user2 user3; do
       sudo chage -E 2026-12-31 $user
   done
   ```

3. **Check before expiry:**
   ```bash
   # List accounts expiring soon
   sudo chage -l username | grep "Account expires"
   ```

4. **Document expiry dates:**
   ```bash
   echo "$(date): Created $username, expires $EXPIRY" >> /var/log/temp_users.log
   ```

5. **Set reminders:**
   ```bash
   # Add to crontab to check weekly
   0 9 * * 1 /path/to/check_expiring_accounts.sh
   ```

---

## 🔐 Security Best Practices

1. ✅ Always set expiry for temporary accounts
2. ✅ Review expiring accounts weekly
3. ✅ Notify users 7 days before expiry
4. ✅ Document reason for temporary access
5. ✅ Remove accounts after expiry (if no longer needed)
6. ✅ Use strong passwords even for temporary accounts
7. ✅ Force password change on first login: `sudo chage -d 0 username`

---

**Last Updated:** 2026-06-16