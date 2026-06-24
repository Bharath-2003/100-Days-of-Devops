# Day 9: MariaDB Service Troubleshooting - Quick Revision Guide

## 🎯 Quick Summary

MariaDB is an open-source relational database management system (RDBMS) that is a fork of MySQL. Troubleshooting MariaDB involves diagnosing service issues, connection problems, performance bottlenecks, and data corruption.

**Key Points:**
- MariaDB uses port 3306 by default
- Main config file: `/etc/mysql/my.cnf` or `/etc/mysql/mariadb.conf.d/50-server.cnf`
- Data directory: `/var/lib/mysql/`
- Error log: `/var/log/mysql/error.log`
- Socket file: `/var/run/mysqld/mysqld.sock`

---

## ⚡ Essential Commands

### Service Management

```bash
# Start/Stop/Restart
sudo systemctl start mariadb
sudo systemctl stop mariadb
sudo systemctl restart mariadb
sudo systemctl reload mariadb

# Check status
sudo systemctl status mariadb

# Enable/Disable on boot
sudo systemctl enable mariadb
sudo systemctl disable mariadb

# View logs
sudo journalctl -u mariadb -f
sudo tail -f /var/log/mysql/error.log
```

### Connection Commands

```bash
# Connect locally
mysql -u root -p
sudo mysql

# Connect remotely
mysql -h hostname -u username -p database

# Execute single command
mysql -u root -p -e "SHOW DATABASES;"

# Execute script
mysql -u root -p database < script.sql

# Connect with specific socket
mysql -u root -p --socket=/var/run/mysqld/mysqld.sock
```

### Diagnostic Commands

```bash
# Check if running
ps aux | grep mysqld
pgrep -a mysqld

# Check port
sudo netstat -tlnp | grep 3306
sudo ss -tlnp | grep 3306
sudo lsof -i :3306

# Check socket
ls -la /var/run/mysqld/mysqld.sock

# Check disk space
df -h /var/lib/mysql

# Check permissions
ls -la /var/lib/mysql
ls -la /var/run/mysqld
```

### Database Commands

```sql
-- Show version
SELECT VERSION();

-- Show databases
SHOW DATABASES;

-- Show tables
SHOW TABLES;

-- Show processlist
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- Show status
SHOW STATUS;
SHOW GLOBAL STATUS;

-- Show variables
SHOW VARIABLES;
SHOW VARIABLES LIKE 'max_connections';

-- Kill process
KILL <process_id>;
```

---

## 🔧 Common Issues and Quick Fixes

### Issue 1: Service Won't Start

**Quick Diagnosis:**
```bash
sudo systemctl status mariadb -l
sudo tail -50 /var/log/mysql/error.log
sudo netstat -tlnp | grep 3306
df -h /var/lib/mysql
```

**Quick Fixes:**
```bash
# Port in use
sudo lsof -i :3306
sudo kill -9 <PID>

# Permission issues
sudo chown -R mysql:mysql /var/lib/mysql
sudo chown -R mysql:mysql /var/run/mysqld

# Disk space
sudo rm /var/log/mysql/mysql-slow.log.old
mysql -u root -p -e "PURGE BINARY LOGS BEFORE DATE(NOW() - INTERVAL 7 DAY);"

# Socket directory
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld
```

### Issue 2: Can't Connect

**Quick Diagnosis:**
```bash
sudo systemctl status mariadb
ls -la /var/run/mysqld/mysqld.sock
sudo netstat -tlnp | grep 3306
```

**Quick Fixes:**
```bash
# Reset root password
sudo systemctl stop mariadb
sudo mysqld_safe --skip-grant-tables &
mysql -u root
USE mysql;
UPDATE user SET password=PASSWORD('newpass') WHERE User='root';
FLUSH PRIVILEGES;
EXIT;
sudo systemctl restart mariadb

# Fix bind address
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
# Change: bind-address = 0.0.0.0
sudo systemctl restart mariadb

# Allow remote access
mysql -u root -p
CREATE USER 'user'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';
FLUSH PRIVILEGES;

# Firewall
sudo ufw allow 3306/tcp
```

### Issue 3: High CPU/Memory

**Quick Diagnosis:**
```bash
top -u mysql
mysql -u root -p -e "SHOW PROCESSLIST;"
sudo tail -f /var/log/mysql/mysql-slow.log
```

**Quick Fixes:**
```sql
-- Kill long queries
SHOW PROCESSLIST;
KILL <process_id>;

-- Optimize configuration
SET GLOBAL max_connections = 200;
SET GLOBAL query_cache_size = 32M;

-- Optimize tables
ANALYZE TABLE table_name;
OPTIMIZE TABLE table_name;
```

### Issue 4: Database Corruption

**Quick Diagnosis:**
```bash
sudo grep -i corrupt /var/log/mysql/error.log
mysqlcheck -u root -p --all-databases
```

**Quick Fixes:**
```bash
# Repair all databases
mysqlcheck -u root -p --auto-repair --all-databases

# Repair specific table
mysqlcheck -u root -p --repair database table

# InnoDB recovery
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
# Add: innodb_force_recovery = 1
sudo systemctl restart mariadb
mysqldump -u root -p --all-databases > backup.sql
# Remove innodb_force_recovery
sudo systemctl restart mariadb
```

---

## 📋 Configuration Quick Reference

### Essential Settings

```ini
[mysqld]
# Basic
port = 3306
socket = /var/run/mysqld/mysqld.sock
datadir = /var/lib/mysql
bind-address = 0.0.0.0

# Connections
max_connections = 200
connect_timeout = 10
wait_timeout = 600

# Logging
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2

# InnoDB
innodb_buffer_pool_size = 2G
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2

# Character Set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

---

## 🔐 Security Quick Reference

### User Management

```sql
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
# Run security script
sudo mysql_secure_installation

# Disable remote root
mysql -u root -p
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
FLUSH PRIVILEGES;

# Remove anonymous users
DELETE FROM mysql.user WHERE User='';
FLUSH PRIVILEGES;

# Remove test database
DROP DATABASE IF EXISTS test;
```

---

## 💾 Backup and Restore

### Backup Commands

```bash
# Single database
mysqldump -u root -p database > backup.sql

# All databases
mysqldump -u root -p --all-databases > all_backup.sql

# Compressed backup
mysqldump -u root -p database | gzip > backup.sql.gz

# Structure only
mysqldump -u root -p --no-data database > structure.sql

# Data only
mysqldump -u root -p --no-create-info database > data.sql

# Specific tables
mysqldump -u root -p database table1 table2 > tables.sql
```

### Restore Commands

```bash
# Restore database
mysql -u root -p database < backup.sql

# Restore compressed
gunzip < backup.sql.gz | mysql -u root -p database

# Restore all databases
mysql -u root -p < all_backup.sql

# Create and restore
mysql -u root -p -e "CREATE DATABASE database;"
mysql -u root -p database < backup.sql
```

---

## 📊 Performance Monitoring

### Key Metrics

```sql
-- Connections
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
SHOW VARIABLES LIKE 'max_connections';

-- Queries
SHOW STATUS LIKE 'Questions';
SHOW STATUS LIKE 'Slow_queries';

-- Tables
SHOW STATUS LIKE 'Open_tables';
SHOW STATUS LIKE 'Opened_tables';

-- InnoDB
SHOW STATUS LIKE 'Innodb_buffer_pool%';
SHOW STATUS LIKE 'Innodb_rows%';

-- Cache
SHOW STATUS LIKE 'Qcache%';
```

### Performance Tools

```bash
# mytop - real-time monitoring
mytop -u root -p password

# mysqltuner - configuration recommendations
wget http://mysqltuner.pl/ -O mysqltuner.pl
perl mysqltuner.pl

# Slow query analysis
mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log

# Check tables
mysqlcheck -u root -p --all-databases
```

---

## 🔍 Query Optimization

### EXPLAIN Query

```sql
-- Basic explain
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';

-- Extended explain
EXPLAIN EXTENDED SELECT * FROM users WHERE email = 'user@example.com';
SHOW WARNINGS;

-- Analyze query
ANALYZE TABLE users;
```

### Index Management

```sql
-- Show indexes
SHOW INDEX FROM users;

-- Create index
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_name_email ON users(name, email);

-- Drop index
DROP INDEX idx_email ON users;

-- Check index usage
SELECT * FROM information_schema.statistics 
WHERE table_schema = 'database' AND table_name = 'users';
```

---

## 🐛 Troubleshooting Checklist

### Service Issues
- [ ] Check service status: `sudo systemctl status mariadb`
- [ ] Check error logs: `sudo tail -f /var/log/mysql/error.log`
- [ ] Check port availability: `sudo netstat -tlnp | grep 3306`
- [ ] Check disk space: `df -h /var/lib/mysql`
- [ ] Check permissions: `ls -la /var/lib/mysql`
- [ ] Check configuration: `mysqld --help --verbose`

### Connection Issues
- [ ] Service is running
- [ ] Socket file exists
- [ ] Correct credentials
- [ ] Firewall allows port 3306
- [ ] Bind address configured
- [ ] User has remote access

### Performance Issues
- [ ] Check running queries: `SHOW PROCESSLIST`
- [ ] Check slow queries: `tail /var/log/mysql/mysql-slow.log`
- [ ] Check connections: `SHOW STATUS LIKE 'Threads%'`
- [ ] Check buffer pool: `SHOW STATUS LIKE 'Innodb_buffer_pool%'`
- [ ] Run mysqltuner
- [ ] Check indexes: `SHOW INDEX FROM table`

---

## 🎓 Interview Questions

### Basic Level

**Q1: What is MariaDB?**
A: MariaDB is an open-source relational database management system (RDBMS) that is a fork of MySQL, designed to be highly compatible with MySQL while offering additional features.

**Q2: What is the default port for MariaDB?**
A: Port 3306

**Q3: Where is the MariaDB configuration file located?**
A: `/etc/mysql/my.cnf` or `/etc/mysql/mariadb.conf.d/50-server.cnf`

**Q4: How do you check if MariaDB is running?**
A: 
```bash
sudo systemctl status mariadb
ps aux | grep mysqld
```

**Q5: What is the difference between MyISAM and InnoDB?**
A:
- **InnoDB**: Supports transactions, foreign keys, row-level locking (default)
- **MyISAM**: No transactions, table-level locking, faster for read-heavy workloads

### Intermediate Level

**Q6: How do you troubleshoot "Can't connect to MySQL server" error?**
A:
1. Check if service is running
2. Verify socket file exists
3. Check bind-address in config
4. Verify firewall rules
5. Check user permissions

**Q7: How do you reset the root password?**
A:
```bash
sudo systemctl stop mariadb
sudo mysqld_safe --skip-grant-tables &
mysql -u root
UPDATE mysql.user SET password=PASSWORD('newpass') WHERE User='root';
FLUSH PRIVILEGES;
```

**Q8: What is the purpose of the slow query log?**
A: The slow query log records queries that take longer than `long_query_time` seconds to execute, helping identify performance bottlenecks.

**Q9: How do you optimize a table?**
A:
```sql
ANALYZE TABLE table_name;
OPTIMIZE TABLE table_name;
```

**Q10: What is innodb_buffer_pool_size?**
A: It's the size of the buffer pool where InnoDB caches table and index data. Should be set to 70-80% of available RAM for dedicated database servers.

### Advanced Level

**Q11: How do you handle database corruption?**
A:
1. Check error logs
2. Use mysqlcheck to identify corrupted tables
3. Repair with `mysqlcheck --repair` or `REPAIR TABLE`
4. For InnoDB, use `innodb_force_recovery`
5. Dump and restore if necessary

**Q12: Explain the different innodb_force_recovery levels.**
A:
- 1: Ignore corrupt pages
- 2: Prevent master thread operations
- 3: Don't run transaction rollbacks
- 4: Don't calculate statistics
- 5: Don't look at undo logs
- 6: Don't do transaction rollback on startup

**Q13: How do you set up MariaDB replication?**
A:
1. Configure master with server-id and log_bin
2. Create replication user
3. Get master status (file and position)
4. Configure slave with server-id
5. Use CHANGE MASTER TO command
6. Start slave

**Q14: What are the ACID properties?**
A:
- **Atomicity**: All or nothing transactions
- **Consistency**: Database remains in valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data persists

**Q15: How do you optimize MariaDB for high traffic?**
A:
- Increase innodb_buffer_pool_size
- Optimize max_connections
- Enable query cache (if applicable)
- Use connection pooling
- Optimize queries and add indexes
- Use read replicas
- Enable slow query log
- Regular maintenance (ANALYZE, OPTIMIZE)

---

## 📝 Pre-Exam Checklist

- [ ] Understand MariaDB architecture
- [ ] Know service management commands
- [ ] Can troubleshoot connection issues
- [ ] Understand configuration files
- [ ] Know backup and restore procedures
- [ ] Can diagnose performance issues
- [ ] Understand user management
- [ ] Know security best practices
- [ ] Can repair corrupted databases
- [ ] Understand replication basics
- [ ] Know monitoring tools
- [ ] Can optimize queries
- [ ] Understand storage engines
- [ ] Know log file locations

---

## 🔄 Common Workflows

### Initial Setup
```bash
# Install
sudo apt install mariadb-server

# Start and enable
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Secure installation
sudo mysql_secure_installation

# Test connection
mysql -u root -p
```

### Daily Maintenance
```bash
# Check status
sudo systemctl status mariadb

# Check logs
sudo tail -f /var/log/mysql/error.log

# Check slow queries
sudo mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log

# Check connections
mysql -u root -p -e "SHOW PROCESSLIST;"

# Backup
mysqldump -u root -p --all-databases > backup_$(date +%Y%m%d).sql
```

### Troubleshooting Workflow
```bash
# 1. Check service
sudo systemctl status mariadb

# 2. Check logs
sudo tail -50 /var/log/mysql/error.log

# 3. Check resources
df -h /var/lib/mysql
top -u mysql

# 4. Check connections
mysql -u root -p -e "SHOW PROCESSLIST;"

# 5. Check configuration
mysql -u root -p -e "SHOW VARIABLES;"

# 6. Run diagnostics
mysqlcheck -u root -p --all-databases
./mysqltuner.pl
```

---

## 💡 Pro Tips

1. **Always check logs first** - Error log contains most troubleshooting info
2. **Use systemctl** - Better than service command for modern systems
3. **Monitor slow queries** - Enable slow query log in production
4. **Regular backups** - Automate with cron jobs
5. **Test configuration** - Use `mysqld --help --verbose` to validate
6. **Use mysqltuner** - Get configuration recommendations
7. **Monitor connections** - Set up alerts for high connection count
8. **Optimize regularly** - Run ANALYZE and OPTIMIZE on large tables
9. **Use indexes wisely** - Too many indexes slow down writes
10. **Document changes** - Keep track of configuration modifications

---

## 📚 Quick Reference Links

- **Documentation**: https://mariadb.com/kb/en/documentation/
- **Troubleshooting**: https://mariadb.com/kb/en/troubleshooting-connection-issues/
- **MySQL Tuner**: https://github.com/major/MySQLTuner-perl
- **Stack Overflow**: https://stackoverflow.com/questions/tagged/mariadb

---

## ✅ Verification Commands

```bash
# Service running
sudo systemctl is-active mariadb

# Can connect
mysql -u root -p -e "SELECT 1;"

# No errors in log
sudo tail -20 /var/log/mysql/error.log | grep -i error

# Disk space OK
df -h /var/lib/mysql | awk 'NR==2 {print $5}' | sed 's/%//'

# Connections under limit
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
```

---

**Remember:**
- MariaDB is MySQL-compatible
- Default port: 3306
- Always check logs first
- Regular backups are critical
- Monitor performance metrics
- Security is paramount

**Last Updated:** 2024-01-16