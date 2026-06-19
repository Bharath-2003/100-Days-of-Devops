# Ansible Fundamentals for DevOps

## 🎯 Overview

Ansible is an open-source automation tool for configuration management, application deployment, and task automation. It uses a simple, agentless architecture and human-readable YAML syntax.

**Duration:** 4 weeks (Weeks 25-28)
**Time Investment:** 3-4 hours/day
**Difficulty:** Intermediate
**Prerequisites:** Linux, Shell scripting, SSH basics

---

## 📚 What You'll Learn

### Week 1: Ansible Basics
- Ansible architecture and concepts
- Installation and setup
- Inventory management
- Ad-hoc commands
- Basic modules

### Week 2: Playbooks
- YAML syntax
- Playbook structure
- Variables and facts
- Conditionals and loops
- Handlers and templates

### Week 3: Roles and Best Practices
- Role structure
- Ansible Galaxy
- Variables precedence
- Ansible Vault
- Error handling

### Week 4: Advanced Topics
- Dynamic inventory
- Custom modules
- Ansible Tower/AWX
- Testing with Molecule
- CI/CD integration

---

## 🎓 Important Concepts

### 1. Ansible Architecture

```
┌─────────────────────────────────────────┐
│         Control Node (Ansible)          │
│  ┌──────────────────────────────────┐  │
│  │  Inventory (hosts)               │  │
│  │  Playbooks (YAML)                │  │
│  │  Roles                           │  │
│  │  Modules                         │  │
│  └──────────────────────────────────┘  │
└──────────────┬──────────────────────────┘
               │ SSH (Agentless)
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌──▼────┐  ┌──▼────┐
│ Host1 │  │ Host2 │  │ Host3 │
│(Linux)│  │(Linux)│  │(Linux)│
└───────┘  └───────┘  └───────┘
```

**Key Components:**
- **Control Node:** Machine where Ansible is installed
- **Managed Nodes:** Servers managed by Ansible
- **Inventory:** List of managed nodes
- **Modules:** Units of code Ansible executes
- **Playbooks:** YAML files defining automation
- **Roles:** Reusable automation content

**Why Ansible?**
- ✅ Agentless (uses SSH)
- ✅ Simple YAML syntax
- ✅ Idempotent operations
- ✅ Large module library
- ✅ Strong community
- ✅ Easy to learn

---

### 2. Inventory Structure

```ini
# Static Inventory (INI format)
[webservers]
web1.example.com
web2.example.com
web3.example.com

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com

[production:children]
webservers
databases
loadbalancers

[webservers:vars]
http_port=80
max_clients=200

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

```yaml
# Static Inventory (YAML format)
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
        web3.example.com:
      vars:
        http_port: 80
        max_clients: 200
    
    databases:
      hosts:
        db1.example.com:
        db2.example.com:
      vars:
        db_port: 5432
    
    loadbalancers:
      hosts:
        lb1.example.com:
  
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

**Inventory Patterns:**
```bash
# All hosts
ansible all -m ping

# Specific group
ansible webservers -m ping

# Multiple groups
ansible webservers:databases -m ping

# Exclude group
ansible all:!databases -m ping

# Intersection
ansible webservers:&production -m ping

# Specific host
ansible web1.example.com -m ping

# Range
ansible web[1:3].example.com -m ping
```

---

### 3. Playbook Structure

```yaml
---
# Playbook: web_setup.yml
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    max_clients: 200
  
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes
    
    - name: Start Apache service
      service:
        name: apache2
        state: started
        enabled: yes
    
    - name: Copy configuration file
      template:
        src: apache.conf.j2
        dest: /etc/apache2/apache2.conf
      notify: Restart Apache
  
  handlers:
    - name: Restart Apache
      service:
        name: apache2
        state: restarted
```

**Playbook Components:**
- **name:** Description of play/task
- **hosts:** Target hosts/groups
- **become:** Run as sudo
- **vars:** Variables
- **tasks:** List of actions
- **handlers:** Triggered by notify
- **roles:** Reusable content

---

### 4. Common Modules

```yaml
# Package Management
- name: Install package (apt)
  apt:
    name: nginx
    state: present

- name: Install package (yum)
  yum:
    name: nginx
    state: present

# Service Management
- name: Start service
  service:
    name: nginx
    state: started
    enabled: yes

# File Operations
- name: Create directory
  file:
    path: /opt/app
    state: directory
    mode: '0755'

- name: Copy file
  copy:
    src: app.conf
    dest: /etc/app/app.conf
    owner: root
    group: root
    mode: '0644'

- name: Template file
  template:
    src: config.j2
    dest: /etc/app/config.yml

# Command Execution
- name: Run command
  command: /usr/bin/some-command

- name: Run shell command
  shell: echo $HOME

# User Management
- name: Create user
  user:
    name: appuser
    state: present
    groups: wheel
    shell: /bin/bash

# Git Operations
- name: Clone repository
  git:
    repo: https://github.com/user/repo.git
    dest: /opt/app
    version: main

# Database
- name: Create database
  postgresql_db:
    name: mydb
    state: present

# Docker
- name: Run container
  docker_container:
    name: webapp
    image: nginx:latest
    state: started
    ports:
      - "80:80"
```

---

### 5. Variables and Facts

```yaml
# Variable Definition
vars:
  http_port: 80
  app_name: myapp

# Variable Files
vars_files:
  - vars/common.yml
  - vars/{{ env }}.yml

# Host Variables
host_vars/web1.example.com:
  http_port: 8080

# Group Variables
group_vars/webservers:
  http_port: 80

# Facts (Gathered automatically)
- name: Display facts
  debug:
    msg: "OS is {{ ansible_distribution }}"

# Custom Facts
- name: Set custom fact
  set_fact:
    deployment_time: "{{ ansible_date_time.iso8601 }}"

# Register Variables
- name: Check service status
  command: systemctl status nginx
  register: nginx_status

- name: Display status
  debug:
    var: nginx_status.stdout
```

**Variable Precedence (Lowest to Highest):**
1. Role defaults
2. Inventory file/script group vars
3. Inventory group_vars/all
4. Playbook group_vars/all
5. Inventory group_vars/*
6. Playbook group_vars/*
7. Inventory file/script host vars
8. Inventory host_vars/*
9. Playbook host_vars/*
10. Host facts
11. Play vars
12. Play vars_prompt
13. Play vars_files
14. Role vars
15. Block vars
16. Task vars
17. Extra vars (-e)

---

### 6. Conditionals and Loops

```yaml
# Conditionals
- name: Install Apache (Debian)
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"

- name: Install Apache (RedHat)
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"

# Multiple Conditions
- name: Complex condition
  debug:
    msg: "Condition met"
  when:
    - ansible_distribution == "Ubuntu"
    - ansible_distribution_version == "20.04"

# Loops
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - curl

# Loop with Dictionary
- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
    state: present
  loop:
    - { name: 'alice', groups: 'wheel' }
    - { name: 'bob', groups: 'users' }

# Loop with Conditionals
- name: Install packages conditionally
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - apache2
  when: ansible_os_family == "Debian"
```

---

### 7. Roles Structure

```
roles/
└── webserver/
    ├── defaults/
    │   └── main.yml          # Default variables
    ├── files/
    │   └── index.html        # Static files
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── meta/
    │   └── main.yml          # Role metadata
    ├── tasks/
    │   └── main.yml          # Main tasks
    ├── templates/
    │   └── nginx.conf.j2     # Jinja2 templates
    ├── tests/
    │   ├── inventory         # Test inventory
    │   └── test.yml          # Test playbook
    └── vars/
        └── main.yml          # Role variables
```

**Using Roles:**
```yaml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - common
    - webserver
    - monitoring

# With Variables
- name: Configure web servers
  hosts: webservers
  roles:
    - role: webserver
      vars:
        http_port: 8080
```

---

## 💻 Hands-On Exercises

### Exercise 1: Installation and Setup (30 minutes)

**Objective:** Install Ansible and configure basic setup

```bash
# 1. Install Ansible (Ubuntu/Debian)
sudo apt update
sudo apt install -y ansible

# 2. Install Ansible (RHEL/CentOS)
sudo yum install -y epel-release
sudo yum install -y ansible

# 3. Install Ansible (pip)
pip3 install ansible

# 4. Verify installation
ansible --version

# 5. Create project directory
mkdir ~/ansible-lab
cd ~/ansible-lab

# 6. Create inventory file
cat > inventory << 'EOF'
[local]
localhost ansible_connection=local

[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
EOF

# 7. Test connectivity
ansible all -i inventory -m ping

# 8. Create ansible.cfg
cat > ansible.cfg << 'EOF'
[defaults]
inventory = ./inventory
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
EOF

# 9. Test with config
ansible all -m ping

# 10. Gather facts
ansible all -m setup
```

**Challenge:**
Set up Ansible to manage 3 VMs:
- 2 web servers
- 1 database server
- Configure SSH key authentication
- Test connectivity

---

### Exercise 2: Ad-hoc Commands (45 minutes)

**Objective:** Master Ansible ad-hoc commands

```bash
# 1. Ping all hosts
ansible all -m ping

# 2. Check disk space
ansible all -m shell -a "df -h"

# 3. Check memory
ansible all -m shell -a "free -h"

# 4. Install package
ansible webservers -m apt -a "name=nginx state=present" --become

# 5. Start service
ansible webservers -m service -a "name=nginx state=started enabled=yes" --become

# 6. Copy file
ansible webservers -m copy -a "src=index.html dest=/var/www/html/index.html" --become

# 7. Create directory
ansible all -m file -a "path=/opt/app state=directory mode=0755" --become

# 8. Create user
ansible all -m user -a "name=appuser state=present groups=sudo" --become

# 9. Gather specific facts
ansible all -m setup -a "filter=ansible_distribution*"

# 10. Run command
ansible all -m command -a "uptime"

# 11. Check service status
ansible webservers -m service -a "name=nginx" --become

# 12. Reboot servers
ansible all -m reboot --become

# 13. Get file content
ansible all -m slurp -a "src=/etc/hostname"

# 14. Synchronize files
ansible webservers -m synchronize -a "src=./app dest=/opt/app"

# 15. Manage cron jobs
ansible all -m cron -a "name='backup' minute='0' hour='2' job='/usr/local/bin/backup.sh'" --become
```

**Challenge:**
Using ad-hoc commands:
1. Install Docker on all servers
2. Create a Docker network
3. Deploy a container
4. Verify it's running
5. Get container logs

**Solution:**
```bash
ansible all -m apt -a "name=docker.io state=present" --become
ansible all -m service -a "name=docker state=started enabled=yes" --become
ansible all -m docker_network -a "name=app-network" --become
ansible all -m docker_container -a "name=webapp image=nginx:latest state=started networks=app-network ports=80:80" --become
ansible all -m command -a "docker ps"
ansible all -m command -a "docker logs webapp"
```

---

### Exercise 3: First Playbook (60 minutes)

**Objective:** Create and run your first playbook

```yaml
# playbook.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Create web directory
      file:
        path: /var/www/myapp
        state: directory
        mode: '0755'
    
    - name: Copy index.html
      copy:
        content: |
          <html>
          <head><title>Ansible Demo</title></head>
          <body><h1>Hello from Ansible!</h1></body>
          </html>
        dest: /var/www/myapp/index.html
        mode: '0644'
    
    - name: Configure Nginx site
      copy:
        content: |
          server {
              listen 80;
              server_name _;
              root /var/www/myapp;
              index index.html;
          }
        dest: /etc/nginx/sites-available/myapp
      notify: Reload Nginx
    
    - name: Enable site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: Reload Nginx
  
  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

```bash
# Run playbook
ansible-playbook playbook.yml

# Check syntax
ansible-playbook playbook.yml --syntax-check

# Dry run
ansible-playbook playbook.yml --check

# Verbose output
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv

# Limit to specific hosts
ansible-playbook playbook.yml --limit web1

# Start at specific task
ansible-playbook playbook.yml --start-at-task="Install Nginx"

# Use tags
ansible-playbook playbook.yml --tags "install"
ansible-playbook playbook.yml --skip-tags "config"
```

**Challenge:**
Create a playbook that:
1. Installs MySQL
2. Creates a database
3. Creates a user with privileges
4. Configures firewall
5. Backs up configuration

**Solution:**
```yaml
---
- name: Setup MySQL
  hosts: databases
  become: yes
  
  vars:
    mysql_root_password: "SecurePass123"
    db_name: "myapp"
    db_user: "appuser"
    db_password: "AppPass123"
  
  tasks:
    - name: Install MySQL
      apt:
        name:
          - mysql-server
          - python3-pymysql
        state: present
    
    - name: Start MySQL
      service:
        name: mysql
        state: started
        enabled: yes
    
    - name: Set root password
      mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        login_unix_socket: /var/run/mysqld/mysqld.sock
    
    - name: Create database
      mysql_db:
        name: "{{ db_name }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
    
    - name: Create user
      mysql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"
        priv: "{{ db_name }}.*:ALL"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
    
    - name: Configure firewall
      ufw:
        rule: allow
        port: '3306'
        proto: tcp
    
    - name: Backup MySQL config
      copy:
        src: /etc/mysql/mysql.conf.d/mysqld.cnf
        dest: /root/mysqld.cnf.backup
        remote_src: yes
```

---

### Exercise 4: Variables and Templates (60 minutes)

**Objective:** Work with variables and Jinja2 templates

```yaml
# playbook-vars.yml
---
- name: Deploy application
  hosts: webservers
  become: yes
  
  vars:
    app_name: myapp
    app_port: 8080
    app_user: appuser
    app_dir: "/opt/{{ app_name }}"
  
  vars_files:
    - vars/common.yml
    - vars/{{ env }}.yml
  
  tasks:
    - name: Create app user
      user:
        name: "{{ app_user }}"
        state: present
        system: yes
    
    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
    
    - name: Deploy configuration
      template:
        src: templates/app.conf.j2
        dest: "{{ app_dir }}/app.conf"
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0644'
      notify: Restart application
    
    - name: Display variables
      debug:
        msg: |
          App: {{ app_name }}
          Port: {{ app_port }}
          User: {{ app_user }}
          Dir: {{ app_dir }}
          Env: {{ env }}
  
  handlers:
    - name: Restart application
      service:
        name: "{{ app_name }}"
        state: restarted
```

```jinja2
{# templates/app.conf.j2 #}
# Application Configuration
# Generated by Ansible on {{ ansible_date_time.iso8601 }}

[application]
name = {{ app_name }}
port = {{ app_port }}
user = {{ app_user }}
directory = {{ app_dir }}
environment = {{ env }}

[server]
host = {{ ansible_default_ipv4.address }}
hostname = {{ ansible_hostname }}
os = {{ ansible_distribution }} {{ ansible_distribution_version }}

[database]
host = {{ db_host }}
port = {{ db_port }}
name = {{ db_name }}
user = {{ db_user }}

{% if env == 'production' %}
[production]
debug = false
log_level = warning
{% else %}
[development]
debug = true
log_level = debug
{% endif %}

[servers]
{% for host in groups['webservers'] %}
server{{ loop.index }} = {{ hostvars[host]['ansible_default_ipv4']['address'] }}
{% endfor %}
```

```yaml
# vars/common.yml
---
db_port: 5432
log_dir: /var/log/myapp

# vars/dev.yml
---
env: development
db_host: dev-db.example.com
db_name: myapp_dev
db_user: dev_user

# vars/prod.yml
---
env: production
db_host: prod-db.example.com
db_name: myapp_prod
db_user: prod_user
```

```bash
# Run with different environments
ansible-playbook playbook-vars.yml -e "env=dev"
ansible-playbook playbook-vars.yml -e "env=prod"
```

**Challenge:**
Create a playbook with templates for:
1. Nginx virtual host configuration
2. Application systemd service file
3. Monitoring configuration
4. Backup script

---

### Exercise 5: Roles (90 minutes)

**Objective:** Create and use Ansible roles

```bash
# 1. Create role structure
ansible-galaxy init webserver

# 2. Role structure created:
webserver/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml
```

```yaml
# roles/webserver/defaults/main.yml
---
http_port: 80
max_clients: 200
server_name: localhost

# roles/webserver/vars/main.yml
---
nginx_user: www-data
nginx_group: www-data

# roles/webserver/tasks/main.yml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Create web directory
  file:
    path: /var/www/{{ server_name }}
    state: directory
    owner: "{{ nginx_user }}"
    group: "{{ nginx_group }}"
    mode: '0755'

- name: Deploy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/{{ server_name }}
  notify: Reload Nginx

- name: Enable site
  file:
    src: /etc/nginx/sites-available/{{ server_name }}
    dest: /etc/nginx/sites-enabled/{{ server_name }}
    state: link
  notify: Reload Nginx

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes

# roles/webserver/handlers/main.yml
---
- name: Reload Nginx
  service:
    name: nginx
    state: reloaded

# roles/webserver/templates/nginx.conf.j2
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    root /var/www/{{ server_name }};
    index index.html;
    
    client_max_body_size 10M;
    
    location / {
        try_files $uri $uri/ =404;
    }
}

# roles/webserver/meta/main.yml
---
dependencies:
  - role: common
  - role: firewall
```

**Using the Role:**
```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - role: webserver
      vars:
        http_port: 8080
        server_name: myapp.example.com
```

**Challenge:**
Create roles for:
1. Common (base configuration)
2. Security (firewall, fail2ban)
3. Monitoring (node_exporter)
4. Application (your app)

**Solution Structure:**
```
roles/
├── common/
│   ├── tasks/
│   │   └── main.yml (update system, install basics)
│   └── handlers/
│       └── main.yml
├── security/
│   ├── tasks/
│   │   └── main.yml (firewall, fail2ban, ssh hardening)
│   └── templates/
│       └── sshd_config.j2
├── monitoring/
│   ├── tasks/
│   │   └── main.yml (install node_exporter)
│   └── files/
│       └── node_exporter.service
└── application/
    ├── tasks/
    │   └── main.yml (deploy app)
    ├── templates/
    │   └── app.service.j2
    └── handlers/
        └── main.yml
```

---

### Exercise 6: Ansible Vault (30 minutes)

**Objective:** Secure sensitive data

```bash
# 1. Create encrypted file
ansible-vault create secrets.yml

# Enter vault password
# Add content:
---
db_password: SuperSecretPass123
api_key: abc123xyz789

# 2. Edit encrypted file
ansible-vault edit secrets.yml

# 3. View encrypted file
ansible-vault view secrets.yml

# 4. Encrypt existing file
ansible-vault encrypt vars/passwords.yml

# 5. Decrypt file
ansible-vault decrypt vars/passwords.yml

# 6. Change vault password
ansible-vault rekey secrets.yml

# 7. Use in playbook
---
- name: Deploy with secrets
  hosts: all
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Create database user
      mysql_user:
        name: appuser
        password: "{{ db_password }}"
        state: present

# 8. Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass

# 9. Use password file
echo "myVaultPassword" > .vault_pass
chmod 600 .vault_pass
ansible-playbook playbook.yml --vault-password-file .vault_pass

# 10. Configure in ansible.cfg
[defaults]
vault_password_file = ./.vault_pass
```

**Challenge:**
Create a secure deployment playbook:
1. Store all passwords in vault
2. Store API keys in vault
3. Store SSH keys in vault
4. Deploy application with secrets

---

### Exercise 7: Dynamic Inventory (45 minutes)

**Objective:** Use dynamic inventory sources

```python
#!/usr/bin/env python3
# inventory.py - Dynamic inventory script

import json
import sys

def get_inventory():
    inventory = {
        'webservers': {
            'hosts': ['web1.example.com', 'web2.example.com'],
            'vars': {
                'http_port': 80,
                'ansible_user': 'ubuntu'
            }
        },
        'databases': {
            'hosts': ['db1.example.com'],
            'vars': {
                'db_port': 5432
            }
        },
        '_meta': {
            'hostvars': {
                'web1.example.com': {
                    'ansible_host': '192.168.1.10'
                },
                'web2.example.com': {
                    'ansible_host': '192.168.1.11'
                },
                'db1.example.com': {
                    'ansible_host': '192.168.1.20'
                }
            }
        }
    }
    return inventory

def get_host(host):
    return {}

if __name__ == '__main__':
    if len(sys.argv) == 2 and sys.argv[1] == '--list':
        print(json.dumps(get_inventory(), indent=2))
    elif len(sys.argv) == 3 and sys.argv[1] == '--host':
        print(json.dumps(get_host(sys.argv[2]), indent=2))
    else:
        print("Usage: inventory.py --list or inventory.py --host <hostname>")
        sys.exit(1)
```

```bash
# Make executable
chmod +x inventory.py

# Test
./inventory.py --list

# Use with Ansible
ansible all -i inventory.py -m ping

# AWS EC2 Dynamic Inventory
# Install boto3
pip3 install boto3

# Create aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
  - us-west-2
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
hostnames:
  - tag:Name

# Use
ansible-inventory -i aws_ec2.yml --list
ansible all -i aws_ec2.yml -m ping
```

---

## 🎯 Practice Projects

### Project 1: LAMP Stack Deployment
Deploy complete LAMP stack:
- Apache web server
- MySQL database
- PHP application
- SSL certificates
- Monitoring
- Backups

### Project 2: Kubernetes Cluster Setup
Use Ansible to:
- Install Docker
- Setup Kubernetes master
- Join worker nodes
- Deploy CNI plugin
- Deploy applications

### Project 3: CI/CD Pipeline
Create Ansible playbooks for:
- Build application
- Run tests
- Deploy to staging
- Run smoke tests
- Deploy to production
- Rollback capability

### Project 4: Infrastructure as Code
Manage complete infrastructure:
- Network configuration
- Security groups
- Load balancers
- Auto-scaling groups
- Monitoring
- Logging

---

## 📝 Best Practices

### 1. Playbook Organization
```
project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   └── staging/
│       ├── hosts
│       └── group_vars/
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
└── group_vars/
    └── all.yml
```

### 2. Idempotency
```yaml
# ✓ Idempotent
- name: Ensure Nginx is installed
  apt:
    name: nginx
    state: present

# ✗ Not idempotent
- name: Install Nginx
  shell: apt-get install nginx
```

### 3. Error Handling
```yaml
- name: Task with error handling
  command: /usr/bin/risky-command
  register: result
  failed_when: result.rc != 0 and result.rc != 2
  ignore_errors: yes

- name: Cleanup on failure
  file:
    path: /tmp/temp-file
    state: absent
  when: result is failed
```

### 4. Testing
```yaml
# Use --check mode
ansible-playbook playbook.yml --check

# Use --diff
ansible-playbook playbook.yml --check --diff

# Use assert
- name: Verify service is running
  assert:
    that:
      - nginx_status.stdout.find('active (running)') != -1
    fail_msg: "Nginx is not running"
    success_msg: "Nginx is running"
```

---

## ✅ Completion Checklist

### Week 1
- [ ] Install and configure Ansible
- [ ] Create inventory files
- [ ] Master ad-hoc commands
- [ ] Understand modules
- [ ] Test connectivity

### Week 2
- [ ] Write playbooks
- [ ] Use variables and facts
- [ ] Create templates
- [ ] Implement handlers
- [ ] Use conditionals and loops

### Week 3
- [ ] Create roles
- [ ] Use Ansible Galaxy
- [ ] Implement Ansible Vault
- [ ] Understand variable precedence
- [ ] Error handling

### Week 4
- [ ] Dynamic inventory
- [ ] Testing with Molecule
- [ ] CI/CD integration
- [ ] Complete projects
- [ ] Production ready

---

**Next:** [AWS →](../aws/README.md)

**Last Updated:** June 19, 2026