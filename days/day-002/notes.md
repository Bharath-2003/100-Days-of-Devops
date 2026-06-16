# Day 002 - Temporary User Setup with Expiry

**Date:** 2026-06-16  
**Topic:** User Account Expiration and Temporary Access Management

## 📝 Key Concepts

### User Account Expiration
- Linux allows setting expiration dates for user accounts
- Useful for temporary contractors, interns, or time-limited access
- Account automatically becomes inaccessible after expiry date
- Different from password expiry (which only expires the password)

### Why Use Account Expiry?
- **Security**: Automatically revoke access after a specific date
- **Compliance**: Meet regulatory requirements for temporary access
- **Automation**: No need to manually disable accounts
- **Audit Trail**: Clear record of when access was granted and expired

### Account Expiry vs Password Expiry

| Feature | Account Expiry | Password Expiry |
|---------|---------------|-----------------|
| What expires | Entire account | Only password |
| User can login | ❌ No | ✅ Yes (after changing password) |
| Command | `usermod -e` or `chage -E` | `chage -M` or `passwd -x` |
| Use case | Temporary users | Security policy |

## 💡 Important Points

- Expiry date format: `YYYY-MM-DD` (e.g., 2026-12-31)
- Account expires at **midnight** on the specified date
- Expired accounts cannot login but files remain intact
- Can be extended by changing the expiry date
- Setting expiry to `-1` removes the expiration
- Root user can always access expired account files

## 🔧 Commands/Tools Used

### Primary Command: `chage` (Change Age)
```bash
# Set account expiry date
sudo chage -E 2026-12-31 username

# View account aging information
sudo chage -l username

# Remove expiry (set to never expire)
sudo chage -E -1 username

# Set expiry to today (immediate expiry)
sudo chage -E 0 username
```

### Alternative Command: `usermod`
```bash
# Set account expiry date
sudo usermod -e 2026-12-31 username

# Remove expiry
sudo usermod -e "" username
```

### Create User with Expiry
```bash
# Method 1: Create then set expiry
sudo useradd -m -s /bin/bash tempuser
sudo passwd tempuser
sudo chage -E 2026-12-31 tempuser

# Method 2: Create with expiry in one command
sudo useradd -m -s /bin/bash -e 2026-12-31 tempuser
sudo passwd tempuser
```

## 📊 Account Expiry Workflow

```
Day 1: Create temporary user
├── sudo useradd -m -s /bin/bash contractor
├── sudo passwd contractor
└── sudo chage -E 2026-06-30 contractor

Day 1-30: User has access
├── contractor can login
├── contractor can work normally
└── All activities logged

Day 31 (After expiry):
├── contractor CANNOT login
├── "Your account has expired" message
├── Files still exist in /home/contractor
└── Admin can extend or delete account
```

## ❓ Questions & Clarifications

- Q: What happens to user's files after account expires?
  A: Files remain intact in their home directory. Only login access is blocked.

- Q: Can an expired account be reactivated?
  A: Yes, by extending the expiry date with `chage -E` or `usermod -e`.

- Q: Does account expiry affect running processes?
  A: No, processes started before expiry continue running. Only new logins are blocked.

- Q: How to check if an account is expired?
  A: Use `chage -l username` or check `/etc/shadow` file.

## 🎯 Key Takeaways

1. Use `chage -E` or `usermod -e` to set account expiry dates
2. Format: YYYY-MM-DD (e.g., 2026-12-31)
3. Expired accounts cannot login but files remain
4. Use `-E -1` to remove expiry (never expire)
5. Perfect for contractors, interns, or temporary access
6. Always verify expiry with `chage -l username`

## 🔗 Related Topics

- Password aging and expiry policies
- User account lifecycle management
- Automated account provisioning
- Compliance and audit requirements
- PAM (Pluggable Authentication Modules)

---
**Next Steps:** Learn about password policies and aging