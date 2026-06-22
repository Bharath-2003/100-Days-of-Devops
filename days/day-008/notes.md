# Day 8: Install Ansible - Comprehensive Notes

## 📋 Task Overview

**Objective:** Install and configure Ansible for configuration management and automation

**Difficulty:** Beginner-Intermediate  
**Estimated Time:** 2-3 hours

---

## 🎯 What is Ansible?

Ansible is an open-source automation tool used for:
- **Configuration Management** - Manage system configurations
- **Application Deployment** - Deploy applications consistently
- **Task Automation** - Automate repetitive tasks
- **Orchestration** - Coordinate multiple systems

### Key Features

1. **Agentless** - No software needed on managed nodes
2. **Simple** - Uses YAML for configuration (Playbooks)
3. **Powerful** - Can manage thousands of nodes
4. **Idempotent** - Safe to run multiple times
5. **Extensible** - Large collection of modules

---

## 🏗️ Ansible Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Control Node                            │
│  ┌────────────────────────────────────────────────┐    │
│  │           Ansible Installation                  │    │
│  │  - ansible                                      │    │
│  │  - ansible-playbook                             │    │
│  │  - ansible-vault                                │    │
│  │  - ansible-galaxy                               │    │
│  └────────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────────┐    │
│  │           Inventory File                        │    │
│  │  - List of managed nodes                        │    │
│  │  - Groups and variables                         │    │
│  └────────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────────┐    │
│  │           Playbooks (YAML)                      │    │
│  │  - Tasks to execute                             │    │
│  │  - Roles and variables                          │    │
│  └────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
                           │
                           │ SSH Connection
                           ▼
┌─────────────────────────────────────────────────────────┐
│                  Managed Nodes                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Server 1   │  │   Server 2   │  │   Server 3   │ │
│  │  (Python)    │  │  (Python)    │  │  (Python)    │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## 📦 Installation Methods

### Method 1: Using Package Manager (Recommended)

#### Ubuntu/Debian
```bash
# Update package index
sudo apt update

# Install software-properties-common
sudo apt install software-properties-common -y

# Add Ansible PPA
sudo add-apt-repository --yes --update ppa:ansible/ansible

# Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version
```

#### CentOS/RHEL 8/9
```bash
# Enable EPEL repository
sudo dnf install epel-release -y

# Install Ansible
sudo dnf install ansible -y

# Verify installation
ansible --version
```

#### CentOS/RHEL 7
```bash
# Enable EPEL repository
sudo yum install epel-release -y

# Install Ansible
sudo yum install ansible -y

# Verify installation
ansible --version
```

#### Fedora
```bash
# Install Ansible
sudo dnf install ansible -y

# Verify installation
ansible --version
```

### Method 2: Using pip (Python Package Manager)

```bash
# Install pip if not already installed
sudo apt install python3-pip -y  # Ubuntu/Debian
sudo yum install python3-pip -y  # CentOS/RHEL

# Upgrade pip
python3 -m pip install --upgrade pip

# Install Ansible
python3 -m pip install ansible

# Install specific version
python3 -m pip install ansible==2.10.7

# Verify installation
ansible --version
```

### Method 3: Using pipx (Isolated Environment)

```bash
# Install pipx
python3 -m pip install --user pipx
python3 -m pipx ensurepath

# Install Ansible
pipx install ansible-core

# Install ansible with collections
pipx install ansible

# Verify installation
ansible --version
```

### Method 4: From Source (Advanced)

```bash
# Clone Ansible repository
git clone https://github.com/ansible/ansible.git
cd ansible

# Checkout stable branch
git checkout stable-2.14

# Install dependencies
pip install -r requirements.txt

# Setup environment
source ./hacking/env-setup

# Verify installation
ansible --version
```

---

## ⚙️ Post-Installation Configuration

### 1. Create Ansible Configuration File

**Global Configuration:**
```bash
# Create ansible.cfg in /etc/ansible/
sudo mkdir -p /etc/ansible
sudo nano /etc/ansible/ansible.cfg
```

**Project-Specific Configuration:**
```bash
# Create ansible.cfg in project directory
mkdir -p ~/ansible-project
cd ~/ansible-project
nano ansible.cfg
```

**Sample ansible.cfg:**
```ini
[defaults]
# Inventory file location
inventory = ./inventory

# Number of parallel processes
forks = 5

# SSH timeout
timeout = 10

# Disable host key checking (use with caution)
host_key_checking = False

# Log file location
log_path = ./ansible.log

# Roles path
roles_path = ./roles

# Retry files
retry_files_enabled = False

# Display skipped hosts
display_skipped_hosts = False

# Gathering facts
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
# SSH arguments
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
# Pipelining (faster execution)
pipelining = True
```

### 2. Create Inventory File

**Simple Inventory (INI format):**
```ini
# inventory or hosts file

# Ungrouped hosts
server1.example.com
192.168.1.10

# Web servers group
[webservers]
web1.example.com
web2.example.com
web3.example.com

# Database servers group
[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

# Application servers with variables
[appservers]
app1.example.com ansible_user=ubuntu ansible_port=2222
app2.example.com ansible_user=ubuntu ansible_port=2222

# Group of groups
[production:children]
webservers
databases
appservers

# Group variables
[webservers:vars]
http_port=80
max_clients=200

[databases:vars]
db_port=5432
```

**YAML Inventory:**
```yaml
# inventory.yml

all:
  hosts:
    server1.example.com:
    192.168.1.10:
  
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
          ansible_host: 192.168.1.20
        db2.example.com:
          ansible_host: 192.168.1.21
      vars:
        db_port: 5432
    
    appservers:
      hosts:
        app1.example.com:
          ansible_user: ubuntu
          ansible_port: 2222
        app2.example.com:
          ansible_user: ubuntu
          ansible_port: 2222
    
    production:
      children:
        webservers:
        databases:
        appservers:
```

### 3. Set Up SSH Keys

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "ansible@control-node"

# Copy public key to managed nodes
ssh-copy-id user@server1.example.com
ssh-copy-id user@server2.example.com
ssh-copy-id user@server3.example.com

# Test SSH connection
ssh user@server1.example.com

# Or use ansible to test
ansible all -m ping
```

---

## 🧪 Verification and Testing

### 1. Check Ansible Version

```bash
# Display Ansible version
ansible --version

# Expected output:
# ansible [core 2.14.0]
#   config file = /etc/ansible/ansible.cfg
#   configured module search path = ['/home/user/.ansible/plugins/modules']
#   ansible python module location = /usr/lib/python3/dist-packages/ansible
#   ansible collection location = /home/user/.ansible/collections
#   executable location = /usr/bin/ansible
#   python version = 3.10.6
```

### 2. Test Connectivity

```bash
# Ping all hosts
ansible all -m ping

# Ping specific group
ansible webservers -m ping

# Ping specific host
ansible server1.example.com -m ping

# Expected output:
# server1.example.com | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### 3. Run Ad-Hoc Commands

```bash
# Check uptime
ansible all -a "uptime"

# Check disk space
ansible all -a "df -h"

# Check memory
ansible all -a "free -m"

# Install package
ansible webservers -m apt -a "name=nginx state=present" --become

# Restart service
ansible webservers -m service -a "name=nginx state=restarted" --become

# Copy file
ansible all -m copy -a "src=/tmp/test.txt dest=/tmp/test.txt"

# Gather facts
ansible all -m setup
```

### 4. Create and Run Simple Playbook

**File: `test-playbook.yml`**
```yaml
---
- name: Test Playbook
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Print hostname
      debug:
        msg: "Hostname is {{ ansible_hostname }}"
    
    - name: Print OS distribution
      debug:
        msg: "OS is {{ ansible_distribution }} {{ ansible_distribution_version }}"
    
    - name: Create test file
      file:
        path: /tmp/ansible-test.txt
        state: touch
        mode: '0644'
    
    - name: Write content to file
      copy:
        content: "Ansible is working!\n"
        dest: /tmp/ansible-test.txt
```

**Run the playbook:**
```bash
ansible-playbook test-playbook.yml

# Run with verbose output
ansible-playbook test-playbook.yml -v
ansible-playbook test-playbook.yml -vv
ansible-playbook test-playbook.yml -vvv

# Dry run (check mode)
ansible-playbook test-playbook.yml --check

# Run on specific hosts
ansible-playbook test-playbook.yml --limit webservers
```

---

## 🔧 Common Ansible Commands

### Inventory Commands
```bash
# List all hosts
ansible all --list-hosts

# List hosts in group
ansible webservers --list-hosts

# Show inventory graph
ansible-inventory --graph

# Show inventory in JSON
ansible-inventory --list

# Show host variables
ansible-inventory --host server1.example.com
```

### Module Commands
```bash
# List all modules
ansible-doc -l

# Show module documentation
ansible-doc ping
ansible-doc copy
ansible-doc service

# Show module examples
ansible-doc -s copy
```

### Playbook Commands
```bash
# Run playbook
ansible-playbook playbook.yml

# Syntax check
ansible-playbook playbook.yml --syntax-check

# List tasks
ansible-playbook playbook.yml --list-tasks

# List tags
ansible-playbook playbook.yml --list-tags

# Run specific tags
ansible-playbook playbook.yml --tags "install,configure"

# Skip tags
ansible-playbook playbook.yml --skip-tags "deploy"

# Start at task
ansible-playbook playbook.yml --start-at-task "Install nginx"
```

### Vault Commands
```bash
# Create encrypted file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Encrypt existing file
ansible-vault encrypt vars.yml

# Decrypt file
ansible-vault decrypt vars.yml

# View encrypted file
ansible-vault view secrets.yml

# Change vault password
ansible-vault rekey secrets.yml

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

### Galaxy Commands
```bash
# Install role
ansible-galaxy install geerlingguy.nginx

# Install from requirements
ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove geerlingguy.nginx

# Search for roles
ansible-galaxy search nginx

# Create role structure
ansible-galaxy init my-role
```

---

## 📁 Directory Structure

### Recommended Project Structure

```
ansible-project/
├── ansible.cfg                 # Ansible configuration
├── inventory/                  # Inventory files
│   ├── production/
│   │   ├── hosts              # Production hosts
│   │   └── group_vars/        # Group variables
│   │       ├── all.yml
│   │       ├── webservers.yml
│   │       └── databases.yml
│   └── staging/
│       ├── hosts              # Staging hosts
│       └── group_vars/
├── playbooks/                 # Playbooks
│   ├── site.yml              # Main playbook
│   ├── webservers.yml
│   ├── databases.yml
│   └── deploy.yml
├── roles/                     # Custom roles
│   ├── common/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   ├── files/
│   │   ├── vars/
│   │   ├── defaults/
│   │   └── meta/
│   ├── nginx/
│   └── postgresql/
├── group_vars/               # Global group variables
│   ├── all.yml
│   └── production.yml
├── host_vars/                # Host-specific variables
│   └── server1.example.com.yml
├── library/                  # Custom modules
├── filter_plugins/           # Custom filters
├── files/                    # Static files
├── templates/                # Jinja2 templates
└── requirements.yml          # Role dependencies
```

---

## 🔐 Security Best Practices

### 1. SSH Key Management
```bash
# Use strong SSH keys
ssh-keygen -t ed25519 -a 100

# Use SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Disable password authentication
# In /etc/ssh/sshd_config:
# PasswordAuthentication no
```

### 2. Ansible Vault
```bash
# Store sensitive data in vault
ansible-vault create group_vars/all/vault.yml

# Use vault in playbooks
---
- name: Example with vault
  hosts: all
  vars_files:
    - group_vars/all/vault.yml
  tasks:
    - name: Use encrypted variable
      debug:
        msg: "{{ vault_api_key }}"
```

### 3. Privilege Escalation
```yaml
# In playbook
---
- name: Secure privilege escalation
  hosts: all
  become: yes
  become_method: sudo
  become_user: root
  
  tasks:
    - name: Install package
      apt:
        name: nginx
        state: present
```

---

## 🐛 Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Permission denied (publickey)"
```bash
# Solution: Check SSH key
ssh -vvv user@host

# Ensure key is added to agent
ssh-add -l
ssh-add ~/.ssh/id_ed25519

# Check authorized_keys on remote host
ssh user@host "cat ~/.ssh/authorized_keys"
```

#### Issue 2: "Host key verification failed"
```bash
# Solution 1: Remove old host key
ssh-keygen -R hostname

# Solution 2: Disable host key checking (not recommended for production)
# In ansible.cfg:
[defaults]
host_key_checking = False
```

#### Issue 3: "Module not found"
```bash
# Solution: Check Python path
ansible all -m setup -a "filter=ansible_python_version"

# Install required Python packages
pip install <package-name>
```

#### Issue 4: Slow playbook execution
```bash
# Solution: Enable pipelining
# In ansible.cfg:
[ssh_connection]
pipelining = True

# Increase forks
[defaults]
forks = 20
```

---

## 📊 Ansible Components

### 1. Modules
- **Command modules:** `command`, `shell`, `raw`
- **File modules:** `copy`, `file`, `template`, `lineinfile`
- **Package modules:** `apt`, `yum`, `dnf`, `pip`
- **Service modules:** `service`, `systemd`
- **User modules:** `user`, `group`
- **Cloud modules:** `ec2`, `azure_rm`, `gcp_compute`

### 2. Plugins
- **Connection plugins:** `ssh`, `local`, `docker`
- **Callback plugins:** `json`, `yaml`, `minimal`
- **Lookup plugins:** `file`, `env`, `password`
- **Filter plugins:** `to_json`, `to_yaml`, `regex_replace`

### 3. Collections
```bash
# Install collection
ansible-galaxy collection install community.general

# List installed collections
ansible-galaxy collection list

# Use collection in playbook
---
- name: Use collection
  hosts: all
  collections:
    - community.general
  tasks:
    - name: Use module from collection
      community.general.docker_container:
        name: mycontainer
        image: nginx
```

---

## 🎓 Key Concepts

### Idempotency
Ansible operations can be run multiple times without changing the result beyond the initial application.

### Facts
System information gathered from managed nodes:
```bash
# Gather facts
ansible all -m setup

# Use facts in playbook
{{ ansible_hostname }}
{{ ansible_distribution }}
{{ ansible_default_ipv4.address }}
```

### Variables
```yaml
# Define variables
vars:
  http_port: 80
  app_name: myapp

# Use variables
- name: Configure port
  lineinfile:
    path: /etc/nginx/nginx.conf
    line: "listen {{ http_port }};"
```

### Handlers
Tasks that run only when notified:
```yaml
tasks:
  - name: Copy config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

---

## 📚 Additional Resources

- **Official Documentation:** https://docs.ansible.com/
- **Ansible Galaxy:** https://galaxy.ansible.com/
- **GitHub Repository:** https://github.com/ansible/ansible
- **Community:** https://www.ansible.com/community
- **Best Practices:** https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html

---

## ✅ Verification Checklist

- [ ] Ansible installed successfully
- [ ] Version verified
- [ ] Configuration file created
- [ ] Inventory file configured
- [ ] SSH keys set up
- [ ] Connectivity tested with ping
- [ ] Ad-hoc commands working
- [ ] Sample playbook executed
- [ ] Documentation reviewed

---

**Last Updated:** 2024-01-16  
**Version:** 1.0