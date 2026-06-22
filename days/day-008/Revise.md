# Day 8: Install Ansible - Quick Revision Guide

## 🎯 Quick Summary

Ansible is an agentless automation tool that uses SSH to manage configurations, deploy applications, and orchestrate tasks across multiple systems. It uses YAML-based playbooks for defining automation tasks.

**Key Points:**
- Agentless (no software on managed nodes)
- Uses SSH for communication
- YAML-based configuration (Playbooks)
- Idempotent operations
- Large module library

---

## ⚡ Essential Commands

### Installation Commands

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y

# CentOS/RHEL
sudo dnf install epel-release -y
sudo dnf install ansible -y

# Using pip
pip3 install ansible

# Verify installation
ansible --version
```

### Basic Commands

```bash
# Ping all hosts
ansible all -m ping

# Run command on all hosts
ansible all -a "uptime"

# Use specific module
ansible all -m setup

# Run with sudo
ansible all -a "apt update" --become

# Limit to specific hosts
ansible webservers -m ping

# Use specific inventory
ansible all -i inventory/hosts -m ping
```

### Playbook Commands

```bash
# Run playbook
ansible-playbook playbook.yml

# Syntax check
ansible-playbook playbook.yml --syntax-check

# Dry run (check mode)
ansible-playbook playbook.yml --check

# Verbose output
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv

# List tasks
ansible-playbook playbook.yml --list-tasks

# List hosts
ansible-playbook playbook.yml --list-hosts

# Run specific tags
ansible-playbook playbook.yml --tags "install,configure"

# Skip tags
ansible-playbook playbook.yml --skip-tags "deploy"

# Limit to hosts
ansible-playbook playbook.yml --limit webservers
```

### Inventory Commands

```bash
# List all hosts
ansible all --list-hosts

# List hosts in group
ansible webservers --list-hosts

# Show inventory graph
ansible-inventory --graph

# Show inventory as JSON
ansible-inventory --list

# Show host variables
ansible-inventory --host server1.example.com
```

### Galaxy Commands

```bash
# Install role
ansible-galaxy install geerlingguy.nginx

# Install from requirements
ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Search for roles
ansible-galaxy search nginx

# Create role structure
ansible-galaxy init my-role

# Remove role
ansible-galaxy remove role-name
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

# Run playbook with vault
ansible-playbook playbook.yml --ask-vault-pass
```

---

## 📋 Configuration Files

### ansible.cfg (Essential Settings)

```ini
[defaults]
inventory = ./inventory/hosts
host_key_checking = False
retry_files_enabled = False
log_path = ./ansible.log
forks = 10

[privilege_escalation]
become = True
become_method = sudo
become_user = root

[ssh_connection]
pipelining = True
```

### Inventory File (hosts)

```ini
# Simple inventory
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com ansible_host=192.168.1.20

[all:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3
```

### Basic Playbook Structure

```yaml
---
- name: Playbook Name
  hosts: all
  become: yes
  
  vars:
    package_name: nginx
  
  tasks:
    - name: Install package
      apt:
        name: "{{ package_name }}"
        state: present
    
    - name: Start service
      service:
        name: "{{ package_name }}"
        state: started
        enabled: yes
```

---

## 🔧 Common Modules

### System Modules
```yaml
# Command module
- command: /usr/bin/uptime

# Shell module
- shell: echo $HOME

# Service module
- service:
    name: nginx
    state: started
    enabled: yes

# User module
- user:
    name: john
    state: present
    shell: /bin/bash

# Group module
- group:
    name: developers
    state: present
```

### File Modules
```yaml
# Copy module
- copy:
    src: /tmp/file.txt
    dest: /etc/file.txt
    mode: '0644'

# File module
- file:
    path: /tmp/directory
    state: directory
    mode: '0755'

# Template module
- template:
    src: config.j2
    dest: /etc/app/config.conf

# Lineinfile module
- lineinfile:
    path: /etc/hosts
    line: '192.168.1.10 server1'
```

### Package Modules
```yaml
# Apt module (Debian/Ubuntu)
- apt:
    name: nginx
    state: present
    update_cache: yes

# Yum module (CentOS/RHEL)
- yum:
    name: httpd
    state: present

# Package module (generic)
- package:
    name: vim
    state: present
```

---

## 🏗️ Project Structure

```
ansible-project/
├── ansible.cfg
├── inventory/
│   └── hosts
├── playbooks/
│   ├── site.yml
│   └── webservers.yml
├── roles/
│   └── common/
│       ├── tasks/
│       ├── handlers/
│       ├── templates/
│       ├── files/
│       └── vars/
├── group_vars/
│   └── all.yml
└── host_vars/
    └── server1.yml
```

---

## 🔐 SSH Setup

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "ansible"

# Copy to managed nodes
ssh-copy-id user@server1.example.com

# Test connection
ssh user@server1.example.com

# Test with Ansible
ansible all -m ping
```

---

## 🐛 Troubleshooting Quick Reference

### Issue: Permission Denied
```bash
# Solution 1: Check SSH keys
ssh-add -l
ssh-add ~/.ssh/id_ed25519

# Solution 2: Use password
ansible all -m ping --ask-pass

# Solution 3: Use become
ansible all -m ping --become --ask-become-pass
```

### Issue: Host Key Verification Failed
```bash
# Solution 1: Remove old key
ssh-keygen -R hostname

# Solution 2: Disable checking (not recommended)
# In ansible.cfg:
[defaults]
host_key_checking = False
```

### Issue: Module Not Found
```bash
# Solution: Install Python packages
pip3 install <module-name>

# Or check Python path
ansible all -m setup -a "filter=ansible_python_version"
```

### Issue: Slow Execution
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

## 📊 Key Concepts

### Idempotency
Operations can be run multiple times without changing the result beyond the initial application.

### Facts
System information gathered from managed nodes:
```yaml
{{ ansible_hostname }}
{{ ansible_distribution }}
{{ ansible_default_ipv4.address }}
{{ ansible_memtotal_mb }}
```

### Variables
```yaml
# Define
vars:
  http_port: 80

# Use
"{{ http_port }}"
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

### Roles
Reusable, organized collections of tasks, variables, files, and templates.

---

## 🎓 Interview Questions

### Basic Level

**Q1: What is Ansible?**
A: Ansible is an open-source automation tool for configuration management, application deployment, and task automation. It's agentless and uses SSH for communication.

**Q2: What are the main components of Ansible?**
A: 
- Control Node (where Ansible is installed)
- Managed Nodes (target systems)
- Inventory (list of managed nodes)
- Playbooks (YAML files with tasks)
- Modules (units of work)

**Q3: What is a playbook?**
A: A playbook is a YAML file that defines a series of tasks to be executed on managed nodes. It's the main way to define automation in Ansible.

**Q4: What is idempotency?**
A: Idempotency means running the same operation multiple times produces the same result. Ansible modules are designed to be idempotent.

**Q5: How do you install Ansible?**
A: 
```bash
# Ubuntu/Debian
sudo apt install ansible

# CentOS/RHEL
sudo dnf install ansible

# Using pip
pip install ansible
```

### Intermediate Level

**Q6: What is the difference between command and shell modules?**
A: 
- `command`: Executes commands without shell processing (no pipes, redirects)
- `shell`: Executes commands through shell (supports pipes, redirects, variables)

**Q7: What is Ansible Vault?**
A: Ansible Vault is a feature that allows you to encrypt sensitive data like passwords, keys, and certificates in playbooks or variable files.

**Q8: What are handlers in Ansible?**
A: Handlers are special tasks that run only when notified by other tasks. They're typically used for service restarts or reloads.

**Q9: What is the difference between vars and defaults in roles?**
A: 
- `defaults`: Lowest priority variables, easily overridden
- `vars`: Higher priority variables, harder to override

**Q10: How do you limit playbook execution to specific hosts?**
A: 
```bash
ansible-playbook playbook.yml --limit webservers
ansible-playbook playbook.yml --limit server1.example.com
```

### Advanced Level

**Q11: How do you implement rolling updates in Ansible?**
A: Use `serial` keyword:
```yaml
- hosts: webservers
  serial: 2
  tasks:
    - name: Update application
      ...
```

**Q12: What are Ansible Collections?**
A: Collections are a distribution format for Ansible content (modules, plugins, roles). They allow better organization and versioning.

**Q13: How do you create custom Ansible modules?**
A: Create a Python script in the `library/` directory that follows Ansible module conventions, using `AnsibleModule` class.

**Q14: What is the order of precedence for variables?**
A: (Highest to lowest)
1. Extra vars (`-e`)
2. Task vars
3. Block vars
4. Role and include vars
5. Play vars
6. Host facts
7. Playbook host_vars
8. Playbook group_vars
9. Inventory host_vars
10. Inventory group_vars
11. Role defaults

**Q15: How do you optimize Ansible performance?**
A:
- Enable pipelining
- Increase forks
- Use fact caching
- Disable gathering facts when not needed
- Use async tasks for long-running operations

---

## 📝 Pre-Exam Checklist

- [ ] Understand Ansible architecture
- [ ] Know installation methods
- [ ] Can create inventory files
- [ ] Can write basic playbooks
- [ ] Understand modules and their usage
- [ ] Know how to use variables
- [ ] Understand handlers
- [ ] Can use Ansible Vault
- [ ] Know Galaxy and roles
- [ ] Can troubleshoot common issues
- [ ] Understand idempotency
- [ ] Know configuration file locations
- [ ] Can use ad-hoc commands
- [ ] Understand privilege escalation

---

## 🔄 Common Workflows

### Initial Setup
```bash
# 1. Install Ansible
sudo apt install ansible

# 2. Create project structure
mkdir -p ansible-project/{inventory,playbooks,roles}
cd ansible-project

# 3. Create configuration
cat > ansible.cfg << EOF
[defaults]
inventory = ./inventory/hosts
host_key_checking = False
EOF

# 4. Create inventory
cat > inventory/hosts << EOF
[servers]
server1.example.com
EOF

# 5. Test connectivity
ansible all -m ping
```

### Deploy Application
```bash
# 1. Create playbook
cat > playbooks/deploy.yml << EOF
---
- hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
    
    - name: Copy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx
  
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
EOF

# 2. Run playbook
ansible-playbook playbooks/deploy.yml

# 3. Verify
ansible webservers -a "systemctl status nginx"
```

---

## 💡 Pro Tips

1. **Use Check Mode** - Always test with `--check` first
2. **Enable Pipelining** - Faster execution
3. **Use Tags** - Organize and run specific tasks
4. **Fact Caching** - Speed up playbook runs
5. **Use Roles** - Organize reusable code
6. **Ansible Vault** - Secure sensitive data
7. **Version Control** - Keep playbooks in Git
8. **Documentation** - Comment your playbooks
9. **Testing** - Use Molecule for role testing
10. **Linting** - Use ansible-lint for best practices

---

## 📚 Quick Reference Links

- **Documentation:** https://docs.ansible.com/
- **Galaxy:** https://galaxy.ansible.com/
- **GitHub:** https://github.com/ansible/ansible
- **Module Index:** https://docs.ansible.com/ansible/latest/collections/
- **Best Practices:** https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html

---

## ✅ Verification Commands

```bash
# Check installation
ansible --version

# Test connectivity
ansible all -m ping

# Verify configuration
ansible-config dump

# List inventory
ansible-inventory --list

# Test playbook syntax
ansible-playbook playbook.yml --syntax-check

# Dry run
ansible-playbook playbook.yml --check
```

---

**Remember:**
- Ansible is agentless (uses SSH)
- Playbooks are YAML files
- Modules are idempotent
- Use roles for organization
- Vault for secrets
- Galaxy for community roles

**Last Updated:** 2024-01-16