# Day 6: Create a Cron Job

## 📋 Task Description
Learn how to schedule automated tasks using cron jobs in Linux. Cron is a time-based job scheduler that allows you to run scripts and commands at specified intervals.

## 🎯 Learning Objectives
- Understand cron daemon and crontab
- Learn cron syntax and scheduling patterns
- Create and manage cron jobs
- Handle cron job output and logging
- Troubleshoot common cron issues

---

## 📚 Comprehensive Notes

### What is Cron?

**Cron** is a time-based job scheduler in Unix-like operating systems. It enables users to schedule jobs (commands or scripts) to run automatically at specified times, dates, or intervals.

**Key Components:**
- **crond**: The cron daemon that runs in the background
- **crontab**: The cron table file containing scheduled jobs
- **cron.d**: Directory for system-wide cron jobs

### Cron Syntax

```
* * * * * command_to_execute
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Special Characters:**
- `*` : Any value (every)
- `,` : Value list separator (e.g., 1,3,5)
- `-` : Range of values (e.g., 1-5)
- `/` : Step values (e.g., */5 means every 5)

**Special Strings:**
- `@reboot` : Run once at startup
- `@yearly` or `@annually` : Run once a year (0 0 1 1 *)
- `@monthly` : Run once a month (0 0 1 * *)
- `@weekly` : Run once a week (0 0 * * 0)
- `@daily` or `@midnight` : Run once a day (0 0 * * *)
- `@hourly` : Run once an hour (0 * * * *)

### Crontab Commands

```bash
# View current user's crontab
crontab -l

# Edit current user's crontab
crontab -e

# Remove current user's crontab
crontab -r

# Edit another user's crontab (requires root)
sudo crontab -u username -e

# View another user's crontab
sudo crontab -u username -l
```

### Common Cron Job Examples

```bash
# Run every minute
* * * * * /path/to/script.sh

# Run every 5 minutes
*/5 * * * * /path/to/script.sh

# Run every hour at minute 30
30 * * * * /path/to/script.sh

# Run every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Run every Monday at 9:00 AM
0 9 * * 1 /path/to/script.sh

# Run on the 1st of every month at midnight
0 0 1 * * /path/to/script.sh

# Run every weekday at 6:00 PM
0 18 * * 1-5 /path/to/script.sh

# Run every 15 minutes during business hours
*/15 9-17 * * 1-5 /path/to/script.sh

# Run at specific times
0 9,12,18 * * * /path/to/script.sh

# Run at system startup
@reboot /path/to/script.sh
```

### Environment Variables in Cron

Cron jobs run with a minimal environment. You need to set variables explicitly:

```bash
# Set environment variables in crontab
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@example.com

# Use full paths in commands
0 2 * * * /usr/bin/python3 /home/user/script.py

# Source environment files
0 2 * * * . /home/user/.bashrc && /home/user/script.sh
```

### Cron Job Output and Logging

```bash
# Redirect output to a log file
0 2 * * * /path/to/script.sh >> /var/log/myscript.log 2>&1

# Discard all output
0 2 * * * /path/to/script.sh > /dev/null 2>&1

# Send output via email (if MAILTO is set)
0 2 * * * /path/to/script.sh

# Log with timestamp
0 2 * * * echo "$(date): Running backup" >> /var/log/backup.log && /path/to/backup.sh
```

### System-Wide Cron Jobs

**Location of system cron files:**
- `/etc/crontab` - System-wide crontab
- `/etc/cron.d/` - Directory for system cron jobs
- `/etc/cron.hourly/` - Scripts run hourly
- `/etc/cron.daily/` - Scripts run daily
- `/etc/cron.weekly/` - Scripts run weekly
- `/etc/cron.monthly/` - Scripts run monthly

**Example /etc/crontab format:**
```bash
# m h dom mon dow user  command
17 *  * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6  * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
```

### Cron Access Control

```bash
# Allow specific users to use cron
# /etc/cron.allow
user1
user2

# Deny specific users from using cron
# /etc/cron.deny
baduser1
baduser2

# If cron.allow exists, only listed users can use cron
# If only cron.deny exists, all users except listed can use cron
# If neither exists, only root can use cron (on some systems)
```

### Troubleshooting Cron Jobs

**Common Issues:**

1. **Script not executing:**
   - Check cron daemon: `systemctl status cron` or `systemctl status crond`
   - Verify crontab syntax: `crontab -l`
   - Check script permissions: `ls -l /path/to/script.sh`
   - Ensure script is executable: `chmod +x /path/to/script.sh`

2. **Environment issues:**
   - Use absolute paths for commands and files
   - Set PATH variable in crontab
   - Source necessary environment files

3. **No output/logs:**
   - Check mail: `mail` or `/var/mail/username`
   - Add logging to script
   - Check syslog: `grep CRON /var/log/syslog`

4. **Permission denied:**
   - Check file ownership and permissions
   - Verify user has access to required resources
   - Check SELinux/AppArmor policies

**Debugging Commands:**
```bash
# Check cron daemon status
systemctl status cron
systemctl status crond

# View cron logs
grep CRON /var/log/syslog
tail -f /var/log/cron

# Test cron job manually
/bin/bash -c "command_from_crontab"

# Check user's mail for cron output
mail

# Verify crontab syntax
crontab -l | grep -v "^#"
```

### Best Practices

1. **Use absolute paths** for all commands and files
2. **Add comments** to document what each job does
3. **Redirect output** to log files for debugging
4. **Test scripts manually** before adding to cron
5. **Use locking mechanisms** to prevent overlapping executions
6. **Set appropriate permissions** on scripts and log files
7. **Monitor cron job execution** and failures
8. **Use meaningful names** for scripts
9. **Keep cron jobs simple** - complex logic should be in scripts
10. **Document dependencies** and requirements

### Advanced Cron Techniques

**Prevent Overlapping Jobs:**
```bash
# Using flock
* * * * * flock -n /tmp/myjob.lock /path/to/script.sh

# Using PID file
* * * * * /path/to/script-with-pidfile.sh
```

**Run Job Only If Previous Succeeded:**
```bash
0 2 * * * /path/to/job1.sh && /path/to/job2.sh
```

**Conditional Execution:**
```bash
# Run only on first Monday of month
0 9 1-7 * 1 /path/to/script.sh

# Run only if file exists
0 2 * * * [ -f /tmp/trigger ] && /path/to/script.sh
```

**Random Delays:**
```bash
# Add random delay (0-300 seconds)
0 2 * * * sleep $((RANDOM \% 300)) && /path/to/script.sh
```

---

## 🔑 Key Takeaways

1. Cron is essential for automating repetitive tasks
2. Understanding cron syntax is crucial for proper scheduling
3. Always use absolute paths in cron jobs
4. Proper logging helps with troubleshooting
5. Test cron jobs manually before scheduling
6. Monitor cron job execution regularly
7. Use appropriate access controls
8. Handle environment variables explicitly

---

## 📖 Additional Resources

- Man pages: `man cron`, `man crontab`, `man 5 crontab`
- Crontab Generator: https://crontab.guru/
- Cron documentation: `/usr/share/doc/cron/`
- System logs: `/var/log/syslog` or `/var/log/cron`

---

## 🔗 Related Topics
- Systemd timers (modern alternative to cron)
- Anacron (for systems not running 24/7)
- At command (one-time scheduled tasks)
- Task scheduling in other operating systems

---

**Date:** 2024-01-15
**Status:** ✅ Completed
**Difficulty:** ⭐⭐ Intermediate