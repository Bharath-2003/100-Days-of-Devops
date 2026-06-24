# Day 9: MariaDB Service Troubleshooting

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [MariaDB Overview](#mariadb-overview)
3. [Installation](#installation)
4. [Service Management](#service-management)
5. [Common Issues and Troubleshooting](#common-issues-and-troubleshooting)
6. [Log Analysis](#log-analysis)
7. [Configuration Files](#configuration-files)
8. [Performance Troubleshooting](#performance-troubleshooting)
9. [Security Issues](#security-issues)
10. [Backup and Recovery](#backup-and-recovery)
11. [Monitoring and Diagnostics](#monitoring-and-diagnostics)
12. [Best Practices](#best-practices)

---

## Introduction

MariaDB is a popular open-source relational database management system (RDBMS) that is a fork of MySQL. It's designed to be highly compatible with MySQL while offering additional features and improvements. This guide focuses on troubleshooting MariaDB service issues, which is a critical skill for DevOps engineers and database administrators.

### Why MariaDB Troubleshooting Matters

- **High Availability**: Database downtime directly impacts application availability
- **Data Integrity**: Proper troubleshooting prevents data loss
- **Performance**: Identifying and resolving bottlenecks improves application performance
- **Security**: Detecting and fixing security issues protects sensitive data
- **Cost Efficiency**: Quick problem resolution reduces operational costs

---

## MariaDB Overview

### Architecture Components

```
┌─────────────────────────────────────────┐
│         Client Applications             │
│  (MySQL CLI, phpMyAdmin, Applications)  │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         Connection Layer                │
│  (Authentication, Connection Pooling)   │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         SQL Layer                       │
│  (Parser, Optimizer, Query Cache)       │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         Storage Engine Layer            │
│  (InnoDB, MyISAM, Memory, etc.)        │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         File System                     │
│  (Data Files, Log Files, Config)       │
└─────────────────────────────────────────┘
```

### Key Components

1. **MariaDB Server (mysqld)**: The main database server process
2. **Storage Engines**: InnoDB (default), MyISAM, Memory, etc.
3. **Configuration Files**: my.cnf or my.ini
4. **Data Directory**: Stores database files
5. **Log Files**: Error logs, slow query logs, binary logs
6. **System Tables**: mysql database with user privileges and system info

### Default Ports and Paths

```bash
# Default Port
Port: 3306

# Configuration Files (Linux)
/etc/my.cnf
/etc/mysql/my.cnf
/etc/mysql/mariadb.conf.d/50-server.cnf
~/.my.cnf

# Data Directory
/var/lib/mysql/

# Log Files
/var/log/mysql/error.log
/var/log/mysql/mysql-slow.log
/var/lib/mysql/mysql-bin.log

# Socket File
/var/run/mysqld/mysqld.sock
/tmp/mysql.sock
```

---

## Installation

### Ubuntu/Debian Installation

```bash
# Update package index
sudo apt update

# Install MariaDB server
sudo apt install mariadb-server mariadb-client -y

# Check version
mariadb --version

# Start service
sudo systemctl start mariadb

# Enable on boot
sudo systemctl enable mariadb

# Check status
sudo systemctl status mariadb
```

### CentOS/RHEL Installation

```bash
# Install MariaDB
sudo dnf install mariadb-server mariadb -y

# Start service
sudo systemctl start mariadb

# Enable on boot
sudo systemctl enable mariadb

# Check status
sudo systemctl status mariadb
```

### Initial Security Setup

```bash
# Run security script
sudo mysql_secure_installation

# Steps:
# 1. Set root password
# 2. Remove anonymous users
# 3. Disallow root login remotely
# 4. Remove test database
# 5. Reload privilege tables
```

### Verify Installation

```bash
# Connect to MariaDB
sudo mysql -u root -p

# Check version
MariaDB> SELECT VERSION();

# Show databases
MariaDB> SHOW DATABASES;

# Exit
MariaDB> EXIT;
```

---

## Service Management

### Systemd Commands (Modern Linux)

```bash
# Start service
sudo systemctl start mariadb

# Stop service
sudo systemctl stop mariadb

# Restart service
sudo systemctl restart mariadb

# Reload configuration
sudo systemctl reload mariadb

# Check status
sudo systemctl status mariadb

# Enable on boot
sudo systemctl enable mariadb

# Disable on boot
sudo systemctl disable mariadb

# Check if enabled
sudo systemctl is-enabled mariadb

# View service logs
sudo journalctl -u mariadb -f

# View recent logs
sudo journalctl -u mariadb --since "1 hour ago"
```

### Service Status Interpretation

```bash
# Active (running) - Service is running normally
● mariadb.service - MariaDB database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled)
   Active: active (running) since Mon 2024-01-15 10:00:00 UTC
   
# Failed - Service failed to start
● mariadb.service - MariaDB database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled)
   Active: failed (Result: exit-code)
   
# Inactive (dead) - Service is stopped
● mariadb.service - MariaDB database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled)
   Active: inactive (dead)
```

### Manual Service Control

```bash
# Start MariaDB manually (for debugging)
sudo mysqld_safe --user=mysql &

# Stop MariaDB
sudo mysqladmin -u root -p shutdown

# Check if MariaDB is running
ps aux | grep mysqld
pgrep -a mysqld

# Check listening ports
sudo netstat -tlnp | grep 3306
sudo ss -tlnp | grep 3306
```

---

## Common Issues and Troubleshooting

### Issue 1: Service Won't Start

**Symptoms:**
```bash
$ sudo systemctl start mariadb
Job for mariadb.service failed because the control process exited with error code.
```

**Diagnosis Steps:**

```bash
# 1. Check service status
sudo systemctl status mariadb -l

# 2. Check error logs
sudo tail -f /var/log/mysql/error.log

# 3. Check if port is already in use
sudo netstat -tlnp | grep 3306
sudo lsof -i :3306

# 4. Check disk space
df -h /var/lib/mysql

# 5. Check permissions
ls -la /var/lib/mysql
ls -la /var/run/mysqld

# 6. Check configuration syntax
sudo mysqld --help --verbose | grep "my.cnf"
```

**Common Causes and Solutions:**

**A. Port Already in Use**
```bash
# Find process using port 3306
sudo lsof -i :3306

# Kill the process
sudo kill -9 <PID>

# Or change MariaDB port in /etc/mysql/my.cnf
[mysqld]
port = 3307
```

**B. Insufficient Disk Space**
```bash
# Check disk usage
df -h

# Clean up logs
sudo rm /var/log/mysql/mysql-slow.log.old
sudo rm /var/lib/mysql/mysql-bin.00000*

# Rotate logs
sudo mysqladmin -u root -p flush-logs
```

**C. Permission Issues**
```bash
# Fix ownership
sudo chown -R mysql:mysql /var/lib/mysql
sudo chown -R mysql:mysql /var/run/mysqld

# Fix permissions
sudo chmod 755 /var/lib/mysql
sudo chmod 755 /var/run/mysqld
```

**D. Corrupted System Tables**
```bash
# Start in safe mode
sudo mysqld_safe --skip-grant-tables &

# Connect without password
mysql -u root

# Flush privileges
FLUSH PRIVILEGES;

# Exit and restart normally
sudo systemctl restart mariadb
```

### Issue 2: Can't Connect to MariaDB

**Symptoms:**
```bash
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'
ERROR 2003 (HY000): Can't connect to MySQL server on 'localhost' (111)
ERROR 1045 (28000): Access denied for user 'root'@'localhost'
```

**Diagnosis Steps:**

```bash
# 1. Check if service is running
sudo systemctl status mariadb

# 2. Check socket file exists
ls -la /var/run/mysqld/mysqld.sock

# 3. Check if listening on port
sudo netstat -tlnp | grep 3306

# 4. Test connection
mysql -u root -p
mysql -h localhost -u root -p
mysql -h 127.0.0.1 -u root -p

# 5. Check bind address
sudo grep bind-address /etc/mysql/my.cnf
```

**Solutions:**

**A. Socket File Missing**
```bash
# Check socket location in config
sudo grep socket /etc/mysql/my.cnf

# Create socket directory
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld

# Restart service
sudo systemctl restart mariadb
```

**B. Wrong Socket Path**
```bash
# Find actual socket
sudo find / -name "mysqld.sock" 2>/dev/null

# Update my.cnf
[client]
socket = /var/run/mysqld/mysqld.sock

[mysqld]
socket = /var/run/mysqld/mysqld.sock
```

**C. Firewall Blocking**
```bash
# Check firewall status
sudo ufw status
sudo firewall-cmd --list-all

# Allow MariaDB port
sudo ufw allow 3306/tcp
sudo firewall-cmd --permanent --add-service=mysql
sudo firewall-cmd --reload
```

**D. Bind Address Issue**
```bash
# Edit /etc/mysql/my.cnf
[mysqld]
bind-address = 0.0.0.0  # Listen on all interfaces
# Or
bind-address = 127.0.0.1  # Listen only on localhost

# Restart service
sudo systemctl restart mariadb
```

**E. Authentication Issues**
```bash
# Reset root password
sudo systemctl stop mariadb
sudo mysqld_safe --skip-grant-tables &

# Connect and reset password
mysql -u root

USE mysql;
UPDATE user SET password=PASSWORD('new_password') WHERE User='root';
FLUSH PRIVILEGES;
EXIT;

# Restart normally
sudo systemctl restart mariadb
```

### Issue 3: High CPU/Memory Usage

**Symptoms:**
```bash
# High CPU usage
top
# mysqld process consuming 90%+ CPU

# High memory usage
free -h
# Most memory consumed by mysqld
```

**Diagnosis Steps:**

```bash
# 1. Check running queries
mysql -u root -p -e "SHOW PROCESSLIST;"

# 2. Check slow queries
mysql -u root -p -e "SHOW VARIABLES LIKE 'slow_query%';"

# 3. Check InnoDB status
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G"

# 4. Check table locks
mysql -u root -p -e "SHOW OPEN TABLES WHERE In_use > 0;"

# 5. Monitor in real-time
mytop -u root -p password
```

**Solutions:**

**A. Kill Long-Running Queries**
```sql
-- Show processlist
SHOW PROCESSLIST;

-- Kill specific query
KILL <process_id>;

-- Kill all queries from a user
SELECT CONCAT('KILL ',id,';') FROM information_schema.processlist 
WHERE user='username';
```

**B. Optimize Configuration**
```ini
# /etc/mysql/my.cnf
[mysqld]
# Reduce memory usage
innodb_buffer_pool_size = 1G  # 70-80% of available RAM
max_connections = 100
thread_cache_size = 8
query_cache_size = 16M
tmp_table_size = 32M
max_heap_table_size = 32M

# Improve performance
innodb_flush_log_at_trx_commit = 2
innodb_log_file_size = 256M
```

**C. Optimize Queries**
```sql
-- Analyze slow queries
SELECT * FROM mysql.slow_log ORDER BY query_time DESC LIMIT 10;

-- Add indexes
SHOW INDEX FROM table_name;
CREATE INDEX idx_column ON table_name(column_name);

-- Analyze tables
ANALYZE TABLE table_name;

-- Optimize tables
OPTIMIZE TABLE table_name;
```

### Issue 4: Database Corruption

**Symptoms:**
```bash
ERROR 1034 (HY000): Index for table 'table_name' is corrupt; try to repair it
ERROR 1016 (HY000): Can't open file: 'table_name.ibd'
```

**Diagnosis Steps:**

```bash
# 1. Check error logs
sudo tail -100 /var/log/mysql/error.log

# 2. Check table status
mysql -u root -p -e "CHECK TABLE database.table_name;"

# 3. Check InnoDB status
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G"

# 4. Check file system
sudo fsck /dev/sda1
```

**Solutions:**

**A. Repair MyISAM Tables**
```bash
# Using mysqlcheck
sudo mysqlcheck -u root -p --auto-repair --all-databases

# Using myisamchk (offline)
sudo systemctl stop mariadb
sudo myisamchk -r /var/lib/mysql/database/table_name.MYI
sudo systemctl start mariadb
```

**B. Repair InnoDB Tables**
```sql
-- Try repair
REPAIR TABLE table_name;

-- If repair fails, dump and restore
mysqldump -u root -p database table_name > table_backup.sql
DROP TABLE table_name;
mysql -u root -p database < table_backup.sql
```

**C. InnoDB Recovery**
```ini
# Add to /etc/mysql/my.cnf
[mysqld]
innodb_force_recovery = 1  # Start with 1, increase if needed (max 6)

# Restart MariaDB
sudo systemctl restart mariadb

# Dump databases
mysqldump -u root -p --all-databases > all_databases.sql

# Remove recovery mode and restore
# Comment out innodb_force_recovery
sudo systemctl restart mariadb
```

### Issue 5: Replication Issues

**Symptoms:**
```bash
Slave_IO_Running: No
Slave_SQL_Running: No
Last_Error: Error 'Duplicate entry' on query
```

**Diagnosis Steps:**

```sql
-- Check slave status
SHOW SLAVE STATUS\G

-- Check master status
SHOW MASTER STATUS;

-- Check replication errors
SELECT * FROM performance_schema.replication_connection_status;
```

**Solutions:**

**A. Restart Replication**
```sql
-- On slave
STOP SLAVE;
START SLAVE;

-- Check status
SHOW SLAVE STATUS\G
```

**B. Skip Errors**
```sql
-- Skip one error
STOP SLAVE;
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE;

-- Or in my.cnf
[mysqld]
slave-skip-errors = 1062,1053
```

**C. Rebuild Slave**
```bash
# On master
mysqldump -u root -p --all-databases --master-data=2 > master_backup.sql

# Transfer to slave
scp master_backup.sql slave:/tmp/

# On slave
mysql -u root -p < /tmp/master_backup.sql

# Configure replication
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=12345;

START SLAVE;
```

---

## Log Analysis

### Error Log

```bash
# Location
/var/log/mysql/error.log
/var/lib/mysql/hostname.err

# View in real-time
sudo tail -f /var/log/mysql/error.log

# Search for errors
sudo grep -i error /var/log/mysql/error.log
sudo grep -i warning /var/log/mysql/error.log

# Common error patterns
sudo grep "Can't connect" /var/log/mysql/error.log
sudo grep "Access denied" /var/log/mysql/error.log
sudo grep "Crash" /var/log/mysql/error.log
```

### Slow Query Log

```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;
SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';

-- Check settings
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
```

```bash
# Analyze slow queries
sudo mysqldumpslow /var/log/mysql/mysql-slow.log

# Top 10 slowest queries
sudo mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log

# Most frequent queries
sudo mysqldumpslow -s c -t 10 /var/log/mysql/mysql-slow.log
```

### Binary Log

```sql
-- Show binary logs
SHOW BINARY LOGS;

-- Show current binary log
SHOW MASTER STATUS;

-- View binary log events
SHOW BINLOG EVENTS IN 'mysql-bin.000001';

-- Purge old binary logs
PURGE BINARY LOGS BEFORE '2024-01-01 00:00:00';
PURGE BINARY LOGS TO 'mysql-bin.000010';
```

### General Query Log

```sql
-- Enable general log (not recommended for production)
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/mysql-general.log';

-- Disable when done
SET GLOBAL general_log = 'OFF';
```

---

## Configuration Files

### Main Configuration File

```ini
# /etc/mysql/my.cnf or /etc/mysql/mariadb.conf.d/50-server.cnf

[client]
port = 3306
socket = /var/run/mysqld/mysqld.sock

[mysqld]
# Basic Settings
user = mysql
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/run/mysqld/mysqld.sock
port = 3306
basedir = /usr
datadir = /var/lib/mysql
tmpdir = /tmp

# Networking
bind-address = 0.0.0.0
max_connections = 200
connect_timeout = 10
wait_timeout = 600
max_allowed_packet = 64M

# Logging
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2
log_queries_not_using_indexes = 1

# InnoDB Settings
innodb_buffer_pool_size = 2G
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1

# Query Cache (deprecated in newer versions)
query_cache_type = 1
query_cache_size = 32M
query_cache_limit = 2M

# Character Set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysqldump]
quick
quote-names
max_allowed_packet = 64M
```

### Configuration Validation

```bash
# Check configuration syntax
sudo mysqld --help --verbose | head -n 20

# Show all variables
mysql -u root -p -e "SHOW VARIABLES;"

# Show specific variable
mysql -u root -p -e "SHOW VARIABLES LIKE 'max_connections';"

# Show runtime configuration
mysql -u root -p -e "SHOW GLOBAL STATUS;"
```

---

## Performance Troubleshooting

### Performance Metrics

```sql
-- Connection statistics
SHOW STATUS LIKE 'Threads%';
SHOW STATUS LIKE 'Connections';
SHOW STATUS LIKE 'Max_used_connections';

-- Query statistics
SHOW STATUS LIKE 'Questions';
SHOW STATUS LIKE 'Queries';
SHOW STATUS LIKE 'Slow_queries';

-- Table statistics
SHOW STATUS LIKE 'Open_tables';
SHOW STATUS LIKE 'Opened_tables';

-- InnoDB statistics
SHOW STATUS LIKE 'Innodb_buffer_pool%';
SHOW STATUS LIKE 'Innodb_rows%';
```

### Query Optimization

```sql
-- Explain query execution plan
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';

-- Extended explain
EXPLAIN EXTENDED SELECT * FROM users WHERE email = 'user@example.com';
SHOW WARNINGS;

-- Analyze query
ANALYZE TABLE users;

-- Show indexes
SHOW INDEX FROM users;

-- Create index
CREATE INDEX idx_email ON users(email);

-- Show table status
SHOW TABLE STATUS LIKE 'users';
```

### Performance Schema

```sql
-- Enable performance schema
SET GLOBAL performance_schema = ON;

-- Top queries by execution time
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;

-- Table I/O statistics
SELECT * FROM performance_schema.table_io_waits_summary_by_table
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;

-- Connection statistics
SELECT * FROM performance_schema.accounts;
```

---

## Security Issues

### User Management

```sql
-- Show users
SELECT user, host FROM mysql.user;

-- Create user
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';

-- Grant privileges
GRANT ALL PRIVILEGES ON database.* TO 'username'@'localhost';
GRANT SELECT, INSERT ON database.* TO 'username'@'%';

-- Show grants
SHOW GRANTS FOR 'username'@'localhost';

-- Revoke privileges
REVOKE ALL PRIVILEGES ON database.* FROM 'username'@'localhost';

-- Drop user
DROP USER 'username'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;
```

### Security Hardening

```bash
# 1. Run security script
sudo mysql_secure_installation

# 2. Disable remote root login
mysql -u root -p
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
FLUSH PRIVILEGES;

# 3. Remove anonymous users
DELETE FROM mysql.user WHERE User='';
FLUSH PRIVILEGES;

# 4. Remove test database
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
FLUSH PRIVILEGES;
```

### SSL/TLS Configuration

```ini
# /etc/mysql/my.cnf
[mysqld]
ssl-ca=/etc/mysql/ssl/ca-cert.pem
ssl-cert=/etc/mysql/ssl/server-cert.pem
ssl-key=/etc/mysql/ssl/server-key.pem
require_secure_transport=ON
```

```sql
-- Require SSL for user
ALTER USER 'username'@'%' REQUIRE SSL;

-- Check SSL status
SHOW VARIABLES LIKE '%ssl%';
```

---

## Backup and Recovery

### Logical Backup (mysqldump)

```bash
# Backup single database
mysqldump -u root -p database_name > database_backup.sql

# Backup all databases
mysqldump -u root -p --all-databases > all_databases.sql

# Backup with compression
mysqldump -u root -p database_name | gzip > database_backup.sql.gz

# Backup specific tables
mysqldump -u root -p database_name table1 table2 > tables_backup.sql

# Backup structure only
mysqldump -u root -p --no-data database_name > structure.sql

# Backup data only
mysqldump -u root -p --no-create-info database_name > data.sql
```

### Restore from Backup

```bash
# Restore database
mysql -u root -p database_name < database_backup.sql

# Restore compressed backup
gunzip < database_backup.sql.gz | mysql -u root -p database_name

# Restore all databases
mysql -u root -p < all_databases.sql

# Create database before restore
mysql -u root -p -e "CREATE DATABASE database_name;"
mysql -u root -p database_name < database_backup.sql
```

### Physical Backup (Mariabackup)

```bash
# Install mariabackup
sudo apt install mariadb-backup

# Full backup
sudo mariabackup --backup --target-dir=/backup/full --user=root --password=password

# Prepare backup
sudo mariabackup --prepare --target-dir=/backup/full

# Restore backup
sudo systemctl stop mariadb
sudo rm -rf /var/lib/mysql/*
sudo mariabackup --copy-back --target-dir=/backup/full
sudo chown -R mysql:mysql /var/lib/mysql
sudo systemctl start mariadb
```

---

## Monitoring and Diagnostics

### Built-in Monitoring

```sql
-- Show processlist
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- Show status
SHOW STATUS;
SHOW GLOBAL STATUS;

-- Show variables
SHOW VARIABLES;
SHOW GLOBAL VARIABLES;

-- Show engine status
SHOW ENGINE INNODB STATUS\G

-- Show open tables
SHOW OPEN TABLES;

-- Show table locks
SHOW OPEN TABLES WHERE In_use > 0;
```

### Command-Line Tools

```bash
# mysqladmin
mysqladmin -u root -p status
mysqladmin -u root -p processlist
mysqladmin -u root -p extended-status
mysqladmin -u root -p variables

# mytop (real-time monitoring)
mytop -u root -p password

# innotop (InnoDB monitoring)
innotop -u root -p password

# mysqltuner (configuration recommendations)
wget http://mysqltuner.pl/ -O mysqltuner.pl
perl mysqltuner.pl
```

### External Monitoring Tools

```bash
# Prometheus MySQL Exporter
docker run -d \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(localhost:3306)/" \
  prom/mysqld-exporter

# Grafana Dashboard
# Import dashboard ID: 7362 (MySQL Overview)
```

---

## Best Practices

### 1. Regular Maintenance

```bash
# Daily tasks
- Monitor error logs
- Check disk space
- Review slow queries
- Check replication status

# Weekly tasks
- Analyze and optimize tables
- Review user accounts
- Check backup integrity
- Update statistics

# Monthly tasks
- Review configuration
- Update MariaDB version
- Security audit
- Performance tuning
```

### 2. Configuration Management

```ini
# Use version control for configs
git init /etc/mysql
git add /etc/mysql/my.cnf
git commit -m "Initial MariaDB configuration"

# Document changes
# Always comment configuration changes
[mysqld]
# Increased for high-traffic application - 2024-01-15
max_connections = 500
```

### 3. Monitoring Strategy

```bash
# Set up alerts for:
- Service down
- High CPU/memory usage
- Disk space < 20%
- Replication lag > 60 seconds
- Slow queries > threshold
- Failed login attempts
```

### 4. Backup Strategy

```bash
# 3-2-1 Backup Rule
- 3 copies of data
- 2 different media types
- 1 offsite backup

# Automated backups
0 2 * * * /usr/local/bin/backup-mariadb.sh
```

### 5. Security Checklist

```bash
✓ Run mysql_secure_installation
✓ Use strong passwords
✓ Limit remote access
✓ Use SSL/TLS
✓ Regular security updates
✓ Audit user privileges
✓ Enable binary logging
✓ Encrypt backups
✓ Monitor failed logins
✓ Use firewall rules
```

---

## Summary

MariaDB troubleshooting requires a systematic approach:

1. **Identify the Problem**: Check logs, status, and error messages
2. **Diagnose the Cause**: Use diagnostic tools and commands
3. **Implement Solution**: Apply fixes based on root cause
4. **Verify Resolution**: Test and monitor after changes
5. **Document**: Record issues and solutions for future reference

### Key Takeaways

- Always check error logs first
- Use systemctl for service management
- Monitor performance metrics regularly
- Keep backups current and tested
- Follow security best practices
- Document all changes
- Test in non-production first

### Next Steps

- Practice common troubleshooting scenarios
- Set up monitoring and alerting
- Create backup and recovery procedures
- Implement security hardening
- Learn advanced performance tuning

---

**Last Updated:** 2024-01-16
**Version:** 1.0