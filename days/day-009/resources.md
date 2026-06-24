# Day 9: MariaDB Service Troubleshooting - Resources

## 📚 Official Documentation

### MariaDB Documentation
- **Official Documentation**: https://mariadb.com/kb/en/documentation/
- **Installation Guide**: https://mariadb.com/kb/en/getting-installing-and-upgrading-mariadb/
- **Server System Variables**: https://mariadb.com/kb/en/server-system-variables/
- **Troubleshooting**: https://mariadb.com/kb/en/troubleshooting-connection-issues/
- **Performance Schema**: https://mariadb.com/kb/en/performance-schema/
- **InnoDB Storage Engine**: https://mariadb.com/kb/en/innodb/

### MySQL Documentation (Compatible)
- **MySQL Reference Manual**: https://dev.mysql.com/doc/refman/8.0/en/
- **MySQL Troubleshooting**: https://dev.mysql.com/doc/refman/8.0/en/problems.html
- **MySQL Performance Schema**: https://dev.mysql.com/doc/refman/8.0/en/performance-schema.html

---

## 🛠️ Tools and Utilities

### Command-Line Tools

#### mysqltuner
```bash
# Download and run
wget http://mysqltuner.pl/ -O mysqltuner.pl
chmod +x mysqltuner.pl
./mysqltuner.pl

# GitHub Repository
https://github.com/major/MySQLTuner-perl
```

#### mytop
```bash
# Install
sudo apt install mytop

# Usage
mytop -u root -p password

# Documentation
https://github.com/jzawodn/mytop
```

#### innotop
```bash
# Install
sudo apt install innotop

# Usage
innotop -u root -p password

# Documentation
https://github.com/innotop/innotop
```

#### Percona Toolkit
```bash
# Install
sudo apt install percona-toolkit

# Useful tools:
pt-query-digest    # Analyze slow query logs
pt-online-schema-change  # Schema changes without downtime
pt-table-checksum  # Verify replication integrity
pt-duplicate-key-checker  # Find duplicate indexes

# Documentation
https://www.percona.com/doc/percona-toolkit/
```

#### mysqlcheck
```bash
# Check all databases
mysqlcheck -u root -p --all-databases

# Repair all databases
mysqlcheck -u root -p --auto-repair --all-databases

# Optimize all databases
mysqlcheck -u root -p --optimize --all-databases
```

### GUI Tools

#### phpMyAdmin
```bash
# Install
sudo apt install phpmyadmin

# Access: http://localhost/phpmyadmin

# Documentation
https://www.phpmyadmin.net/docs/
```

#### MySQL Workbench
```bash
# Download
https://dev.mysql.com/downloads/workbench/

# Features:
- Visual database design
- SQL development
- Server administration
- Performance monitoring
```

#### DBeaver
```bash
# Download
https://dbeaver.io/download/

# Features:
- Universal database tool
- SQL editor
- ER diagrams
- Data export/import
```

#### HeidiSQL (Windows)
```bash
# Download
https://www.heidisql.com/download.php

# Features:
- Lightweight SQL client
- Multiple database support
- Query builder
```

---

## 📊 Monitoring Solutions

### Prometheus + Grafana

#### MySQL Exporter
```bash
# Install
docker run -d \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(localhost:3306)/" \
  prom/mysqld-exporter

# Prometheus configuration
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']

# Grafana Dashboard
# Import Dashboard ID: 7362 (MySQL Overview)
# Import Dashboard ID: 6239 (MySQL InnoDB Metrics)
```

### Zabbix
```bash
# Template
https://www.zabbix.com/integrations/mysql

# Features:
- Pre-built templates
- Automatic discovery
- Alerting
- Historical data
```

### Nagios
```bash
# Plugin
https://exchange.nagios.org/directory/Plugins/Databases/MySQL

# check_mysql plugin
./check_mysql -H localhost -u nagios -p password
```

### PMM (Percona Monitoring and Management)
```bash
# Install PMM Server
docker run -d \
  -p 80:80 \
  --name pmm-server \
  percona/pmm-server:2

# Install PMM Client
sudo apt install pmm2-client

# Add MySQL service
pmm-admin add mysql --username=root --password=password

# Access: http://localhost

# Documentation
https://www.percona.com/software/database-tools/percona-monitoring-and-management
```

---

## 📖 Learning Resources

### Online Courses

#### Udemy
- **MySQL Database Administration**: https://www.udemy.com/course/mysql-database-administration/
- **MariaDB for Beginners**: https://www.udemy.com/course/mariadb-for-beginners/
- **MySQL Performance Tuning**: https://www.udemy.com/course/mysql-performance-tuning/

#### Coursera
- **Database Management Essentials**: https://www.coursera.org/learn/database-management
- **MySQL for Data Analytics**: https://www.coursera.org/learn/mysql-data-analytics

#### LinkedIn Learning
- **MySQL Essential Training**: https://www.linkedin.com/learning/mysql-essential-training
- **Database Administration Foundations**: https://www.linkedin.com/learning/database-administration-foundations

#### Pluralsight
- **MySQL Fundamentals**: https://www.pluralsight.com/courses/mysql-fundamentals-part1
- **MySQL Performance Tuning**: https://www.pluralsight.com/courses/mysql-performance-tuning

### YouTube Channels

#### TechWorld with Nana
- Database tutorials and DevOps integration
- https://www.youtube.com/@TechWorldwithNana

#### Traversy Media
- MySQL crash courses and tutorials
- https://www.youtube.com/@TraversyMedia

#### freeCodeCamp.org
- Full MySQL courses
- https://www.youtube.com/@freecodecamp

#### ProgrammingKnowledge
- MySQL administration tutorials
- https://www.youtube.com/@ProgrammingKnowledge

---

## 📚 Books

### Beginner to Intermediate

1. **"Learning MySQL" by Seyed M.M. Tahaghoghi**
   - Comprehensive introduction
   - Practical examples
   - ISBN: 978-0596008642

2. **"MySQL Crash Course" by Ben Forta**
   - Quick learning guide
   - Hands-on approach
   - ISBN: 978-0672327124

3. **"MySQL Cookbook" by Paul DuBois**
   - Problem-solution format
   - Real-world scenarios
   - ISBN: 978-1449374020

### Advanced

4. **"High Performance MySQL" by Baron Schwartz**
   - Performance optimization
   - Scaling strategies
   - ISBN: 978-1449314286

5. **"MySQL Troubleshooting" by Sveta Smirnova**
   - Troubleshooting techniques
   - Real-world problems
   - ISBN: 978-1449312008

6. **"Effective MySQL" Series by Ronald Bradford**
   - Replication techniques
   - Backup and recovery
   - Performance optimization

---

## 🎯 Cheat Sheets and Quick References

### Command Cheat Sheets

#### Service Management
```bash
# Start/Stop/Restart
sudo systemctl start mariadb
sudo systemctl stop mariadb
sudo systemctl restart mariadb
sudo systemctl status mariadb

# Enable/Disable
sudo systemctl enable mariadb
sudo systemctl disable mariadb
```

#### Connection Commands
```bash
# Local connection
mysql -u root -p

# Remote connection
mysql -h hostname -u username -p database

# Execute command
mysql -u root -p -e "SHOW DATABASES;"

# Execute script
mysql -u root -p database < script.sql
```

#### Backup and Restore
```bash
# Backup
mysqldump -u root -p database > backup.sql
mysqldump -u root -p --all-databases > all_backup.sql

# Restore
mysql -u root -p database < backup.sql
mysql -u root -p < all_backup.sql
```

### SQL Cheat Sheet

#### Database Operations
```sql
-- Create database
CREATE DATABASE dbname;

-- Drop database
DROP DATABASE dbname;

-- Use database
USE dbname;

-- Show databases
SHOW DATABASES;
```

#### Table Operations
```sql
-- Create table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Show tables
SHOW TABLES;

-- Describe table
DESCRIBE users;

-- Drop table
DROP TABLE users;
```

#### User Management
```sql
-- Create user
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';

-- Grant privileges
GRANT ALL PRIVILEGES ON database.* TO 'username'@'localhost';

-- Show grants
SHOW GRANTS FOR 'username'@'localhost';

-- Revoke privileges
REVOKE ALL PRIVILEGES ON database.* FROM 'username'@'localhost';

-- Drop user
DROP USER 'username'@'localhost';

-- Flush privileges
FLUSH PRIVILEGES;
```

### Troubleshooting Cheat Sheet

#### Check Service Status
```bash
sudo systemctl status mariadb
ps aux | grep mysqld
sudo netstat -tlnp | grep 3306
```

#### Check Logs
```bash
sudo tail -f /var/log/mysql/error.log
sudo journalctl -u mariadb -f
```

#### Check Configuration
```bash
mysql -u root -p -e "SHOW VARIABLES;"
mysql -u root -p -e "SHOW STATUS;"
```

#### Performance Checks
```sql
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS\G
SHOW STATUS LIKE 'Threads%';
SHOW STATUS LIKE 'Slow_queries';
```

---

## 🎓 Certifications

### Oracle MySQL Certifications

#### MySQL Database Administrator (OCA)
- **Exam**: 1Z0-888
- **Topics**: Installation, configuration, security, backup
- **Link**: https://education.oracle.com/mysql-database-administration

#### MySQL Developer (OCP)
- **Exam**: 1Z0-909
- **Topics**: SQL, stored procedures, optimization
- **Link**: https://education.oracle.com/mysql-developer

### MariaDB Certifications

#### MariaDB Certified Database Administrator
- **Provider**: MariaDB Corporation
- **Topics**: Installation, configuration, troubleshooting
- **Link**: https://mariadb.com/kb/en/mariadb-certification/

---

## 💻 Practice Labs and Sandboxes

### Online SQL Editors

#### SQLFiddle
- **URL**: http://sqlfiddle.com/
- **Features**: Test SQL queries online
- **Supports**: MySQL, PostgreSQL, SQLite

#### DB Fiddle
- **URL**: https://www.db-fiddle.com/
- **Features**: Modern SQL playground
- **Supports**: Multiple databases

#### SQL Online IDE
- **URL**: https://sqliteonline.com/
- **Features**: Quick SQL testing
- **Supports**: MySQL, PostgreSQL, SQLite

### Local Practice Environments

#### Docker Setup
```bash
# Run MariaDB in Docker
docker run -d \
  --name mariadb-test \
  -e MYSQL_ROOT_PASSWORD=password \
  -p 3306:3306 \
  mariadb:latest

# Connect
mysql -h 127.0.0.1 -u root -p

# Stop and remove
docker stop mariadb-test
docker rm mariadb-test
```

#### Vagrant Setup
```ruby
# Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.network "forwarded_port", guest: 3306, host: 3306
  
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y mariadb-server
    systemctl start mariadb
  SHELL
end
```

---

## 🔧 Configuration Examples

### Production Configuration Template

```ini
# /etc/mysql/mariadb.conf.d/50-server.cnf

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
max_connections = 500
connect_timeout = 10
wait_timeout = 600
max_allowed_packet = 64M
thread_cache_size = 16

# Logging
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 2
log_queries_not_using_indexes = 1

# Binary Logging
log_bin = /var/log/mysql/mysql-bin.log
expire_logs_days = 7
max_binlog_size = 100M
binlog_format = ROW

# InnoDB Settings
innodb_buffer_pool_size = 4G
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
innodb_buffer_pool_instances = 4

# Query Cache (deprecated in newer versions)
query_cache_type = 0
query_cache_size = 0

# Character Set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# Performance Schema
performance_schema = ON
```

### Replication Configuration

#### Master Configuration
```ini
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
binlog_do_db = production_db
```

#### Slave Configuration
```ini
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin
log_bin = /var/log/mysql/mysql-bin.log
read_only = 1
```

---

## 🐛 Common Error Codes Reference

### Connection Errors
- **1045**: Access denied for user
- **2002**: Can't connect through socket
- **2003**: Can't connect to MySQL server
- **2006**: MySQL server has gone away
- **2013**: Lost connection during query

### Table Errors
- **1005**: Can't create table
- **1016**: Can't open file
- **1034**: Index is corrupt
- **1146**: Table doesn't exist
- **1054**: Unknown column

### Data Errors
- **1062**: Duplicate entry for key
- **1064**: SQL syntax error
- **1136**: Column count doesn't match
- **1406**: Data too long for column
- **1451**: Cannot delete or update (foreign key constraint)

### Resource Errors
- **1040**: Too many connections
- **1041**: Out of memory
- **1114**: Table is full
- **1205**: Lock wait timeout exceeded

---

## 🌐 Community Resources

### Forums and Discussion

#### Stack Overflow
- **Tag**: [mysql]
- **URL**: https://stackoverflow.com/questions/tagged/mysql
- **Tag**: [mariadb]
- **URL**: https://stackoverflow.com/questions/tagged/mariadb

#### Database Administrators Stack Exchange
- **URL**: https://dba.stackexchange.com/
- **Focus**: Professional database administration

#### MariaDB Community
- **Forum**: https://mariadb.com/kb/en/community/
- **Mailing Lists**: https://mariadb.com/kb/en/mailing-lists/

#### Reddit
- **r/mysql**: https://www.reddit.com/r/mysql/
- **r/mariadb**: https://www.reddit.com/r/mariadb/
- **r/database**: https://www.reddit.com/r/Database/

### Blogs and Articles

#### Percona Blog
- **URL**: https://www.percona.com/blog/
- **Topics**: Performance, optimization, best practices

#### MySQL Server Blog
- **URL**: https://mysqlserverteam.com/
- **Topics**: New features, updates, tips

#### Planet MySQL
- **URL**: https://planet.mysql.com/
- **Topics**: Aggregated MySQL blogs

---

## 📊 Benchmarking Tools

### sysbench
```bash
# Install
sudo apt install sysbench

# Prepare test
sysbench /usr/share/sysbench/oltp_read_write.lua \
  --mysql-host=localhost \
  --mysql-user=root \
  --mysql-password=password \
  --mysql-db=testdb \
  --tables=10 \
  --table-size=10000 \
  prepare

# Run benchmark
sysbench /usr/share/sysbench/oltp_read_write.lua \
  --mysql-host=localhost \
  --mysql-user=root \
  --mysql-password=password \
  --mysql-db=testdb \
  --tables=10 \
  --table-size=10000 \
  --threads=4 \
  --time=60 \
  run

# Cleanup
sysbench /usr/share/sysbench/oltp_read_write.lua \
  --mysql-host=localhost \
  --mysql-user=root \
  --mysql-password=password \
  --mysql-db=testdb \
  --tables=10 \
  cleanup
```

### mysqlslap
```bash
# Simple benchmark
mysqlslap --user=root --password --auto-generate-sql

# Custom benchmark
mysqlslap --user=root --password \
  --concurrency=50 \
  --iterations=10 \
  --number-int-cols=5 \
  --number-char-cols=20 \
  --auto-generate-sql
```

---

## 🔐 Security Resources

### Security Best Practices Guide
- **MariaDB Security**: https://mariadb.com/kb/en/securing-mariadb/
- **MySQL Security**: https://dev.mysql.com/doc/refman/8.0/en/security.html

### Security Tools

#### mysql_secure_installation
```bash
sudo mysql_secure_installation
```

#### Audit Plugin
```sql
INSTALL SONAME 'server_audit';
SET GLOBAL server_audit_logging = ON;
SET GLOBAL server_audit_events = 'CONNECT,QUERY,TABLE';
```

---

## 📱 Mobile Apps

### Database Management Apps

#### SQLPro for MySQL (iOS)
- **Link**: https://apps.apple.com/app/sqlpro-for-mysql/id899174769

#### MySQL Client (Android)
- **Link**: https://play.google.com/store/apps/details?id=com.navicat.mysql

---

## 🎬 Video Tutorials

### Installation and Setup
- **MariaDB Installation on Ubuntu**: https://www.youtube.com/watch?v=0o0tSaVQfV4
- **MySQL Complete Tutorial**: https://www.youtube.com/watch?v=7S_tz1z_5bA

### Troubleshooting
- **MySQL Troubleshooting Tips**: https://www.youtube.com/watch?v=Cz3WcZLRaWc
- **Database Performance Tuning**: https://www.youtube.com/watch?v=qDNjKMJnrec

### Advanced Topics
- **MySQL Replication**: https://www.youtube.com/watch?v=Lj_jAFpjMYE
- **MySQL High Availability**: https://www.youtube.com/watch?v=3Zv-8ixJKKo

---

## 📝 Sample Projects and Scripts

### GitHub Repositories

#### Awesome MySQL
- **URL**: https://github.com/shlomi-noach/awesome-mysql
- **Description**: Curated list of MySQL resources

#### MySQL Utilities
- **URL**: https://github.com/mysql/mysql-utilities
- **Description**: Collection of command-line utilities

#### MariaDB Docker
- **URL**: https://github.com/MariaDB/mariadb-docker
- **Description**: Official MariaDB Docker images

#### MySQL Backup Scripts
- **URL**: https://github.com/wangbin579/mysql-backup-script
- **Description**: Automated backup scripts

---

## 🔄 Migration Resources

### MySQL to MariaDB Migration
- **Guide**: https://mariadb.com/kb/en/migrating-from-mysql-to-mariadb/
- **Compatibility**: https://mariadb.com/kb/en/mariadb-vs-mysql-compatibility/

### PostgreSQL to MySQL Migration
- **Tools**: pgloader, AWS DMS
- **Guide**: https://www.postgresql.org/docs/current/migration.html

---

## 📊 Performance Tuning Resources

### Optimization Guides
- **MySQL Performance Tuning**: https://dev.mysql.com/doc/refman/8.0/en/optimization.html
- **MariaDB Optimization**: https://mariadb.com/kb/en/optimization-and-tuning/

### Query Optimization
- **EXPLAIN Guide**: https://dev.mysql.com/doc/refman/8.0/en/explain.html
- **Index Optimization**: https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html

---

## 🎯 Interview Preparation

### Common Interview Questions
1. What is the difference between MyISAM and InnoDB?
2. How do you optimize a slow query?
3. Explain ACID properties
4. What is database normalization?
5. How do you set up replication?
6. What are indexes and when to use them?
7. How do you handle database backups?
8. Explain transaction isolation levels
9. What is query cache?
10. How do you troubleshoot connection issues?

### Practice Platforms
- **LeetCode Database**: https://leetcode.com/problemset/database/
- **HackerRank SQL**: https://www.hackerrank.com/domains/sql
- **SQLZoo**: https://sqlzoo.net/

---

## 📚 Additional Reading

### White Papers
- **MySQL High Availability**: https://www.mysql.com/why-mysql/white-papers/
- **MariaDB Enterprise**: https://mariadb.com/resources/white-papers/

### Case Studies
- **MySQL at Scale**: https://www.mysql.com/customers/
- **MariaDB Success Stories**: https://mariadb.com/resources/case-studies/

---

## 🔗 Quick Links Summary

| Resource Type | Link |
|--------------|------|
| Official Docs | https://mariadb.com/kb/en/documentation/ |
| MySQL Docs | https://dev.mysql.com/doc/ |
| Stack Overflow | https://stackoverflow.com/questions/tagged/mysql |
| GitHub | https://github.com/MariaDB |
| Percona Blog | https://www.percona.com/blog/ |
| MySQL Tuner | https://github.com/major/MySQLTuner-perl |
| PMM | https://www.percona.com/software/database-tools/percona-monitoring-and-management |

---

**Last Updated:** 2024-01-16
**Version:** 1.0