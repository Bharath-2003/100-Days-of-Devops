# Day 8: Install Ansible - Additional Resources

## 📚 Official Documentation

### Primary Resources
- **Ansible Documentation:** https://docs.ansible.com/
- **Ansible GitHub:** https://github.com/ansible/ansible
- **Ansible Galaxy:** https://galaxy.ansible.com/
- **Ansible Blog:** https://www.ansible.com/blog
- **Release Notes:** https://github.com/ansible/ansible/blob/devel/changelogs/CHANGELOG-v2.14.rst

### Installation Guides
- **Installation Guide:** https://docs.ansible.com/ansible/latest/installation_guide/index.html
- **Platform Support:** https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#control-node-requirements
- **Python Requirements:** https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#prerequisites

---

## 🛠️ Tools and Utilities

### Essential Tools

1. **Ansible Lint**
   ```bash
   # Install
   pip install ansible-lint
   
   # Usage
   ansible-lint playbook.yml
   ```
   - **Purpose:** Lint Ansible playbooks
   - **Website:** https://ansible-lint.readthedocs.io/

2. **Ansible Navigator**
   ```bash
   # Install
   pip install ansible-navigator
   
   # Usage
   ansible-navigator run playbook.yml
   ```
   - **Purpose:** TUI for Ansible
   - **Website:** https://ansible-navigator.readthedocs.io/

3. **Molecule**
   ```bash
   # Install
   pip install molecule molecule-docker
   
   # Usage
   molecule init role my-role
   molecule test
   ```
   - **Purpose:** Test Ansible roles
   - **Website:** https://molecule.readthedocs.io/

4. **Ansible Semaphore**
   - **Purpose:** Web UI for Ansible
   - **Website:** https://www.ansible-semaphore.com/
   - **GitHub:** https://github.com/ansible-semaphore/semaphore

5. **AWX (Ansible Tower Open Source)**
   - **Purpose:** Web-based UI and REST API
   - **Website:** https://github.com/ansible/awx
   - **Documentation:** https://ansible.readthedocs.io/projects/awx/

### IDE Extensions

1. **VS Code Extensions**
   - Ansible by Red Hat
   - YAML by Red Hat
   - Ansible Vault
   - Jinja2 Snippet Kit

2. **PyCharm/IntelliJ**
   - Ansible Support Plugin

3. **Vim/Neovim**
   - ansible-vim plugin

---

## 📖 Learning Resources

### Official Courses

1. **Red Hat Training**
   - DO407: Automation with Ansible I
   - DO409: Automation with Ansible II
   - Website: https://www.redhat.com/en/services/training

2. **Ansible for DevOps**
   - Free online book
   - Author: Jeff Geerling
   - Website: https://www.ansiblefordevops.com/

### Online Courses

1. **Udemy**
   - "Ansible for the Absolute Beginner"
   - "Mastering Ansible"
   - "Ansible Advanced"

2. **Pluralsight**
   - "Getting Started with Ansible"
   - "Ansible Fundamentals"

3. **LinkedIn Learning**
   - "Learning Ansible"
   - "Ansible Essential Training"

4. **A Cloud Guru**
   - "Ansible Quick Start"
   - "Hands-On Ansible"

### YouTube Channels

1. **Jeff Geerling**
   - Channel: https://www.youtube.com/c/JeffGeerling
   - Focus: Ansible tutorials and tips

2. **TechWorld with Nana**
   - Ansible crash course
   - DevOps tutorials

3. **LearnLinuxTV**
   - Ansible tutorial series
   - Linux administration

4. **NetworkChuck**
   - Ansible for beginners
   - Practical examples

---

## 📝 Cheat Sheets

### Quick Reference

**Installation Commands:**
```bash
# Ubuntu/Debian
sudo apt install ansible

# CentOS/RHEL
sudo dnf install ansible

# pip
pip install ansible

# Verify
ansible --version
```

**Basic Commands:**
```bash
# Ad-hoc commands
ansible all -m ping
ansible all -a "uptime"
ansible all -m setup

# Playbook commands
ansible-playbook playbook.yml
ansible-playbook playbook.yml --check
ansible-playbook playbook.yml --syntax-check

# Inventory commands
ansible-inventory --list
ansible-inventory --graph

# Galaxy commands
ansible-galaxy install role-name
ansible-galaxy list
```

**Configuration Locations:**
```
/etc/ansible/ansible.cfg          # System-wide
~/.ansible.cfg                     # User-specific
./ansible.cfg                      # Project-specific (highest priority)
```

### Downloadable Cheat Sheets

1. **Official Ansible Cheat Sheet**
   - https://docs.ansible.com/ansible/latest/user_guide/cheatsheet.html

2. **DevHints Ansible Cheat Sheet**
   - https://devhints.io/ansible

3. **Ansible Module Index**
   - https://docs.ansible.com/ansible/latest/collections/index_module.html

---

## 🎓 Certifications

### Red Hat Certifications

1. **Red Hat Certified Specialist in Ansible Automation (EX407)**
   - **Level:** Intermediate
   - **Duration:** 4 hours
   - **Format:** Performance-based
   - **Website:** https://www.redhat.com/en/services/certification/rhcs-ansible-automation

2. **Red Hat Certified Engineer (RHCE)**
   - Includes Ansible automation
   - Advanced level certification

### Preparation Resources

1. **Practice Exams**
   - Whizlabs
   - Udemy practice tests
   - Linux Academy labs

2. **Study Guides**
   - Official Red Hat study guide
   - Community study materials
   - GitHub practice repositories

---

## 💻 Sample Projects and Repositories

### GitHub Repositories

1. **Ansible Examples**
   - https://github.com/ansible/ansible-examples
   - Official examples from Ansible

2. **Awesome Ansible**
   - https://github.com/ansible-community/awesome-ansible
   - Curated list of Ansible resources

3. **Ansible for DevOps Examples**
   - https://github.com/geerlingguy/ansible-for-devops
   - Jeff Geerling's examples

4. **Ansible Best Practices**
   - https://github.com/enginyoyen/ansible-best-practises
   - Production-ready examples

### Sample Playbooks

1. **Web Server Setup**
   ```yaml
   ---
   - name: Setup Web Server
     hosts: webservers
     become: yes
     tasks:
       - name: Install nginx
         apt:
           name: nginx
           state: present
       
       - name: Start nginx
         service:
           name: nginx
           state: started
           enabled: yes
   ```

2. **User Management**
   ```yaml
   ---
   - name: Manage Users
     hosts: all
     become: yes
     tasks:
       - name: Create users
         user:
           name: "{{ item }}"
           state: present
           shell: /bin/bash
         loop:
           - alice
           - bob
           - charlie
   ```

3. **Package Installation**
   ```yaml
   ---
   - name: Install Packages
     hosts: all
     become: yes
     tasks:
       - name: Install common packages
         package:
           name:
             - vim
             - git
             - curl
             - wget
           state: present
   ```

---

## 🌐 Community Resources

### Forums and Discussion

1. **Ansible Community**
   - Website: https://www.ansible.com/community
   - Forum: https://forum.ansible.com/

2. **Reddit**
   - r/ansible: https://www.reddit.com/r/ansible/
   - r/devops: https://www.reddit.com/r/devops/

3. **Stack Overflow**
   - Tag: [ansible]
   - https://stackoverflow.com/questions/tagged/ansible

4. **Discord/Slack**
   - Ansible Community Discord
   - DevOps Slack channels

### Mailing Lists

1. **Ansible Project**
   - ansible-project@googlegroups.com

2. **Ansible Development**
   - ansible-devel@googlegroups.com

3. **Ansible Announce**
   - ansible-announce@googlegroups.com

---

## 📚 Books

### Recommended Reading

1. **"Ansible for DevOps"**
   - Author: Jeff Geerling
   - Level: Beginner to Advanced
   - Website: https://www.ansiblefordevops.com/

2. **"Ansible: Up and Running"**
   - Authors: Lorin Hochstein, René Moser
   - Publisher: O'Reilly
   - Level: Intermediate

3. **"Mastering Ansible"**
   - Author: James Freeman
   - Publisher: Packt
   - Level: Advanced

4. **"Ansible Playbook Essentials"**
   - Author: Gourav Shah
   - Publisher: Packt
   - Level: Beginner

5. **"Learning Ansible 2"**
   - Author: Fabio Alessandro Locati
   - Publisher: Packt
   - Level: Beginner to Intermediate

---

## 🔧 Configuration Examples

### ansible.cfg Examples

**Basic Configuration:**
```ini
[defaults]
inventory = ./inventory
host_key_checking = False
retry_files_enabled = False
log_path = ./ansible.log

[privilege_escalation]
become = True
become_method = sudo
```

**Advanced Configuration:**
```ini
[defaults]
inventory = ./inventory
forks = 20
timeout = 30
host_key_checking = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/facts_cache
fact_caching_timeout = 86400
callback_whitelist = profile_tasks, timer
stdout_callback = yaml

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

### Inventory Examples

**Static Inventory:**
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[all:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3
```

**Dynamic Inventory (AWS):**
```bash
# Install boto3
pip install boto3

# Use AWS EC2 plugin
ansible-inventory -i aws_ec2.yml --graph
```

---

## 🎯 Practice Labs

### Free Practice Environments

1. **Katacoda Ansible Scenarios**
   - Interactive browser-based labs
   - Website: https://www.katacoda.com/courses/ansible

2. **Play with Docker**
   - Practice Ansible with containers
   - Website: https://labs.play-with-docker.com/

3. **Vagrant + VirtualBox**
   - Create local test environment
   - Use Vagrantfile for multi-VM setup

### Lab Setup Example

```bash
# Vagrantfile for Ansible lab
Vagrant.configure("2") do |config|
  config.vm.define "control" do |control|
    control.vm.box = "ubuntu/focal64"
    control.vm.hostname = "ansible-control"
    control.vm.network "private_network", ip: "192.168.56.10"
  end

  (1..3).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.box = "ubuntu/focal64"
      node.vm.hostname = "node#{i}"
      node.vm.network "private_network", ip: "192.168.56.#{10+i}"
    end
  end
end
```

---

## 🔍 Troubleshooting Resources

### Common Issues Database

1. **Ansible Issues on GitHub**
   - https://github.com/ansible/ansible/issues

2. **Known Issues**
   - https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html

### Debug Tools

```bash
# Verbose output
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv
ansible-playbook playbook.yml -vvv

# Debug module
- debug:
    var: variable_name
    verbosity: 2

# Check mode
ansible-playbook playbook.yml --check

# Diff mode
ansible-playbook playbook.yml --diff
```

---

## 📊 Monitoring and Logging

### Tools

1. **Ansible Tower/AWX**
   - Job history and logs
   - Dashboard and reporting

2. **ARA (Ansible Run Analysis)**
   - https://github.com/ansible-community/ara
   - Records and visualizes playbook runs

3. **Ansible Callback Plugins**
   - profile_tasks
   - timer
   - json
   - logstash

### Configuration

```ini
# In ansible.cfg
[defaults]
log_path = /var/log/ansible/ansible.log
callback_whitelist = profile_tasks, timer

# Install ARA
pip install ara
export ANSIBLE_CALLBACK_PLUGINS="$(python3 -m ara.setup.callback_plugins)"
```

---

## 🚀 Advanced Topics

### Resources for Advanced Users

1. **Custom Modules**
   - https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html

2. **Custom Plugins**
   - https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html

3. **Ansible Collections**
   - https://docs.ansible.com/ansible/latest/user_guide/collections_using.html

4. **Ansible Content Collections**
   - https://galaxy.ansible.com/

---

## 📱 Mobile Apps

### Ansible Mobile Tools

1. **Termux (Android)**
   - Run Ansible on Android
   - Install: `pkg install ansible`

2. **iSH (iOS)**
   - Linux shell for iOS
   - Can run Ansible

---

## 🔗 Quick Links

- **Official Site:** https://www.ansible.com/
- **Documentation:** https://docs.ansible.com/
- **Galaxy:** https://galaxy.ansible.com/
- **GitHub:** https://github.com/ansible/ansible
- **Forum:** https://forum.ansible.com/
- **Blog:** https://www.ansible.com/blog
- **Twitter:** @ansible
- **YouTube:** Ansible Official Channel

---

## 💡 Tips and Best Practices

### Best Practice Resources

1. **Official Best Practices**
   - https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html

2. **Style Guide**
   - https://docs.ansible.com/ansible/latest/dev_guide/style_guide/index.html

3. **Security Best Practices**
   - Use Ansible Vault for secrets
   - Implement least privilege
   - Regular updates

---

**Last Updated:** 2024-01-16  
**Maintained By:** DevOps Learning Community