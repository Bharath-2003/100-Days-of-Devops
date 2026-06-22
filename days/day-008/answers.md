# Day 8: Install Ansible - Step-by-Step Solutions

## 📋 Task: Install and Configure Ansible

This document provides multiple complete solutions for installing and configuring Ansible on different Linux distributions.

---

## Solution 1: Ubuntu/Debian Installation (Recommended)

### Step 1: Update System
```bash
# Update package index
sudo apt update

# Upgrade existing packages (optional)
sudo apt upgrade -y
```

### Step 2: Install Prerequisites
```bash
# Install software-properties-common
sudo apt install software-properties-common -y

# Verify installation
dpkg -l | grep software-properties-common
```

### Step 3: Add Ansible PPA
```bash
# Add Ansible official PPA
sudo add-apt-repository --yes --update ppa:ansible/ansible

# Update package index again
sudo apt update
```

### Step 4: Install Ansible
```bash
# Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version

# Expected output:
# ansible [core 2.14.x]
#   config file = /etc/ansible/ansible.cfg
#   configured module search path = [...]
#   ansible python module location = /usr/lib/python3/dist-packages/ansible
#   executable location = /usr/bin/ansible
#   python version = 3.x.x
```

### Step 5: Verify Installation
```bash
# Check Ansible version
ansible --version

# Check available commands
which ansible
which ansible-playbook
which ansible-galaxy
which ansible-vault

# Test with localhost
ansible localhost -m ping
```

**Expected Output:**
```
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## Solution 2: CentOS/RHEL 8/9 Installation

### Step 1: Enable EPEL Repository
```bash
# Install EPEL release
sudo dnf install epel-release -y

# Verify EPEL is enabled
sudo dnf repolist | grep epel
```

### Step 2: Install Ansible
```bash
# Install Ansible
sudo dnf install ansible -y

# Verify installation
ansible --version
```

### Step 3: Configure Firewall (if needed)
```bash
# Allow SSH (Ansible uses SSH)
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

# Verify
sudo firewall-cmd --list-all
```

### Step 4: Test Installation
```bash
# Test with localhost
ansible localhost -m ping

# Check Python version
ansible localhost -m setup -a "filter=ansible_python_version"
```

---

## Solution 3: Installation Using pip (All Distributions)

### Step 1: Install Python and pip
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip -y

# CentOS/RHEL
sudo dnf install python3 python3-pip -y

# Verify Python installation
python3 --version
pip3 --version
```

### Step 2: Upgrade pip
```bash
# Upgrade pip to latest version
python3 -m pip install --upgrade pip

# Verify pip version
pip3 --version
```

### Step 3: Install Ansible via pip
```bash
# Install Ansible
python3 -m pip install ansible

# Or install specific version
python3 -m pip install ansible==2.14.0

# Install with user flag (no sudo needed)
python3 -m pip install --user ansible
```

### Step 4: Add to PATH (if installed with --user)
```bash
# Add to PATH in ~/.bashrc
echo 'export PATH=$PATH:~/.local/bin' >> ~/.bashrc

# Reload bashrc
source ~/.bashrc

# Verify
which ansible
ansible --version
```

### Step 5: Install Additional Python Packages
```bash
# Install useful packages
pip3 install jinja2 PyYAML paramiko

# For cloud modules
pip3 install boto3 botocore  # AWS
pip3 install azure-cli        # Azure
pip3 install google-auth      # GCP
```

---

## Solution 4: Complete Setup with Configuration

### Step 1: Install Ansible (Ubuntu)
```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### Step 2: Create Project Directory
```bash
# Create project directory
mkdir -p ~/ansible-lab
cd ~/ansible-lab

# Create directory structure
mkdir -p {inventory,playbooks,roles,group_vars,host_vars,files,templates}

# Create files
touch ansible.cfg
touch inventory/hosts
touch playbooks/test.yml
```

### Step 3: Create Ansible Configuration
```bash
# Create ansible.cfg
cat > ansible.cfg << 'EOF'
[defaults]
inventory = ./inventory/hosts
host_key_checking = False
retry_files_enabled = False
log_path = ./ansible.log
roles_path = ./roles
forks = 10
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
EOF
```

### Step 4: Create Inventory File
```bash
# Create inventory file
cat > inventory/hosts << 'EOF'
# Local machine
localhost ansible_connection=local

# Example remote hosts
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com ansible_host=192.168.1.20

[all:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3
EOF
```

### Step 5: Create Test Playbook
```bash
# Create test playbook
cat > playbooks/test.yml << 'EOF'
---
- name: Test Ansible Installation
  hosts: localhost
  gather_facts: yes
  
  tasks:
    - name: Print Ansible version
      debug:
        msg: "Ansible version: {{ ansible_version.full }}"
    
    - name: Print hostname
      debug:
        msg: "Hostname: {{ ansible_hostname }}"
    
    - name: Print OS information
      debug:
        msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
    
    - name: Create test directory
      file:
        path: /tmp/ansible-test
        state: directory
        mode: '0755'
    
    - name: Create test file
      copy:
        content: |
          Ansible Installation Test
          Date: {{ ansible_date_time.iso8601 }}
          User: {{ ansible_user_id }}
        dest: /tmp/ansible-test/info.txt
        mode: '0644'
    
    - name: Display test file content
      command: cat /tmp/ansible-test/info.txt
      register: file_content
    
    - name: Show file content
      debug:
        var: file_content.stdout_lines
EOF
```

### Step 6: Run Test Playbook
```bash
# Run the test playbook
ansible-playbook playbooks/test.yml

# Run with verbose output
ansible-playbook playbooks/test.yml -v

# Check syntax
ansible-playbook playbooks/test.yml --syntax-check

# Dry run
ansible-playbook playbooks/test.yml --check
```

### Step 7: Test Ad-Hoc Commands
```bash
# Ping localhost
ansible localhost -m ping

# Get system information
ansible localhost -m setup

# Check disk space
ansible localhost -a "df -h"

# Check memory
ansible localhost -a "free -m"

# Check uptime
ansible localhost -a "uptime"
```

---

## Solution 5: Multi-Node Setup with SSH Keys

### Step 1: Install Ansible (Control Node)
```bash
# On control node
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### Step 2: Generate SSH Key Pair
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "ansible-control" -f ~/.ssh/ansible_key

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/ansible_key

# Display public key
cat ~/.ssh/ansible_key.pub
```

### Step 3: Set Up Managed Nodes
```bash
# Copy SSH key to managed nodes
ssh-copy-id -i ~/.ssh/ansible_key.pub user@node1.example.com
ssh-copy-id -i ~/.ssh/ansible_key.pub user@node2.example.com
ssh-copy-id -i ~/.ssh/ansible_key.pub user@node3.example.com

# Test SSH connection
ssh -i ~/.ssh/ansible_key user@node1.example.com
```

### Step 4: Create Inventory with SSH Key
```bash
# Create inventory
cat > inventory/hosts << 'EOF'
[managed_nodes]
node1.example.com ansible_ssh_private_key_file=~/.ssh/ansible_key
node2.example.com ansible_ssh_private_key_file=~/.ssh/ansible_key
node3.example.com ansible_ssh_private_key_file=~/.ssh/ansible_key

[managed_nodes:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3
EOF
```

### Step 5: Test Connectivity
```bash
# Ping all managed nodes
ansible managed_nodes -m ping

# Check connectivity with verbose output
ansible managed_nodes -m ping -v

# Gather facts from all nodes
ansible managed_nodes -m setup

# Run command on all nodes
ansible managed_nodes -a "hostname"
ansible managed_nodes -a "uptime"
```

### Step 6: Create Multi-Node Playbook
```bash
# Create playbook for managed nodes
cat > playbooks/setup-nodes.yml << 'EOF'
---
- name: Setup Managed Nodes
  hosts: managed_nodes
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      when: ansible_os_family == "Debian"
    
    - name: Install common packages
      apt:
        name:
          - vim
          - git
          - curl
          - wget
          - htop
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Create ansible user
      user:
        name: ansible
        shell: /bin/bash
        create_home: yes
        state: present
    
    - name: Add ansible user to sudoers
      lineinfile:
        path: /etc/sudoers.d/ansible
        line: 'ansible ALL=(ALL) NOPASSWD: ALL'
        create: yes
        mode: '0440'
        validate: 'visudo -cf %s'
    
    - name: Display node information
      debug:
        msg: |
          Hostname: {{ ansible_hostname }}
          IP Address: {{ ansible_default_ipv4.address }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Python: {{ ansible_python_version }}
EOF
```

### Step 7: Run Multi-Node Playbook
```bash
# Run playbook
ansible-playbook playbooks/setup-nodes.yml

# Run with limit
ansible-playbook playbooks/setup-nodes.yml --limit node1.example.com

# Run with tags (if defined)
ansible-playbook playbooks/setup-nodes.yml --tags "install"
```

---

## Solution 6: Installation with Ansible Galaxy

### Step 1: Install Ansible
```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### Step 2: Install Roles from Galaxy
```bash
# Install popular roles
ansible-galaxy install geerlingguy.nginx
ansible-galaxy install geerlingguy.docker
ansible-galaxy install geerlingguy.postgresql

# List installed roles
ansible-galaxy list

# Show role info
ansible-galaxy info geerlingguy.nginx
```

### Step 3: Create Requirements File
```bash
# Create requirements.yml
cat > requirements.yml << 'EOF'
---
roles:
  - name: geerlingguy.nginx
    version: 3.1.4
  
  - name: geerlingguy.docker
    version: 6.1.0
  
  - name: geerlingguy.postgresql
    version: 3.4.0

collections:
  - name: community.general
    version: 6.0.0
  
  - name: ansible.posix
    version: 1.5.0
EOF
```

### Step 4: Install from Requirements
```bash
# Install roles and collections
ansible-galaxy install -r requirements.yml

# Force reinstall
ansible-galaxy install -r requirements.yml --force

# Install to custom path
ansible-galaxy install -r requirements.yml -p ./custom-roles/
```

### Step 5: Use Installed Role
```bash
# Create playbook using role
cat > playbooks/nginx-setup.yml << 'EOF'
---
- name: Setup Nginx
  hosts: webservers
  become: yes
  
  roles:
    - role: geerlingguy.nginx
      nginx_vhosts:
        - listen: "80"
          server_name: "example.com"
          root: "/var/www/html"
          index: "index.html"
EOF
```

---

## Verification Steps

### 1. Check Installation
```bash
# Verify Ansible is installed
ansible --version

# Check all Ansible commands
which ansible
which ansible-playbook
which ansible-galaxy
which ansible-vault
which ansible-doc
which ansible-config
```

### 2. Test Basic Functionality
```bash
# Test ping module
ansible localhost -m ping

# Test setup module
ansible localhost -m setup | head -20

# Test command module
ansible localhost -m command -a "date"

# Test shell module
ansible localhost -m shell -a "echo $HOME"
```

### 3. Verify Configuration
```bash
# Show configuration
ansible-config dump

# Show specific config
ansible-config dump --only-changed

# List configuration files
ansible-config list
```

### 4. Test Playbook Execution
```bash
# Create simple playbook
cat > test-playbook.yml << 'EOF'
---
- hosts: localhost
  tasks:
    - name: Test task
      debug:
        msg: "Ansible is working!"
EOF

# Run playbook
ansible-playbook test-playbook.yml
```

---

## Troubleshooting Common Issues

### Issue 1: Command Not Found
```bash
# Solution: Add to PATH
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc

# Or create symlink
sudo ln -s /usr/local/bin/ansible /usr/bin/ansible
```

### Issue 2: Python Not Found
```bash
# Solution: Install Python
sudo apt install python3 python3-pip -y

# Set Python interpreter in inventory
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Issue 3: Permission Denied
```bash
# Solution: Check SSH keys
ssh-add -l
ssh-add ~/.ssh/id_ed25519

# Or use password
ansible all -m ping --ask-pass

# Or use become
ansible all -m ping --become --ask-become-pass
```

### Issue 4: Module Not Found
```bash
# Solution: Install required Python packages
pip3 install <module-name>

# Or install Ansible with all extras
pip3 install 'ansible[all]'
```

---

## Post-Installation Tasks

### 1. Create Ansible User
```bash
# On managed nodes
sudo useradd -m -s /bin/bash ansible
sudo usermod -aG sudo ansible

# Set up passwordless sudo
echo "ansible ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/ansible
```

### 2. Configure SSH
```bash
# On control node
cat >> ~/.ssh/config << 'EOF'
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600
EOF

# Create sockets directory
mkdir -p ~/.ssh/sockets
```

### 3. Set Up Logging
```bash
# Create log directory
sudo mkdir -p /var/log/ansible
sudo chown $USER:$USER /var/log/ansible

# Configure logging in ansible.cfg
[defaults]
log_path = /var/log/ansible/ansible.log
```

---

## Success Criteria

- [ ] Ansible installed successfully
- [ ] Version command works
- [ ] Localhost ping successful
- [ ] Configuration file created
- [ ] Inventory file configured
- [ ] Test playbook runs successfully
- [ ] SSH connectivity established
- [ ] Ad-hoc commands working
- [ ] Ansible Galaxy accessible
- [ ] Documentation accessible

---

## Next Steps

1. Create your first playbook
2. Set up inventory for your infrastructure
3. Install useful roles from Ansible Galaxy
4. Learn about Ansible Vault for secrets
5. Explore Ansible modules
6. Practice with ad-hoc commands
7. Build reusable roles
8. Integrate with CI/CD pipeline

---

**Completion Time:** 1-2 hours  
**Difficulty:** Beginner-Intermediate  
**Last Updated:** 2024-01-16