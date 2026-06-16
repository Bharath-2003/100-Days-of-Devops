# Day 002 - Resources & References

**Date:** 2026-06-16  
**Topic:** Temporary User Setup with Account Expiry

## 📚 Course Materials

- **Video Link:** KodeKloud - 100 Days of DevOps - Day 2
- **Duration:** ~15-20 minutes
- **Difficulty:** Beginner to Intermediate

## 🔗 Official Documentation

- [chage Manual](https://man7.org/linux/man-pages/man1/chage.1.html) - Change user password expiry information
- [usermod Manual](https://man7.org/linux/man-pages/man8/usermod.8.html) - Modify user account
- [useradd Manual](https://man7.org/linux/man-pages/man8/useradd.8.html) - Create new user with options
- [shadow File Format](https://man7.org/linux/man-pages/man5/shadow.5.html) - Understanding /etc/shadow

## 📖 Additional Reading

### Articles
- [Linux User Account Expiration](https://www.cyberciti.biz/faq/linux-howto-check-user-expiration-date/) - Comprehensive guide on account expiry
- [Managing Temporary Users](https://www.redhat.com/sysadmin/managing-users-linux) - Red Hat guide on user management
- [Password Aging in Linux](https://www.tecmint.com/manage-user-password-expiration-and-aging-in-linux/) - Understanding password policies

### Tutorials
- [chage Command Examples](https://www.geeksforgeeks.org/chage-command-in-linux-with-examples/) - Practical examples
- [User Account Management](https://linuxize.com/post/how-to-change-user-password-expiration-in-linux/) - Complete tutorial

## 🎥 Video Resources

- [Linux User Management](https://www.youtube.com/results?search_query=linux+user+account+expiry) - YouTube tutorials
- [chage Command Tutorial](https://www.youtube.com/results?search_query=linux+chage+command) - Video demonstrations

## 🛠️ Tools & Software

| Tool | Purpose | Installation Link |
|------|---------|------------------|
| chage | Manage password aging | Pre-installed on Linux |
| usermod | Modify user accounts | Pre-installed on Linux |
| useradd | Create users | Pre-installed on Linux |
| date | Date calculations | Pre-installed on Linux |

## 📝 Cheat Sheets

### Account Expiry Commands
```bash
# Set account expiry
sudo chage -E YYYY-MM-DD username
sudo usermod -e YYYY-MM-DD username

# View account info
sudo chage -l username

# Remove expiry (never expire)
sudo chage -E -1 username
sudo usermod -e "" username

# Expire immediately
sudo chage -E 0 username

# Create user with expiry
sudo useradd -m -s /bin/bash -e YYYY-MM-DD username

# Check expiry from shadow file
sudo grep username /etc/shadow | cut -d: -f8
```

### Date Calculations
```bash
# Get date 30 days from now
date -d "+30 days" +%Y-%m-%d

# Get date 90 days from now
date -d "+90 days" +%Y-%m-%d

# Get yesterday's date
date -d "yesterday" +%Y-%m-%d

# Get specific date
date -d "2026-12-31" +%Y-%m-%d

# Current date in ISO format
date +%Y-%m-%d
```

### Verification Commands
```bash
# Check if account is expired
sudo chage -l username | grep "Account expires"

# List all users with expiry
sudo awk -F: '$8 != "" && $8 != "0" {print $1, $8}' /etc/shadow

# Find expiring accounts (next 30 days)
# See scripts in answers.md

# Test account login
su - username
```

## 💻 Code Examples

### Example 1: Create Temporary Contractor Account
```bash
#!/bin/bash
# Create contractor account with 90-day expiry

USERNAME="contractor"
EXPIRY_DATE=$(date -d "+90 days" +%Y-%m-%d)

# Create user
sudo useradd -m -s /bin/bash -e $EXPIRY_DATE $USERNAME

# Set password
echo "$USERNAME:TempPass123!" | sudo chpasswd

# Force password change on first login
sudo chage -d 0 $USERNAME

# Display info
echo "User $USERNAME created"
echo "Expires on: $EXPIRY_DATE"
sudo chage -l $USERNAME
```

### Example 2: Extend Expiring Accounts
```bash
#!/bin/bash
# Extend accounts expiring in next 7 days by 30 days

CURRENT=$(date +%s)
WEEK=$((7 * 24 * 60 * 60))
THRESHOLD=$((CURRENT + WEEK))

while IFS=: read -r user _ _ _ _ _ _ _ expiry _; do
    if [ -n "$expiry" ] && [ "$expiry" != "0" ]; then
        expiry_sec=$((expiry * 24 * 60 * 60))
        
        if [ "$expiry_sec" -le "$THRESHOLD" ] && [ "$expiry_sec" -ge "$CURRENT" ]; then
            # Extend by 30 days
            new_date=$(date -d "+30 days" +%Y-%m-%d)
            sudo chage -E $new_date $user
            echo "Extended $user to $new_date"
        fi
    fi
done < /etc/shadow
```

### Example 3: Email Notification for Expiring Accounts
```bash
#!/bin/bash
# Send email notification for accounts expiring soon

ADMIN_EMAIL="admin@example.com"
DAYS_WARNING=7

CURRENT=$(date +%s)
WARNING=$((DAYS_WARNING * 24 * 60 * 60))
THRESHOLD=$((CURRENT + WARNING))

while IFS=: read -r user _ _ _ _ _ _ _ expiry _; do
    if [ -n "$expiry" ] && [ "$expiry" != "0" ]; then
        expiry_sec=$((expiry * 24 * 60 * 60))
        
        if [ "$expiry_sec" -le "$THRESHOLD" ] && [ "$expiry_sec" -ge "$CURRENT" ]; then
            expiry_date=$(date -d "@$expiry_sec" +%Y-%m-%d)
            days_left=$(( (expiry_sec - CURRENT) / 86400 ))
            
            # Send email
            echo "Account $user expires in $days_left days ($expiry_date)" | \
                mail -s "Account Expiry Warning" $ADMIN_EMAIL
        fi
    fi
done < /etc/shadow
```

### Example 4: Bulk User Creation with Expiry
```bash
#!/bin/bash
# Create multiple interns with same expiry date

EXPIRY_DATE="2026-08-31"
INTERNS=("intern1" "intern2" "intern3" "intern4" "intern5")

for intern in "${INTERNS[@]}"; do
    # Create user
    sudo useradd -m -s /bin/bash -e $EXPIRY_DATE $intern
    
    # Generate random password
    PASSWORD=$(openssl rand -base64 12)
    echo "$intern:$PASSWORD" | sudo chpasswd
    
    # Force password change
    sudo chage -d 0 $intern
    
    # Add to interns group
    sudo usermod -aG interns $intern
    
    echo "Created: $intern | Password: $PASSWORD | Expires: $EXPIRY_DATE"
done
```

## 🌐 Community Resources

- **Forums:** 
  - [Unix & Linux Stack Exchange - User Management](https://unix.stackexchange.com/questions/tagged/user-management)
  - [r/linuxadmin](https://www.reddit.com/r/linuxadmin/)
  - [Server Fault - Linux Users](https://serverfault.com/questions/tagged/linux+users)

- **GitHub Repositories:**
  - [User Management Scripts](https://github.com/topics/user-management)
  - [Linux Admin Tools](https://github.com/topics/linux-administration)

## 📊 Practice Platforms

- [KodeKloud Labs](https://kodekloud.com) - Hands-on Linux labs
- [Linux Journey](https://linuxjourney.com/) - Interactive learning
- [OverTheWire - Bandit](https://overthewire.org/wargames/bandit/) - Command line practice

## 🎯 Related Topics to Explore

- Password aging and expiry policies (`chage -M`, `chage -m`)
- Account locking and unlocking (`usermod -L`, `usermod -U`)
- Password complexity requirements (PAM configuration)
- Automated user provisioning and deprovisioning
- LDAP and centralized user management
- Audit logging for user activities
- Compliance requirements (SOX, HIPAA, PCI-DSS)
- Identity and Access Management (IAM) systems

## 📌 Bookmarks

- [Linux Security Hardening Guide](https://www.cisecurity.org/cis-benchmarks/)
- [User Account Best Practices](https://www.nist.gov/publications/guide-enterprise-password-management)
- [Bash Scripting Guide](https://www.gnu.org/software/bash/manual/bash.html)

## 🔍 Quick Reference

### Common Scenarios

1. **Create 30-day temporary account:**
   ```bash
   sudo useradd -m -s /bin/bash -e $(date -d "+30 days" +%Y-%m-%d) tempuser
   ```

2. **Create 90-day contractor account:**
   ```bash
   sudo useradd -m -s /bin/bash -e $(date -d "+90 days" +%Y-%m-%d) contractor
   ```

3. **Extend expiry by 30 days:**
   ```bash
   sudo chage -E $(date -d "+30 days" +%Y-%m-%d) username
   ```

4. **Check days until expiry:**
   ```bash
   sudo chage -l username | grep "Account expires"
   ```

5. **List all temporary accounts:**
   ```bash
   sudo awk -F: '$8 != "" && $8 != "0" {print $1}' /etc/shadow
   ```

## 📅 Account Lifecycle Management

### Phase 1: Onboarding
```bash
# Create account with expiry
sudo useradd -m -s /bin/bash -e YYYY-MM-DD username
sudo passwd username
sudo usermod -aG required_groups username
```

### Phase 2: Active Period
```bash
# Monitor expiry
sudo chage -l username

# Extend if needed
sudo chage -E new_date username
```

### Phase 3: Expiry Warning
```bash
# Send notifications 7 days before
# Send notifications 1 day before
```

### Phase 4: Post-Expiry
```bash
# Account expires automatically
# User cannot login
# Files remain intact
```

### Phase 5: Cleanup
```bash
# After confirmation, delete account
sudo userdel -r username
```

## 🔐 Compliance Considerations

### Regulatory Requirements
- **SOX**: Temporary access must be documented and time-limited
- **HIPAA**: Access to PHI must expire after need ends
- **PCI-DSS**: Temporary accounts must be disabled after use
- **GDPR**: Access to personal data must be time-limited

### Audit Trail
```bash
# Log all account creations
logger "Created temporary account: $username expires: $expiry_date"

# Track extensions
logger "Extended account: $username new expiry: $new_date"

# Monitor expired accounts
logger "Account expired: $username on $expiry_date"
```

---
**Last Updated:** 2026-06-16