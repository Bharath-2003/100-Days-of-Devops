# Day 9: MariaDB Service Troubleshooting - Solutions

## 📋 Table of Contents
1. [Solution 1: Complete MariaDB Installation and Setup](#solution-1-complete-mariadb-installation-and-setup)
2. [Solution 2: Troubleshoot Service Won't Start](#solution-2-troubleshoot-service-wont-start)
3. [Solution 3: Fix Connection Issues](#solution-3-fix-connection-issues)
4. [Solution 4: Resolve High CPU/Memory Usage](#solution-4-resolve-high-cpumemory-usage)
5. [Solution 5: Repair Database Corruption](#solution-5-repair-database-corruption)
6. [Solution 6: Complete Monitoring Setup](#solution-6-complete-monitoring-setup)

---

## Solution 1: Complete MariaDB Installation and Setup

### Objective
Install MariaDB, configure it securely, and verify it's working correctly.

### Step-by-Step Implementation

#### Step 1: Install MariaDB

**On Ubuntu/Debian:**
```bash
# Update package index
sudo apt update

# Install MariaDB server and client
sudo apt install mariadb-server mariadb-client -y

# Verify installation
mariadb --version
# Output: mariadb  Ver 15.1 Distrib 10.6.12-MariaDB
```

**On CentOS/RHEL:**
```bash
# Install MariaDB
sudo dnf install mariadb-server mariadb -y

# Verify installation
mariadb --version
```

#### Step 2: Start and Enable Service

```bash
# Start MariaDB service
sudo systemctl start mariadb

# Check status
sudo systemctl status mariadb
# Should show: Active: active (running)

# Enable on boot
sudo systemctl enable mariadb

# Verify it's enabled
sudo systemctl is-enabled mariadb
# Output: enabled
```

#### Step 3: Verify Service is Running

```bash
# Check if process is running
ps aux | grep mysqld
# Should show: /usr/sbin/mysqld

# Check listening port
sudo netstat -tlnp | grep 3306
# Should show: tcp 0 0 127.0.0.1:3306 0.0.0.0:* LISTEN

# Alternative with ss
sudo ss -tlnp | grep 3306

# Check socket file
ls -la /var/run/mysqld/mysqld.sock
# Should exist with mysql:mysql ownership
```

#### Step 4: Initial Security Configuration

```bash
# Run security script
sudo mysql_secure_installation

# Follow prompts:
# 1. Enter current password for root (press Enter for none)
# 2. Switch to unix_socket authentication? [Y/n] Y
# 3. Change the root password? [Y/n] Y
#    New password: YourStrongPassword123!
#    Re-enter new password: YourStrongPassword123!
# 4. Remove anonymous users? [Y/n] Y
# 5. Disallow root login remotely? [Y/n] Y
# 6. Remove test database? [Y/n] Y
# 7. Reload privilege tables now? [Y/n] Y
```

#### Step 5: Test Connection

```bash
# Connect to MariaDB
sudo mysql -u root -p
# Enter password when prompted

# Once connected, run these commands:
MariaDB> SELECT VERSION();
# Output: 10.6.12-MariaDB

MariaDB> SHOW DATABASES;
# Should show: information_schema, mysql, performance_schema

MariaDB> SELECT USER();
# Output: root@localhost

MariaDB> SHOW VARIABLES LIKE 'port';
# Output: port | 3306

MariaDB> EXIT;
```

#### Step 6: Create Test Database and User

```sql
-- Connect as root
sudo mysql -u root -p

-- Create database
CREATE DATABASE testdb;

-- Create user
CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'TestPass123!';

-- Grant privileges
GRANT ALL PRIVILEGES ON testdb.* TO 'testuser'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;

-- Verify user
SELECT user, host FROM mysql.user WHERE user='testuser';

-- Exit
EXIT;
```

#### Step 7: Test New User

```bash
# Connect as new user
mysql -u testuser -p testdb
# Enter password: TestPass123!

# Create test table
CREATE TABLE test_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# Insert data
INSERT INTO test_table (name) VALUES ('Test Entry 1');
INSERT INTO test_table (name) VALUES ('Test Entry 2');

# Query data
SELECT * FROM test_table;

# Exit
EXIT;
```

#### Step 8: Configure Basic Settings

```bash
# Edit configuration file
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

# Add/modify these settings:
[mysqld]
# Basic Settings
max_connections = 200
connect_timeout = 10
wait_timeout = 600

# Logging
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2

# Character Set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Save and exit (Ctrl+X, Y, Enter)

# Restart MariaDB
sudo systemctl restart mariadb

# Verify changes
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"
mysql -u root -p -e "SHOW VARIABLES LIKE 'character_set_server';"
```

### Verification

```bash
# 1. Service is running
sudo systemctl status mariadb | grep Active
# Output: Active: active (running)

# 2. Can connect
mysql -u root -p -e "SELECT 1;"
# Output: 1

# 3. Test database exists
mysql -u root -p -e "SHOW DATABASES LIKE 'testdb';"
# Output: testdb

# 4. Logs are working
sudo tail -5 /var/log/mysql/error.log
# Should show recent log entries

# 5. Configuration applied
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"
# Output: max_connections | 200
```

### Expected Output

```
✓ MariaDB installed successfully
✓ Service running and enabled
✓ Security configuration completed
✓ Test database and user created
✓ Configuration optimized
✓ All connections working
```

---

## Solution 2: Troubleshoot Service Won't Start

### Objective
Diagnose and fix issues preventing MariaDB service from starting.

### Scenario
```bash
$ sudo systemctl start mariadb
Job for mariadb.service failed because the control process exited with error code.
See "systemctl status mariadb.service" and "journalctl -xe" for details.
```

### Step-by-Step Troubleshooting

#### Step 1: Check Service Status

```bash
# Get detailed status
sudo systemctl status mariadb -l

# Check recent logs
sudo journalctl -u mariadb -n 50 --no-pager

# Check error log
sudo tail -50 /var/log/mysql/error.log
```

#### Step 2: Common Issue - Port Already in Use

**Diagnosis:**
```bash
# Check if port 3306 is in use
sudo netstat -tlnp | grep 3306
# Or
sudo lsof -i :3306

# If output shows another process using port 3306:
# tcp 0 0 0.0.0.0:3306 0.0.0.0:* LISTEN 12345/mysqld
```

**Solution:**
```bash
# Option 1: Kill the conflicting process
sudo kill -9 12345

# Option 2: Change MariaDB port
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

# Change port:
[mysqld]
port = 3307

# Save and restart
sudo systemctl start mariadb
```

#### Step 3: Common Issue - Insufficient Disk Space

**Diagnosis:**
```bash
# Check disk space
df -h /var/lib/mysql

# If usage is > 95%:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1       50G   48G   2.0G  96% /
```

**Solution:**
```bash
# Check large files
sudo du -sh /var/lib/mysql/*
sudo du -sh /var/log/mysql/*

# Remove old binary logs
mysql -u root -p -e "PURGE BINARY LOGS BEFORE DATE(NOW() - INTERVAL 7 DAY);"

# Remove old slow query logs
sudo rm /var/log/mysql/mysql-slow.log.1
sudo rm /var/log/mysql/mysql-slow.log.2.gz

# Rotate logs
sudo mysqladmin -u root -p flush-logs

# Clean up temporary files
sudo rm -rf /tmp/mysql-*

# Try starting again
sudo systemctl start mariadb
```

#### Step 4: Common Issue - Permission Problems

**Diagnosis:**
```bash
# Check ownership
ls -la /var/lib/mysql
ls -la /var/run/mysqld

# If ownership is wrong:
# drwxr-xr-x 5 root root 4096 Jan 15 10:00 mysql
```

**Solution:**
```bash
# Fix ownership
sudo chown -R mysql:mysql /var/lib/mysql
sudo chown -R mysql:mysql /var/run/mysqld
sudo chown -R mysql:mysql /var/log/mysql

# Fix permissions
sudo chmod 755 /var/lib/mysql
sudo chmod 755 /var/run/mysqld
sudo chmod 755 /var/log/mysql

# Fix socket directory
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld
sudo chmod 755 /var/run/mysqld

# Try starting again
sudo systemctl start mariadb
```

#### Step 5: Common Issue - Corrupted Configuration

**Diagnosis:**
```bash
# Test configuration
sudo mysqld --help --verbose 2>&1 | grep "my.cnf"

# Check for syntax errors
sudo mysqld --validate-config

# If errors found, backup and reset
```

**Solution:**
```bash
# Backup current config
sudo cp /etc/mysql/my.cnf /etc/mysql/my.cnf.backup

# Create minimal config
sudo tee /etc/mysql/my.cnf > /dev/null <<EOF
[mysqld]
user = mysql
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/run/mysqld/mysqld.sock
port = 3306
datadir = /var/lib/mysql
EOF

# Try starting
sudo systemctl start mariadb

# If successful, gradually add back settings
```

#### Step 6: Common Issue - Corrupted System Tables

**Diagnosis:**
```bash
# Check error log for corruption messages
sudo grep -i corrupt /var/log/mysql/error.log
sudo grep -i crash /var/log/mysql/error.log
```

**Solution:**
```bash
# Start in safe mode
sudo mysqld_safe --skip-grant-tables --skip-networking &

# Wait a few seconds, then connect
mysql -u root

# Flush privileges
FLUSH PRIVILEGES;

# Exit
EXIT;

# Stop safe mode
sudo killall mysqld

# Start normally
sudo systemctl start mariadb

# If still failing, repair system tables
sudo mysql_upgrade -u root -p --force

# Restart
sudo systemctl restart mariadb
```

#### Step 7: Advanced - InnoDB Recovery

**If InnoDB tables are corrupted:**

```bash
# Edit config
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

# Add recovery mode (start with 1, increase if needed)
[mysqld]
innodb_force_recovery = 1

# Save and try starting
sudo systemctl start mariadb

# If successful, dump all databases
mysqldump -u root -p --all-databases > /tmp/all_databases.sql

# Stop MariaDB
sudo systemctl stop mariadb

# Remove recovery mode
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
# Comment out: # innodb_force_recovery = 1

# Backup and remove data directory
sudo mv /var/lib/mysql /var/lib/mysql.backup
sudo mkdir /var/lib/mysql
sudo chown mysql:mysql /var/lib/mysql

# Initialize new data directory
sudo mysql_install_db --user=mysql --datadir=/var/lib/mysql

# Start MariaDB
sudo systemctl start mariadb

# Restore data
mysql -u root -p < /tmp/all_databases.sql
```

### Verification

```bash
# Service should be running
sudo systemctl status mariadb
# Output: Active: active (running)

# Can connect
mysql -u root -p -e "SELECT 1;"

# No errors in log
sudo tail -20 /var/log/mysql/error.log
# Should show successful startup messages
```

---

## Solution 3: Fix Connection Issues

### Objective
Resolve various MariaDB connection problems.

### Scenario 1: Socket File Not Found

**Error:**
```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

**Solution:**
```bash
# Check if service is running
sudo systemctl status mariadb

# If running, find actual socket location
sudo find / -name "mysqld.sock" 2>/dev/null
# Output: /tmp/mysql.sock

# Update client configuration
sudo nano /etc/mysql/mariadb.conf.d/50-client.cnf

[client]
socket = /tmp/mysql.sock

# Or create symlink
sudo ln -s /tmp/mysql.sock /var/run/mysqld/mysqld.sock

# Test connection
mysql -u root -p
```

### Scenario 2: Access Denied

**Error:**
```
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

**Solution:**
```bash
# Stop MariaDB
sudo systemctl stop mariadb

# Start in safe mode
sudo mysqld_safe --skip-grant-tables --skip-networking &

# Connect without password
mysql -u root

# Reset password
USE mysql;
UPDATE user SET password=PASSWORD('NewPassword123!') WHERE User='root';
FLUSH PRIVILEGES;
EXIT;

# Stop safe mode
sudo killall mysqld

# Start normally
sudo systemctl start mariadb

# Test with new password
mysql -u root -p
# Enter: NewPassword123!
```

### Scenario 3: Can't Connect Remotely

**Error:**
```
ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.1.100' (111)
```

**Solution:**
```bash
# Step 1: Check bind address
sudo grep bind-address /etc/mysql/mariadb.conf.d/50-server.cnf

# If it shows 127.0.0.1, change to 0.0.0.0
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
bind-address = 0.0.0.0

# Step 2: Allow user from remote host
mysql -u root -p

CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'username'@'%';
FLUSH PRIVILEGES;
EXIT;

# Step 3: Configure firewall
sudo ufw allow 3306/tcp
# Or for specific IP
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Step 4: Restart MariaDB
sudo systemctl restart mariadb

# Step 5: Test from remote machine
mysql -h 192.168.1.100 -u username -p
```

### Scenario 4: Too Many Connections

**Error:**
```
ERROR 1040 (HY000): Too many connections
```

**Solution:**
```bash
# Check current connections
mysql -u root -p -e "SHOW PROCESSLIST;"

# Check max connections
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"

# Increase max connections temporarily
mysql -u root -p -e "SET GLOBAL max_connections = 500;"

# Make permanent
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
max_connections = 500

# Restart
sudo systemctl restart mariadb

# Kill idle connections
mysql -u root -p

SELECT CONCAT('KILL ', id, ';') 
FROM information_schema.processlist 
WHERE command = 'Sleep' AND time > 300;

# Copy and execute the KILL commands
```

### Verification

```bash
# Test local connection
mysql -u root -p -e "SELECT 'Local connection works';"

# Test remote connection (from another machine)
mysql -h server_ip -u username -p -e "SELECT 'Remote connection works';"

# Check active connections
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
```

---

## Solution 4: Resolve High CPU/Memory Usage

### Objective
Identify and fix performance issues causing high resource usage.

### Step-by-Step Implementation

#### Step 1: Identify the Problem

```bash
# Check CPU usage
top -u mysql
# Look for mysqld process

# Check memory usage
free -h
ps aux --sort=-%mem | head -10

# Check MariaDB status
mysql -u root -p -e "SHOW PROCESSLIST;"
```

#### Step 2: Find Slow Queries

```bash
# Enable slow query log
mysql -u root -p

SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;
SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';

# Wait a few minutes, then analyze
sudo mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log
```

#### Step 3: Kill Long-Running Queries

```sql
-- Connect to MariaDB
mysql -u root -p

-- Show all processes
SHOW FULL PROCESSLIST;

-- Kill specific query
KILL <process_id>;

-- Kill all queries from a user
SELECT CONCAT('KILL ', id, ';') 
FROM information_schema.processlist 
WHERE user='problematic_user' AND time > 60;

-- Execute the generated KILL commands
```

#### Step 4: Optimize Configuration

```bash
# Edit configuration
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
# Connection Settings
max_connections = 150
thread_cache_size = 16
connect_timeout = 10
wait_timeout = 300

# Buffer Settings (adjust based on available RAM)
innodb_buffer_pool_size = 2G  # 70-80% of available RAM
innodb_log_file_size = 256M
innodb_log_buffer_size = 16M

# Query Cache (if using older versions)
query_cache_type = 1
query_cache_size = 32M
query_cache_limit = 2M

# Table Cache
table_open_cache = 2000
table_definition_cache = 1400

# Temporary Tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Save and restart
sudo systemctl restart mariadb
```

#### Step 5: Optimize Queries and Indexes

```sql
-- Find tables without indexes
SELECT 
    table_schema,
    table_name,
    table_rows
FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema', 'mysql', 'performance_schema')
AND table_rows > 1000
ORDER BY table_rows DESC;

-- Analyze query performance
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';

-- Add missing indexes
CREATE INDEX idx_email ON users(email);

-- Analyze tables
ANALYZE TABLE users;

-- Optimize tables
OPTIMIZE TABLE users;
```

#### Step 6: Monitor Performance

```bash
# Install monitoring tools
sudo apt install mytop

# Run mytop
mytop -u root -p password

# Check InnoDB status
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G" | less

# Check buffer pool usage
mysql -u root -p -e "SHOW STATUS LIKE 'Innodb_buffer_pool%';"
```

### Verification

```bash
# CPU usage should be lower
top -u mysql

# Memory usage should be stable
free -h

# No slow queries
sudo tail -f /var/log/mysql/mysql-slow.log

# Connections under control
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
```

---

## Solution 5: Repair Database Corruption

### Objective
Detect and repair corrupted database tables.

### Step-by-Step Implementation

#### Step 1: Detect Corruption

```bash
# Check error log
sudo grep -i corrupt /var/log/mysql/error.log
sudo grep -i crash /var/log/mysql/error.log

# Check all tables
mysql -u root -p -e "CHECK TABLE database.table_name;"

# Check all databases
mysqlcheck -u root -p --all-databases
```

#### Step 2: Repair MyISAM Tables

```bash
# Online repair
mysqlcheck -u root -p --auto-repair --all-databases

# Specific database
mysqlcheck -u root -p --auto-repair database_name

# Specific table
mysqlcheck -u root -p --repair database_name table_name

# Offline repair (if online fails)
sudo systemctl stop mariadb
sudo myisamchk -r /var/lib/mysql/database/table_name.MYI
sudo systemctl start mariadb
```

#### Step 3: Repair InnoDB Tables

```sql
-- Try repair command
REPAIR TABLE table_name;

-- If repair fails, dump and restore
-- Dump table
mysqldump -u root -p database table_name > table_backup.sql

-- Drop table
DROP TABLE table_name;

-- Restore table
mysql -u root -p database < table_backup.sql
```

#### Step 4: InnoDB Force Recovery

```bash
# If InnoDB won't start, use force recovery
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
innodb_force_recovery = 1  # Start with 1, increase if needed (max 6)

# Restart
sudo systemctl restart mariadb

# Dump all databases
mysqldump -u root -p --all-databases > /tmp/all_databases_backup.sql

# Stop MariaDB
sudo systemctl stop mariadb

# Remove recovery mode
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
# Comment out: # innodb_force_recovery = 1

# Backup data directory
sudo mv /var/lib/mysql /var/lib/mysql.corrupt

# Initialize new data directory
sudo mkdir /var/lib/mysql
sudo chown mysql:mysql /var/lib/mysql
sudo mysql_install_db --user=mysql --datadir=/var/lib/mysql

# Start MariaDB
sudo systemctl start mariadb

# Restore data
mysql -u root -p < /tmp/all_databases_backup.sql
```

#### Step 5: Verify Repair

```bash
# Check all tables
mysqlcheck -u root -p --check --all-databases

# Check specific database
mysqlcheck -u root -p --check database_name

# Verify data integrity
mysql -u root -p database_name -e "SELECT COUNT(*) FROM table_name;"
```

### Verification

```bash
# No corruption errors
mysqlcheck -u root -p --all-databases
# Output: database.table OK

# Service running normally
sudo systemctl status mariadb

# Can query tables
mysql -u root -p -e "SELECT * FROM database.table LIMIT 5;"
```

---

## Solution 6: Complete Monitoring Setup

### Objective
Set up comprehensive monitoring for MariaDB.

### Step-by-Step Implementation

#### Step 1: Enable Performance Schema

```sql
-- Connect to MariaDB
mysql -u root -p

-- Enable performance schema
SET GLOBAL performance_schema = ON;

-- Verify
SHOW VARIABLES LIKE 'performance_schema';
```

#### Step 2: Configure Logging

```bash
# Edit configuration
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

[mysqld]
# Error Log
log_error = /var/log/mysql/error.log

# Slow Query Log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2
log_queries_not_using_indexes = 1

# Binary Log
log_bin = /var/log/mysql/mysql-bin.log
expire_logs_days = 7
max_binlog_size = 100M

# General Log (only for debugging)
# general_log = 1
# general_log_file = /var/log/mysql/mysql-general.log

# Restart
sudo systemctl restart mariadb
```

#### Step 3: Install Monitoring Tools

```bash
# Install mytop
sudo apt install mytop

# Install innotop
sudo apt install innotop

# Install mysqltuner
wget http://mysqltuner.pl/ -O mysqltuner.pl
chmod +x mysqltuner.pl
```

#### Step 4: Create Monitoring Script

```bash
# Create monitoring script
sudo nano /usr/local/bin/monitor-mariadb.sh

#!/bin/bash
# MariaDB Monitoring Script

MYSQL_USER="root"
MYSQL_PASS="your_password"
LOG_FILE="/var/log/mariadb-monitor.log"

echo "=== MariaDB Monitor - $(date) ===" >> $LOG_FILE

# Check service status
systemctl is-active mariadb >> $LOG_FILE 2>&1

# Check connections
mysql -u $MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Threads_connected';" >> $LOG_FILE 2>&1

# Check slow queries
mysql -u $MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Slow_queries';" >> $LOG_FILE 2>&1

# Check buffer pool usage
mysql -u $MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Innodb_buffer_pool_pages%';" >> $LOG_FILE 2>&1

# Check disk space
df -h /var/lib/mysql >> $LOG_FILE 2>&1

echo "" >> $LOG_FILE

# Make executable
sudo chmod +x /usr/local/bin/monitor-mariadb.sh

# Add to cron (run every 5 minutes)
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/monitor-mariadb.sh") | crontab -
```

#### Step 5: Set Up Alerts

```bash
# Create alert script
sudo nano /usr/local/bin/mariadb-alerts.sh

#!/bin/bash
# MariaDB Alert Script

MYSQL_USER="root"
MYSQL_PASS="your_password"
ALERT_EMAIL="admin@example.com"

# Check if service is running
if ! systemctl is-active --quiet mariadb; then
    echo "MariaDB is down!" | mail -s "ALERT: MariaDB Down" $ALERT_EMAIL
fi

# Check connections
CONNECTIONS=$(mysql -u $MYSQL_USER -p$MYSQL_PASS -e "SHOW STATUS LIKE 'Threads_connected';" | awk '{print $2}' | tail -1)
if [ $CONNECTIONS -gt 150 ]; then
    echo "High connections: $CONNECTIONS" | mail -s "ALERT: High Connections" $ALERT_EMAIL
fi

# Check disk space
DISK_USAGE=$(df -h /var/lib/mysql | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "Disk usage: $DISK_USAGE%" | mail -s "ALERT: High Disk Usage" $ALERT_EMAIL
fi

# Make executable
sudo chmod +x /usr/local/bin/mariadb-alerts.sh

# Add to cron (run every 10 minutes)
(crontab -l 2>/dev/null; echo "*/10 * * * * /usr/local/bin/mariadb-alerts.sh") | crontab -
```

#### Step 6: Use mysqltuner

```bash
# Run mysqltuner
./mysqltuner.pl

# Review recommendations and apply them
# Example output:
# [!!] Maximum possible memory usage: 4.2G (85% of installed RAM)
# [OK] Slow queries: 0% (0/1K)
# [!!] Temporary tables created on disk: 25% (250 on disk / 1K total)
```

### Verification

```bash
# Check logs are being written
sudo tail -f /var/log/mysql/error.log
sudo tail -f /var/log/mysql/mysql-slow.log

# Check monitoring script
sudo /usr/local/bin/monitor-mariadb.sh
cat /var/log/mariadb-monitor.log

# Test mytop
mytop -u root -p password

# Run mysqltuner
./mysqltuner.pl
```

---

## Summary

All solutions have been implemented and tested:

1. ✅ Complete MariaDB installation and setup
2. ✅ Service startup troubleshooting
3. ✅ Connection issue resolution
4. ✅ Performance optimization
5. ✅ Database corruption repair
6. ✅ Comprehensive monitoring setup

### Key Commands Reference

```bash
# Service Management
sudo systemctl start/stop/restart/status mariadb

# Connection
mysql -u root -p
mysql -h hostname -u username -p database

# Troubleshooting
sudo tail -f /var/log/mysql/error.log
mysql -u root -p -e "SHOW PROCESSLIST;"
mysqlcheck -u root -p --all-databases

# Monitoring
mytop -u root -p password
./mysqltuner.pl
mysql -u root -p -e "SHOW STATUS;"

# Backup
mysqldump -u root -p database > backup.sql
mysql -u root -p database < backup.sql
```

---

**Last Updated:** 2024-01-16
**Version:** 1.0