# Day 6: Create a Cron Job - Solutions

## 📝 Task Solutions

### Solution 1: Create a Simple Cron Job

**Objective:** Create a cron job that runs every minute and logs the current date and time.

**Step-by-Step Solution:**

```bash
# Step 1: Create a simple script
cat > ~/test-cron.sh << 'EOF'
#!/bin/bash
echo "$(date): Cron job executed successfully" >> ~/cron-test.log
EOF

# Step 2: Make the script executable
chmod +x ~/test-cron.sh

# Step 3: Test the script manually
~/test-cron.sh

# Step 4: Verify the log file was created
cat ~/cron-test.log

# Step 5: Open crontab editor
crontab -e

# Step 6: Add the following line to run every minute
* * * * * /home/$(whoami)/test-cron.sh

# Step 7: Save and exit the editor
# For vi/vim: Press ESC, then type :wq and press Enter
# For nano: Press Ctrl+X, then Y, then Enter

# Step 8: Verify the crontab entry
crontab -l

# Step 9: Wait a few minutes and check the log
tail -f ~/cron-test.log

# Step 10: Remove the test cron job when done
crontab -e
# Delete the line or comment it out with #
```

**Expected Output:**
```
# After a few minutes, ~/cron-test.log should contain:
Fri Jan 15 10:01:00 IST 2024: Cron job executed successfully
Fri Jan 15 10:02:00 IST 2024: Cron job executed successfully
Fri Jan 15 10:03:00 IST 2024: Cron job executed successfully
```

---

### Solution 2: Create a Backup Cron Job

**Objective:** Create a cron job that backs up a directory daily at 2 AM.

**Step-by-Step Solution:**

```bash
# Step 1: Create the backup script
cat > ~/backup-script.sh << 'EOF'
#!/bin/bash

# Configuration
SOURCE_DIR="/home/$(whoami)/important-data"
BACKUP_DIR="/home/$(whoami)/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"
LOG_FILE="/home/$(whoami)/backup.log"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Log start
echo "$(date): Starting backup..." >> "$LOG_FILE"

# Create backup
if tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR" 2>> "$LOG_FILE"; then
    echo "$(date): Backup completed successfully: ${BACKUP_FILE}" >> "$LOG_FILE"
    
    # Remove backups older than 7 days
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
    echo "$(date): Old backups cleaned up" >> "$LOG_FILE"
else
    echo "$(date): Backup failed!" >> "$LOG_FILE"
    exit 1
fi
EOF

# Step 2: Make the script executable
chmod +x ~/backup-script.sh

# Step 3: Create test directory and files
mkdir -p ~/important-data
echo "Important file 1" > ~/important-data/file1.txt
echo "Important file 2" > ~/important-data/file2.txt

# Step 4: Test the backup script
~/backup-script.sh

# Step 5: Verify backup was created
ls -lh ~/backups/
cat ~/backup.log

# Step 6: Add to crontab to run daily at 2 AM
crontab -e

# Add this line:
0 2 * * * /home/$(whoami)/backup-script.sh

# Step 7: Verify crontab entry
crontab -l
```

**Crontab Entry:**
```
# Daily backup at 2 AM
0 2 * * * /home/username/backup-script.sh
```

---

### Solution 3: Create Multiple Cron Jobs

**Objective:** Set up various cron jobs for different schedules.

**Step-by-Step Solution:**

```bash
# Step 1: Create multiple scripts

# Script 1: System health check (every 5 minutes)
cat > ~/health-check.sh << 'EOF'
#!/bin/bash
LOG="/home/$(whoami)/health-check.log"
echo "=== $(date) ===" >> "$LOG"
echo "CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}')" >> "$LOG"
echo "Memory: $(free -h | grep Mem | awk '{print $3 "/" $2}')" >> "$LOG"
echo "Disk: $(df -h / | tail -1 | awk '{print $5}')" >> "$LOG"
echo "" >> "$LOG"
EOF

# Script 2: Log rotation (daily at midnight)
cat > ~/log-rotate.sh << 'EOF'
#!/bin/bash
LOG_DIR="/home/$(whoami)/logs"
mkdir -p "$LOG_DIR"
DATE=$(date +%Y%m%d)

# Rotate logs
for log in ~/cron-test.log ~/backup.log ~/health-check.log; do
    if [ -f "$log" ]; then
        cp "$log" "${LOG_DIR}/$(basename $log).${DATE}"
        > "$log"  # Clear the original log
    fi
done
EOF

# Script 3: Weekly report (every Monday at 9 AM)
cat > ~/weekly-report.sh << 'EOF'
#!/bin/bash
REPORT="/home/$(whoami)/weekly-report.txt"
echo "Weekly Report - $(date)" > "$REPORT"
echo "========================" >> "$REPORT"
echo "" >> "$REPORT"
echo "Disk Usage:" >> "$REPORT"
df -h >> "$REPORT"
echo "" >> "$REPORT"
echo "Top Processes:" >> "$REPORT"
ps aux --sort=-%mem | head -10 >> "$REPORT"
EOF

# Step 2: Make all scripts executable
chmod +x ~/health-check.sh ~/log-rotate.sh ~/weekly-report.sh

# Step 3: Test each script
~/health-check.sh
~/log-rotate.sh
~/weekly-report.sh

# Step 4: Edit crontab
crontab -e

# Step 5: Add all cron jobs
# Add these lines:
```

**Complete Crontab Configuration:**
```bash
# Environment variables
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=your-email@example.com

# System health check - every 5 minutes
*/5 * * * * /home/username/health-check.sh

# Daily backup - 2 AM every day
0 2 * * * /home/username/backup-script.sh

# Log rotation - midnight every day
0 0 * * * /home/username/log-rotate.sh

# Weekly report - 9 AM every Monday
0 9 * * 1 /home/username/weekly-report.sh

# Cleanup temp files - 3 AM every Sunday
0 3 * * 0 find /tmp -type f -mtime +7 -delete

# System update check - 6 AM on 1st of every month
0 6 1 * * /usr/bin/apt update && /usr/bin/apt list --upgradable > /home/username/updates.txt
```

---

### Solution 4: Advanced Cron Job with Error Handling

**Objective:** Create a robust cron job with proper error handling and notifications.

**Step-by-Step Solution:**

```bash
# Step 1: Create advanced backup script with error handling
cat > ~/advanced-backup.sh << 'EOF'
#!/bin/bash

# Configuration
SOURCE_DIR="/home/$(whoami)/important-data"
BACKUP_DIR="/home/$(whoami)/backups"
LOG_FILE="/home/$(whoami)/backup-detailed.log"
ERROR_LOG="/home/$(whoami)/backup-errors.log"
MAX_BACKUPS=7
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to log errors
log_error() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] ERROR: $1" | tee -a "$ERROR_LOG" >&2
}

# Function to send notification (requires mail command)
send_notification() {
    local subject="$1"
    local message="$2"
    
    if command -v mail &> /dev/null; then
        echo "$message" | mail -s "$subject" root
    fi
}

# Check if source directory exists
if [ ! -d "$SOURCE_DIR" ]; then
    log_error "Source directory does not exist: $SOURCE_DIR"
    send_notification "Backup Failed" "Source directory not found"
    exit 1
fi

# Create backup directory
if ! mkdir -p "$BACKUP_DIR"; then
    log_error "Failed to create backup directory: $BACKUP_DIR"
    exit 1
fi

# Check available disk space (need at least 1GB)
AVAILABLE_SPACE=$(df "$BACKUP_DIR" | tail -1 | awk '{print $4}')
if [ "$AVAILABLE_SPACE" -lt 1048576 ]; then
    log_error "Insufficient disk space for backup"
    send_notification "Backup Failed" "Insufficient disk space"
    exit 1
fi

# Start backup
log_message "Starting backup of $SOURCE_DIR"

# Create backup with progress
if tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR" 2>> "$ERROR_LOG"; then
    BACKUP_SIZE=$(du -h "${BACKUP_DIR}/${BACKUP_FILE}" | cut -f1)
    log_message "Backup completed successfully: ${BACKUP_FILE} (Size: ${BACKUP_SIZE})"
    
    # Verify backup integrity
    if tar -tzf "${BACKUP_DIR}/${BACKUP_FILE}" > /dev/null 2>&1; then
        log_message "Backup integrity verified"
    else
        log_error "Backup integrity check failed"
        send_notification "Backup Warning" "Backup created but integrity check failed"
    fi
    
    # Remove old backups
    log_message "Cleaning up old backups (keeping last $MAX_BACKUPS)"
    OLD_BACKUPS=$(ls -t "${BACKUP_DIR}"/backup_*.tar.gz 2>/dev/null | tail -n +$((MAX_BACKUPS + 1)))
    
    if [ -n "$OLD_BACKUPS" ]; then
        echo "$OLD_BACKUPS" | while read -r old_backup; do
            rm -f "$old_backup"
            log_message "Removed old backup: $(basename "$old_backup")"
        done
    fi
    
    # Calculate total backup size
    TOTAL_SIZE=$(du -sh "$BACKUP_DIR" | cut -f1)
    log_message "Total backup size: $TOTAL_SIZE"
    
    # Success notification
    send_notification "Backup Successful" "Backup completed: ${BACKUP_FILE}"
    
else
    log_error "Backup failed for $SOURCE_DIR"
    send_notification "Backup Failed" "Check error log for details"
    exit 1
fi

log_message "Backup process completed"
exit 0
EOF

# Step 2: Make script executable
chmod +x ~/advanced-backup.sh

# Step 3: Create a wrapper script for cron
cat > ~/cron-wrapper.sh << 'EOF'
#!/bin/bash

# Wrapper script for cron jobs
# Provides consistent environment and error handling

# Set up environment
export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
export HOME=/home/$(whoami)

# Lock file to prevent concurrent execution
LOCK_FILE="/tmp/$(basename $1).lock"

# Function to cleanup on exit
cleanup() {
    rm -f "$LOCK_FILE"
}

trap cleanup EXIT

# Check if already running
if [ -f "$LOCK_FILE" ]; then
    echo "Script already running. Exiting."
    exit 1
fi

# Create lock file
touch "$LOCK_FILE"

# Execute the actual script
"$@"
EXIT_CODE=$?

# Cleanup and exit
exit $EXIT_CODE
EOF

chmod +x ~/cron-wrapper.sh

# Step 4: Test the scripts
~/advanced-backup.sh
cat ~/backup-detailed.log

# Step 5: Add to crontab with wrapper
crontab -e

# Add this line:
0 2 * * * /home/$(whoami)/cron-wrapper.sh /home/$(whoami)/advanced-backup.sh
```

---

### Solution 5: Monitoring Cron Jobs

**Objective:** Create a monitoring system for cron jobs.

**Step-by-Step Solution:**

```bash
# Step 1: Create cron job monitor script
cat > ~/cron-monitor.sh << 'EOF'
#!/bin/bash

REPORT_FILE="/home/$(whoami)/cron-monitor-report.txt"
CRON_LOG="/var/log/syslog"  # or /var/log/cron on some systems

echo "Cron Job Monitoring Report" > "$REPORT_FILE"
echo "Generated: $(date)" >> "$REPORT_FILE"
echo "================================" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# List all cron jobs
echo "Current Cron Jobs:" >> "$REPORT_FILE"
echo "-----------------" >> "$REPORT_FILE"
crontab -l >> "$REPORT_FILE" 2>&1
echo "" >> "$REPORT_FILE"

# Check recent cron executions
echo "Recent Cron Executions (last 24 hours):" >> "$REPORT_FILE"
echo "---------------------------------------" >> "$REPORT_FILE"
if [ -f "$CRON_LOG" ]; then
    grep CRON "$CRON_LOG" | grep "$(date +%b\ %d)" >> "$REPORT_FILE"
else
    echo "Cron log not accessible" >> "$REPORT_FILE"
fi
echo "" >> "$REPORT_FILE"

# Check for errors
echo "Recent Cron Errors:" >> "$REPORT_FILE"
echo "------------------" >> "$REPORT_FILE"
if [ -f "$CRON_LOG" ]; then
    grep -i "error\|fail" "$CRON_LOG" | grep CRON | tail -20 >> "$REPORT_FILE"
else
    echo "No errors found or log not accessible" >> "$REPORT_FILE"
fi

# Display report
cat "$REPORT_FILE"
EOF

chmod +x ~/cron-monitor.sh

# Step 2: Run the monitor
~/cron-monitor.sh

# Step 3: Add to crontab to run daily
crontab -e

# Add:
0 8 * * * /home/$(whoami)/cron-monitor.sh
```

---

## 🔍 Verification Steps

### Verify Cron Service is Running
```bash
# Check cron service status
sudo systemctl status cron
# or
sudo systemctl status crond

# Start cron if not running
sudo systemctl start cron

# Enable cron to start on boot
sudo systemctl enable cron
```

### Check Cron Logs
```bash
# View cron logs (Ubuntu/Debian)
grep CRON /var/log/syslog | tail -20

# View cron logs (CentOS/RHEL)
tail -20 /var/log/cron

# Follow cron logs in real-time
tail -f /var/log/syslog | grep CRON
```

### Test Cron Job Manually
```bash
# Run the command exactly as it appears in crontab
/bin/bash -c "/home/username/script.sh"

# Check exit code
echo $?

# Test with cron's environment
env -i /bin/bash --noprofile --norc -c "/home/username/script.sh"
```

---

## 🐛 Troubleshooting Common Issues

### Issue 1: Cron Job Not Running

**Problem:** Cron job doesn't execute at scheduled time.

**Solutions:**
```bash
# 1. Check cron service
sudo systemctl status cron

# 2. Verify crontab syntax
crontab -l

# 3. Check cron logs
grep CRON /var/log/syslog

# 4. Verify script permissions
ls -l /path/to/script.sh
chmod +x /path/to/script.sh

# 5. Test script manually
/path/to/script.sh
```

### Issue 2: Script Works Manually But Not in Cron

**Problem:** Script runs fine when executed manually but fails in cron.

**Solutions:**
```bash
# 1. Use absolute paths
# Instead of: cd data && ./process.sh
# Use: cd /home/user/data && /home/user/data/process.sh

# 2. Set environment variables in crontab
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
HOME=/home/username

# 3. Source environment in script
#!/bin/bash
source /home/username/.bashrc
# rest of script

# 4. Add debugging
#!/bin/bash
set -x  # Enable debug mode
# rest of script
```

### Issue 3: No Output or Logs

**Problem:** Can't see cron job output or errors.

**Solutions:**
```bash
# 1. Redirect output to log file
* * * * * /path/to/script.sh >> /var/log/myscript.log 2>&1

# 2. Check mail
mail

# 3. Set MAILTO in crontab
MAILTO=your-email@example.com

# 4. Add logging to script
#!/bin/bash
LOG_FILE="/var/log/myscript.log"
echo "$(date): Script started" >> "$LOG_FILE"
# rest of script
echo "$(date): Script completed" >> "$LOG_FILE"
```

---

## ✅ Success Criteria

- [ ] Cron service is running
- [ ] Crontab entries are correctly formatted
- [ ] Scripts are executable and use absolute paths
- [ ] Cron jobs execute at scheduled times
- [ ] Output is properly logged
- [ ] No errors in cron logs
- [ ] Old logs/backups are cleaned up automatically
- [ ] Monitoring is in place

---

## 📚 Additional Examples

### Database Backup Cron Job
```bash
# Daily MySQL backup at 3 AM
0 3 * * * /usr/bin/mysqldump -u root -p'password' database_name > /backup/db_$(date +\%Y\%m\%d).sql
```

### Website Monitoring
```bash
# Check website every 5 minutes
*/5 * * * * curl -f https://example.com > /dev/null 2>&1 || echo "Website down at $(date)" >> /var/log/website-monitor.log
```

### Certificate Renewal
```bash
# Renew Let's Encrypt certificate monthly
0 3 1 * * /usr/bin/certbot renew --quiet
```

### System Cleanup
```bash
# Clean temp files weekly
0 2 * * 0 find /tmp -type f -atime +7 -delete
```

---

**Completion Date:** 2024-01-15
**Status:** ✅ All solutions tested and verified