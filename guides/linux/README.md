# Linux Fundamentals for DevOps

## 🎯 Overview

Linux is the foundation of DevOps. Almost all servers, containers, and cloud instances run Linux. Mastering Linux is non-negotiable for a successful DevOps career.

**Duration:** 3 weeks (Weeks 1-3)
**Time Investment:** 3-4 hours/day
**Difficulty:** Beginner to Intermediate

---

## 📚 What You'll Learn

### Week 1: Linux Basics
- File system hierarchy and navigation
- Basic commands and utilities
- File and directory operations
- Text editors (vim, nano)
- Help systems (man pages, --help)

### Week 2: System Administration
- User and group management
- File permissions and ownership
- Process management
- Package management
- System services (systemd)

### Week 3: Advanced Topics
- Shell environment and variables
- Text processing (grep, sed, awk)
- Networking commands
- Log management
- Cron jobs and automation

---

## 🎓 Important Concepts

### 1. Linux File System Hierarchy

```
/                    Root directory
├── /bin            Essential user binaries
├── /boot           Boot loader files
├── /dev            Device files
├── /etc            System configuration files
├── /home           User home directories
├── /lib            System libraries
├── /media          Removable media mount points
├── /mnt            Temporary mount points
├── /opt            Optional software
├── /proc           Process information (virtual)
├── /root           Root user home directory
├── /run            Runtime data
├── /sbin           System binaries
├── /srv            Service data
├── /sys            System information (virtual)
├── /tmp            Temporary files
├── /usr            User programs and data
│   ├── /usr/bin    User binaries
│   ├── /usr/lib    User libraries
│   ├── /usr/local  Locally installed software
│   └── /usr/share  Shared data
└── /var            Variable data (logs, databases)
    ├── /var/log    Log files
    ├── /var/www    Web server files
    └── /var/tmp    Temporary files (persistent)
```

**Key Concepts:**
- Everything is a file in Linux
- Case-sensitive file system
- Hidden files start with `.`
- No drive letters (C:, D:) - everything under `/`

---

### 2. File Permissions

```
-rwxr-xr--  1 user group 4096 Jan 15 10:30 file.txt
│││││││││
│││││││└└─ Other permissions (r--)
││││││└──── Group permissions (r-x)
│││││└───── Owner permissions (rwx)
││││└────── Number of hard links
│││└─────── Owner
││└──────── Group
│└───────── File size
└────────── File type (- = regular file, d = directory, l = link)
```

**Permission Types:**
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1

**Examples:**
```bash
chmod 755 file.txt    # rwxr-xr-x
chmod 644 file.txt    # rw-r--r--
chmod 600 file.txt    # rw-------
chmod +x script.sh    # Add execute permission
chmod u+x script.sh   # Add execute for user only
chmod g-w file.txt    # Remove write for group
```

---

### 3. Process Management

**Process States:**
- Running (R)
- Sleeping (S)
- Stopped (T)
- Zombie (Z)

**Key Concepts:**
- PID: Process ID
- PPID: Parent Process ID
- UID: User ID
- Priority: Nice value (-20 to 19)

**Process Hierarchy:**
```
systemd (PID 1)
├── sshd
│   └── bash
│       └── vim
├── nginx
│   ├── worker process
│   └── worker process
└── cron
    └── backup script
```

---

### 4. Package Management

**Debian/Ubuntu (APT):**
```bash
apt update              # Update package list
apt upgrade             # Upgrade packages
apt install package     # Install package
apt remove package      # Remove package
apt search package      # Search for package
apt show package        # Show package info
```

**RHEL/CentOS (YUM/DNF):**
```bash
yum update              # Update packages
yum install package     # Install package
yum remove package      # Remove package
yum search package      # Search for package
yum info package        # Show package info
```

---

### 5. Systemd Services

**Service Management:**
```bash
systemctl start service     # Start service
systemctl stop service      # Stop service
systemctl restart service   # Restart service
systemctl reload service    # Reload configuration
systemctl status service    # Check status
systemctl enable service    # Enable at boot
systemctl disable service   # Disable at boot
```

**Service States:**
- active (running)
- inactive (dead)
- failed
- activating (starting)
- deactivating (stopping)

---

### 6. Text Processing

**grep (Search):**
```bash
grep "pattern" file           # Search for pattern
grep -i "pattern" file        # Case-insensitive
grep -r "pattern" directory   # Recursive search
grep -v "pattern" file        # Invert match
grep -n "pattern" file        # Show line numbers
grep -c "pattern" file        # Count matches
```

**sed (Stream Editor):**
```bash
sed 's/old/new/' file         # Replace first occurrence
sed 's/old/new/g' file        # Replace all occurrences
sed -i 's/old/new/g' file     # Edit file in-place
sed '1,10d' file              # Delete lines 1-10
sed -n '5,10p' file           # Print lines 5-10
```

**awk (Text Processing):**
```bash
awk '{print $1}' file         # Print first column
awk '{print $1,$3}' file      # Print columns 1 and 3
awk '/pattern/ {print}' file  # Print matching lines
awk 'NR==5' file              # Print line 5
awk '{sum+=$1} END {print sum}' file  # Sum first column
```

---

### 7. Networking Basics

**Key Concepts:**
- IP Address: Unique identifier for network interface
- Subnet Mask: Defines network range
- Gateway: Router to other networks
- DNS: Translates domain names to IP addresses
- Port: Endpoint for network communication

**Common Ports:**
- 22: SSH
- 80: HTTP
- 443: HTTPS
- 3306: MySQL
- 5432: PostgreSQL
- 6379: Redis
- 27017: MongoDB

---

## 💻 Hands-On Exercises

### Exercise 1: File System Navigation (30 minutes)

**Objective:** Master basic navigation and file operations

```bash
# 1. Navigate to home directory
cd ~

# 2. Create directory structure
mkdir -p projects/devops/{scripts,configs,logs}

# 3. Navigate to scripts directory
cd projects/devops/scripts

# 4. Create files
touch script1.sh script2.sh script3.sh

# 5. List files with details
ls -lah

# 6. Create a file with content
echo "#!/bin/bash" > hello.sh
echo "echo 'Hello, DevOps!'" >> hello.sh

# 7. View file content
cat hello.sh

# 8. Copy file
cp hello.sh hello_backup.sh

# 9. Move file
mv hello_backup.sh ../configs/

# 10. Remove file
rm script3.sh

# 11. Find files
find ~/projects -name "*.sh"

# 12. Search in files
grep -r "Hello" ~/projects
```

**Expected Output:**
```
projects/
└── devops/
    ├── scripts/
    │   ├── script1.sh
    │   ├── script2.sh
    │   └── hello.sh
    ├── configs/
    │   └── hello_backup.sh
    └── logs/
```

---

### Exercise 2: User and Permission Management (45 minutes)

**Objective:** Understand users, groups, and permissions

```bash
# 1. Create a new user (requires sudo)
sudo useradd -m -s /bin/bash devuser

# 2. Set password
sudo passwd devuser

# 3. Create a group
sudo groupadd developers

# 4. Add user to group
sudo usermod -aG developers devuser

# 5. View user info
id devuser
groups devuser

# 6. Create a shared directory
sudo mkdir /opt/shared
sudo chown :developers /opt/shared
sudo chmod 770 /opt/shared

# 7. Test permissions
sudo -u devuser touch /opt/shared/test.txt
ls -l /opt/shared/

# 8. Change file ownership
sudo chown devuser:developers /opt/shared/test.txt

# 9. Set special permissions
sudo chmod g+s /opt/shared  # Set GID

# 10. View effective permissions
ls -ld /opt/shared
```

**Challenge:**
Create a directory where:
- Owner can read, write, execute
- Group can read and execute
- Others have no permissions
- All new files inherit group ownership

**Solution:**
```bash
mkdir secure_dir
chmod 750 secure_dir
chmod g+s secure_dir
```

---

### Exercise 3: Process Management (30 minutes)

**Objective:** Monitor and manage processes

```bash
# 1. View all processes
ps aux

# 2. View process tree
pstree

# 3. View processes for current user
ps -u $USER

# 4. Find specific process
ps aux | grep nginx

# 5. Real-time process monitoring
top
# Press 'q' to quit

# 6. Better process viewer
htop  # Install if not available: sudo apt install htop

# 7. Start a background process
sleep 100 &

# 8. List background jobs
jobs

# 9. Bring job to foreground
fg %1

# 10. Send to background (Ctrl+Z, then:)
bg %1

# 11. Kill a process
kill PID
kill -9 PID  # Force kill

# 12. Kill by name
pkill sleep
killall sleep

# 13. View system resource usage
free -h        # Memory
df -h          # Disk space
du -sh *       # Directory sizes
uptime         # System uptime and load
```

**Challenge:**
Write a script that:
1. Finds all processes using more than 50% CPU
2. Logs them to a file
3. Sends an alert (echo to console)

**Solution:**
```bash
#!/bin/bash
ps aux | awk '$3 > 50 {print $0}' > high_cpu.log
if [ -s high_cpu.log ]; then
    echo "ALERT: High CPU processes detected!"
    cat high_cpu.log
fi
```

---

### Exercise 4: Package Management (30 minutes)

**Objective:** Install and manage software packages

```bash
# For Ubuntu/Debian:

# 1. Update package list
sudo apt update

# 2. Search for package
apt search nginx

# 3. Show package info
apt show nginx

# 4. Install package
sudo apt install nginx -y

# 5. Check if installed
which nginx
nginx -v

# 6. List installed packages
apt list --installed | grep nginx

# 7. Remove package
sudo apt remove nginx

# 8. Remove package and config files
sudo apt purge nginx

# 9. Remove unused dependencies
sudo apt autoremove

# 10. Install multiple packages
sudo apt install vim git curl wget -y
```

**Challenge:**
Create a script that installs a LAMP stack (Linux, Apache, MySQL, PHP)

**Solution:**
```bash
#!/bin/bash
echo "Installing LAMP stack..."
sudo apt update
sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql
sudo systemctl start apache2
sudo systemctl enable apache2
echo "LAMP stack installed successfully!"
echo "Apache status:"
sudo systemctl status apache2 --no-pager
```

---

### Exercise 5: Systemd Services (45 minutes)

**Objective:** Manage system services

```bash
# 1. Check service status
sudo systemctl status sshd

# 2. Start service
sudo systemctl start nginx

# 3. Stop service
sudo systemctl stop nginx

# 4. Restart service
sudo systemctl restart nginx

# 5. Reload configuration
sudo systemctl reload nginx

# 6. Enable service at boot
sudo systemctl enable nginx

# 7. Disable service at boot
sudo systemctl disable nginx

# 8. View service logs
sudo journalctl -u nginx

# 9. Follow logs in real-time
sudo journalctl -u nginx -f

# 10. View failed services
systemctl --failed

# 11. List all services
systemctl list-units --type=service

# 12. View service dependencies
systemctl list-dependencies nginx
```

**Challenge:**
Create a custom systemd service for a simple Python web server

**Solution:**
```bash
# 1. Create Python script
cat > /opt/webapp.py << 'EOF'
#!/usr/bin/env python3
from http.server import HTTPServer, SimpleHTTPRequestHandler
HTTPServer(('', 8080), SimpleHTTPRequestHandler).serve_forever()
EOF

chmod +x /opt/webapp.py

# 2. Create systemd service file
sudo tee /etc/systemd/system/webapp.service << 'EOF'
[Unit]
Description=Simple Web Application
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt
ExecStart=/usr/bin/python3 /opt/webapp.py
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# 3. Reload systemd
sudo systemctl daemon-reload

# 4. Start and enable service
sudo systemctl start webapp
sudo systemctl enable webapp

# 5. Check status
sudo systemctl status webapp

# 6. Test
curl http://localhost:8080
```

---

### Exercise 6: Text Processing (60 minutes)

**Objective:** Master grep, sed, and awk

**Setup:**
```bash
# Create sample log file
cat > /tmp/access.log << 'EOF'
192.168.1.100 - - [15/Jan/2026:10:30:45] "GET /index.html HTTP/1.1" 200 1234
192.168.1.101 - - [15/Jan/2026:10:31:12] "GET /about.html HTTP/1.1" 200 5678
192.168.1.100 - - [15/Jan/2026:10:32:33] "POST /api/login HTTP/1.1" 401 89
192.168.1.102 - - [15/Jan/2026:10:33:21] "GET /products.html HTTP/1.1" 200 9012
192.168.1.101 - - [15/Jan/2026:10:34:56] "GET /contact.html HTTP/1.1" 404 234
192.168.1.103 - - [15/Jan/2026:10:35:44] "GET /index.html HTTP/1.1" 200 1234
192.168.1.100 - - [15/Jan/2026:10:36:12] "POST /api/data HTTP/1.1" 500 567
EOF
```

**Tasks:**

```bash
# 1. Find all 404 errors
grep "404" /tmp/access.log

# 2. Count 200 responses
grep -c "200" /tmp/access.log

# 3. Find all POST requests
grep "POST" /tmp/access.log

# 4. Extract IP addresses
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' /tmp/access.log

# 5. Replace HTTP with HTTPS
sed 's/HTTP/HTTPS/g' /tmp/access.log

# 6. Extract status codes
awk '{print $9}' /tmp/access.log

# 7. Count requests per IP
awk '{print $1}' /tmp/access.log | sort | uniq -c

# 8. Calculate total bytes transferred
awk '{sum+=$10} END {print sum}' /tmp/access.log

# 9. Find errors (4xx and 5xx)
awk '$9 >= 400 {print}' /tmp/access.log

# 10. Extract URLs
awk '{print $7}' /tmp/access.log | sort | uniq
```

**Challenge:**
Create a log analysis script that:
1. Counts requests per status code
2. Lists top 5 IP addresses
3. Calculates average response size
4. Finds peak hour

**Solution:**
```bash
#!/bin/bash
LOG_FILE="/tmp/access.log"

echo "=== Log Analysis Report ==="
echo

echo "1. Requests per Status Code:"
awk '{print $9}' $LOG_FILE | sort | uniq -c | sort -rn

echo
echo "2. Top 5 IP Addresses:"
awk '{print $1}' $LOG_FILE | sort | uniq -c | sort -rn | head -5

echo
echo "3. Average Response Size:"
awk '{sum+=$10; count++} END {print sum/count " bytes"}' $LOG_FILE

echo
echo "4. Requests per Hour:"
awk -F'[: ]' '{print $5}' $LOG_FILE | sort | uniq -c
```

---

### Exercise 7: Networking Commands (45 minutes)

**Objective:** Understand network troubleshooting

```bash
# 1. View network interfaces
ip addr show
ifconfig  # Older command

# 2. View routing table
ip route show
route -n  # Older command

# 3. Test connectivity
ping -c 4 google.com

# 4. Trace route
traceroute google.com
tracepath google.com

# 5. DNS lookup
nslookup google.com
dig google.com
host google.com

# 6. View open ports
sudo netstat -tulpn
sudo ss -tulpn  # Modern alternative

# 7. Test port connectivity
telnet google.com 80
nc -zv google.com 80  # netcat

# 8. Download file
wget https://example.com/file.txt
curl -O https://example.com/file.txt

# 9. View network statistics
netstat -s
ss -s

# 10. Monitor network traffic
sudo tcpdump -i eth0
sudo iftop  # Install: sudo apt install iftop
```

**Challenge:**
Create a network diagnostic script that:
1. Checks internet connectivity
2. Tests DNS resolution
3. Checks if specific ports are open
4. Reports network interface status

**Solution:**
```bash
#!/bin/bash

echo "=== Network Diagnostic Report ==="
echo

# Check internet connectivity
echo "1. Internet Connectivity:"
if ping -c 1 8.8.8.8 &> /dev/null; then
    echo "✓ Internet is reachable"
else
    echo "✗ No internet connection"
fi

# Test DNS
echo
echo "2. DNS Resolution:"
if nslookup google.com &> /dev/null; then
    echo "✓ DNS is working"
else
    echo "✗ DNS resolution failed"
fi

# Check ports
echo
echo "3. Port Status:"
for port in 22 80 443; do
    if nc -zv localhost $port 2>&1 | grep -q succeeded; then
        echo "✓ Port $port is open"
    else
        echo "✗ Port $port is closed"
    fi
done

# Network interfaces
echo
echo "4. Network Interfaces:"
ip -br addr show
```

---

### Exercise 8: Cron Jobs (30 minutes)

**Objective:** Schedule automated tasks

```bash
# 1. View current cron jobs
crontab -l

# 2. Edit cron jobs
crontab -e

# 3. Cron syntax:
# * * * * * command
# │ │ │ │ │
# │ │ │ │ └─── Day of week (0-7, 0 and 7 are Sunday)
# │ │ │ └───── Month (1-12)
# │ │ └─────── Day of month (1-31)
# │ └───────── Hour (0-23)
# └─────────── Minute (0-59)

# Examples:
# Run every minute
* * * * * /path/to/script.sh

# Run every hour
0 * * * * /path/to/script.sh

# Run every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Run every Monday at 9 AM
0 9 * * 1 /path/to/script.sh

# Run every 15 minutes
*/15 * * * * /path/to/script.sh

# 4. View system cron jobs
ls -la /etc/cron.daily/
ls -la /etc/cron.weekly/
ls -la /etc/cron.monthly/

# 5. View cron logs
sudo grep CRON /var/log/syslog
```

**Challenge:**
Create cron jobs for:
1. Daily backup at 2 AM
2. Log rotation every Sunday at midnight
3. System health check every 5 minutes
4. Weekly report every Friday at 5 PM

**Solution:**
```bash
# Edit crontab
crontab -e

# Add these lines:
# Daily backup at 2 AM
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# Log rotation every Sunday at midnight
0 0 * * 0 /opt/scripts/rotate_logs.sh

# System health check every 5 minutes
*/5 * * * * /opt/scripts/health_check.sh

# Weekly report every Friday at 5 PM
0 17 * * 5 /opt/scripts/weekly_report.sh
```

---

## 🎯 Practice Projects

### Project 1: System Monitoring Script
Create a script that monitors system resources and sends alerts.

**Requirements:**
- Check CPU usage
- Check memory usage
- Check disk space
- Check running services
- Log results
- Send email if thresholds exceeded

### Project 2: Automated Backup System
Create a backup system for important directories.

**Requirements:**
- Backup specified directories
- Compress backups
- Rotate old backups (keep last 7 days)
- Log backup status
- Schedule with cron

### Project 3: Log Analyzer
Create a tool to analyze web server logs.

**Requirements:**
- Parse Apache/Nginx logs
- Count requests per IP
- Identify most accessed pages
- Find error rates
- Generate HTML report

### Project 4: User Management Tool
Create a script to manage users in bulk.

**Requirements:**
- Create multiple users from CSV
- Set passwords
- Add to groups
- Create home directories
- Set quotas

---

## 📝 Cheat Sheet

See [`cheat-sheet.md`](cheat-sheet.md) for quick command reference.

---

## 📚 Additional Resources

- [Linux Journey](https://linuxjourney.com/) - Interactive learning
- [The Linux Command Line](http://linuxcommand.org/tlcl.php) - Free book
- [Linux Academy](https://acloudguru.com/) - Video courses
- [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) - Practice challenges

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand file system hierarchy
- [ ] Navigate using cd, ls, pwd
- [ ] Create, copy, move, delete files
- [ ] Use vim or nano editor
- [ ] Read man pages

### Week 2
- [ ] Create and manage users
- [ ] Understand file permissions
- [ ] Manage processes
- [ ] Install packages
- [ ] Manage services with systemd

### Week 3
- [ ] Use grep, sed, awk
- [ ] Understand networking basics
- [ ] Analyze log files
- [ ] Schedule cron jobs
- [ ] Complete practice projects

---

**Next:** [Shell Scripting →](../shell-scripting/README.md)

**Last Updated:** June 19, 2026