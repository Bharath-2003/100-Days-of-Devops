# Day 6: Create a Cron Job - Quick Revision Guide

## 🎯 Quick Summary

Cron is a time-based job scheduler in Linux that automates repetitive tasks by running commands or scripts at specified intervals.

---

## ⚡ Key Concepts (5-Minute Review)

### 1. Cron Syntax
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─ Day of week (0-7)
│ │ │ └─── Month (1-12)
│ │ └───── Day of month (1-31)
│ └─────── Hour (0-23)
└───────── Minute (0-59)
```

### 2. Essential Commands
```bash
crontab -e    # Edit crontab
crontab -l    # List crontab entries
crontab -r    # Remove crontab
crontab -u user -e    # Edit another user's crontab
```

### 3. Special Strings
```bash
@reboot   # Run at startup
@daily    # Run once a day (0 0 * * *)
@hourly   # Run once an hour (0 * * * *)
@weekly   # Run once a week (0 0 * * 0)
@monthly  # Run once a month (0 0 1 * *)
@yearly   # Run once a year (0 0 1 1 *)
```

---

## 📝 Common Examples (Quick Reference)

```bash
# Every minute
* * * * * /path/to/script.sh

# Every 5 minutes
*/5 * * * * /path/to/script.sh

# Every hour at minute 30
30 * * * * /path/to/script.sh

# Daily at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# First day of month at midnight
0 0 1 * * /path/to/script.sh

# Weekdays at 6 PM
0 18 * * 1-5 /path/to/script.sh

# Multiple times (9 AM, 12 PM, 6 PM)
0 9,12,18 * * * /path/to/script.sh
```

---

## 🔧 Quick Setup Steps

### Create a Simple Cron Job
```bash
# 1. Create script
echo '#!/bin/bash' > ~/myscript.sh
echo 'echo "$(date): Job ran" >> ~/cron.log' >> ~/myscript.sh

# 2. Make executable
chmod +x ~/myscript.sh

# 3. Add to crontab
crontab -e
# Add: * * * * * /home/username/myscript.sh

# 4. Verify
crontab -l
```

---

## 🐛 Quick Troubleshooting

### Job Not Running?
```bash
# Check cron service
sudo systemctl status cron

# View logs
grep CRON /var/log/syslog

# Test manually
/path/to/script.sh
```

### No Output?
```bash
# Add logging to crontab
* * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1

# Check mail
mail
```

### Permission Issues?
```bash
# Check permissions
ls -l /path/to/script.sh

# Make executable
chmod +x /path/to/script.sh
```

---

## 💡 Best Practices Checklist

- [ ] Use absolute paths for all commands and files
- [ ] Make scripts executable (`chmod +x`)
- [ ] Add logging/output redirection
- [ ] Test scripts manually before scheduling
- [ ] Use comments in crontab
- [ ] Set environment variables if needed
- [ ] Monitor cron job execution
- [ ] Implement error handling in scripts

---

## 🎓 Must-Remember Points

1. **Cron runs with minimal environment** - Always use absolute paths
2. **Syntax matters** - Use crontab.guru to validate
3. **Logging is crucial** - Redirect output to files
4. **Test first** - Run scripts manually before scheduling
5. **Monitor regularly** - Check logs for failures

---

## 📊 Cron Syntax Cheat Sheet

| Field | Values | Special Characters |
|-------|--------|-------------------|
| Minute | 0-59 | * , - / |
| Hour | 0-23 | * , - / |
| Day of Month | 1-31 | * , - / |
| Month | 1-12 | * , - / |
| Day of Week | 0-7 (0=Sun) | * , - / |

**Special Characters:**
- `*` = Any value
- `,` = List (1,3,5)
- `-` = Range (1-5)
- `/` = Step (*/5)

---

## 🔍 Quick Verification Commands

```bash
# Check cron service
systemctl status cron

# View crontab
crontab -l

# Check logs
tail -20 /var/log/syslog | grep CRON

# Test cron environment
* * * * * env > /tmp/cron-env.txt
```

---

## 📚 Common Use Cases

### 1. Daily Backup
```bash
0 2 * * * tar -czf /backup/data_$(date +\%Y\%m\%d).tar.gz /data
```

### 2. Log Rotation
```bash
0 0 * * * find /var/log -name "*.log" -mtime +7 -delete
```

### 3. System Monitoring
```bash
*/5 * * * * df -h | grep -v "tmpfs" >> /var/log/disk-usage.log
```

### 4. Website Check
```bash
*/10 * * * * curl -f https://example.com || echo "Site down" | mail -s "Alert" admin@example.com
```

---

## ⚠️ Common Mistakes to Avoid

1. ❌ Using relative paths
   ✅ Use absolute paths: `/home/user/script.sh`

2. ❌ Forgetting to make script executable
   ✅ Run: `chmod +x script.sh`

3. ❌ Not redirecting output
   ✅ Add: `>> /var/log/job.log 2>&1`

4. ❌ Assuming environment variables
   ✅ Set PATH in crontab or script

5. ❌ Not testing manually first
   ✅ Test: `/path/to/script.sh`

---

## 🚀 Quick Practice Exercise

**Task:** Create a cron job that backs up a directory every day at 3 AM

**Solution:**
```bash
# 1. Create backup script
cat > ~/backup.sh << 'EOF'
#!/bin/bash
tar -czf ~/backups/backup_$(date +%Y%m%d).tar.gz ~/important-data
find ~/backups -name "backup_*.tar.gz" -mtime +7 -delete
EOF

# 2. Make executable
chmod +x ~/backup.sh

# 3. Add to crontab
crontab -e
# Add: 0 3 * * * /home/username/backup.sh

# 4. Verify
crontab -l
```

---

## 📖 Related Commands

```bash
# View system cron jobs
ls -la /etc/cron.{hourly,daily,weekly,monthly}

# Check cron access
cat /etc/cron.allow
cat /etc/cron.deny

# View another user's crontab (as root)
sudo crontab -u username -l

# Edit system-wide crontab
sudo vim /etc/crontab
```

---

## 🎯 Interview Questions (Quick Prep)

**Q1: What is the difference between cron and anacron?**
A: Cron runs jobs at specific times; anacron runs jobs periodically regardless of system uptime.

**Q2: How do you prevent a cron job from running if the previous instance is still running?**
A: Use file locking with `flock` or PID files.

**Q3: Where are cron logs stored?**
A: `/var/log/syslog` (Ubuntu/Debian) or `/var/log/cron` (CentOS/RHEL)

**Q4: What does `*/5 * * * *` mean?**
A: Run every 5 minutes.

**Q5: How do you run a cron job only on weekdays?**
A: Use `* * * * 1-5` for the day of week field.

---

## 🔗 Quick Links

- **Crontab Generator:** https://crontab.guru/
- **Man Page:** `man 5 crontab`
- **Logs:** `/var/log/syslog` or `/var/log/cron`

---

## ✅ Pre-Exam Checklist

Before moving to the next day, ensure you can:

- [ ] Explain cron syntax
- [ ] Create and edit crontab entries
- [ ] Use special time strings (@daily, @hourly, etc.)
- [ ] Redirect cron job output to log files
- [ ] Troubleshoot non-running cron jobs
- [ ] Check cron logs
- [ ] Use absolute paths in cron jobs
- [ ] Set environment variables in crontab
- [ ] Prevent overlapping job executions
- [ ] Monitor cron job execution

---

## 🎓 Key Takeaways

1. **Cron automates repetitive tasks** using time-based scheduling
2. **Syntax is critical** - minute, hour, day, month, weekday
3. **Always use absolute paths** in cron jobs
4. **Logging is essential** for troubleshooting
5. **Test manually first** before scheduling

---

## 📝 Quick Command Reference Card

```bash
# Crontab Management
crontab -e              # Edit
crontab -l              # List
crontab -r              # Remove
crontab -u user -e      # Edit for user

# Service Management
systemctl status cron   # Check status
systemctl start cron    # Start service
systemctl restart cron  # Restart service

# Debugging
grep CRON /var/log/syslog    # View logs
tail -f /var/log/syslog      # Follow logs
mail                         # Check mail

# Testing
/bin/bash -x /path/to/script.sh    # Debug mode
env -i /bin/bash /path/to/script.sh # Test environment
```

---

**Revision Time:** 15-20 minutes
**Last Updated:** 2024-01-15
**Next Topic:** Day 7 - [Next Topic]

---

## 💪 Challenge Yourself

Can you create a cron job that:
1. Runs every weekday at 9 AM
2. Backs up a directory
3. Logs the output
4. Sends an email on failure
5. Cleans up backups older than 30 days

**Answer:**
```bash
0 9 * * 1-5 /home/user/backup.sh >> /var/log/backup.log 2>&1 || echo "Backup failed" | mail -s "Backup Alert" admin@example.com
```

---

**Status:** ✅ Ready for Review
**Difficulty:** ⭐⭐ Intermediate