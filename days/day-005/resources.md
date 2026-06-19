# Day 5: SELinux Installation and Configuration - Resources

## Official Documentation

### Red Hat Documentation
- **Red Hat Enterprise Linux SELinux Guide**
  - URL: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/index
  - Description: Comprehensive official guide for SELinux on RHEL
  - Topics: Installation, configuration, troubleshooting, policy management

- **Red Hat SELinux User's and Administrator's Guide**
  - URL: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/index
  - Description: Detailed SELinux administration guide for RHEL 8
  - Topics: Concepts, modes, contexts, booleans, troubleshooting

### SELinux Project
- **SELinux Project Wiki**
  - URL: https://github.com/SELinuxProject/selinux/wiki
  - Description: Official SELinux project documentation
  - Topics: Architecture, policy language, tools, development

- **SELinux Notebook**
  - URL: https://github.com/SELinuxProject/selinux-notebook
  - Description: Comprehensive SELinux reference guide
  - Topics: In-depth technical documentation

### CentOS Documentation
- **CentOS SELinux HowTo**
  - URL: https://wiki.centos.org/HowTos/SELinux
  - Description: CentOS-specific SELinux guide
  - Topics: Basic configuration, troubleshooting, common scenarios

### Fedora Documentation
- **Fedora SELinux FAQ**
  - URL: https://docs.fedoraproject.org/en-US/quick-docs/selinux-getting-started/
  - Description: Quick start guide for SELinux on Fedora
  - Topics: Getting started, common tasks, troubleshooting

---

## Tutorials and Guides

### Beginner Tutorials
- **DigitalOcean: An Introduction to SELinux**
  - URL: https://www.digitalocean.com/community/tutorials/an-introduction-to-selinux-on-centos-7-part-1-basic-concepts
  - Description: Three-part series on SELinux basics
  - Topics: Concepts, configuration, troubleshooting

- **LinuxConfig: SELinux Tutorial**
  - URL: https://linuxconfig.org/selinux-tutorial
  - Description: Practical SELinux tutorial with examples
  - Topics: Installation, modes, contexts, policies

- **TecMint: SELinux Essentials**
  - URL: https://www.tecmint.com/selinux-essentials-and-control-filesystem-access/
  - Description: Essential SELinux commands and concepts
  - Topics: Basic commands, file contexts, troubleshooting

### Advanced Guides
- **NSA SELinux Documentation**
  - URL: https://www.nsa.gov/what-we-do/research/selinux/
  - Description: Original SELinux documentation from NSA
  - Topics: Architecture, policy development, security

- **Gentoo SELinux Handbook**
  - URL: https://wiki.gentoo.org/wiki/SELinux
  - Description: Comprehensive SELinux guide
  - Topics: Installation, policy management, development

---

## Video Tutorials

### YouTube Channels
- **Red Hat Training**
  - Channel: Red Hat
  - Topics: SELinux basics, administration, troubleshooting
  - Level: Beginner to Advanced

- **Linux Academy**
  - Platform: A Cloud Guru
  - Topics: SELinux fundamentals, practical examples
  - Level: Beginner to Intermediate

- **Udemy SELinux Courses**
  - Platform: Udemy
  - Topics: Complete SELinux courses
  - Level: All levels

---

## Tools and Utilities

### Essential Tools
1. **policycoreutils**
   - Description: Core SELinux utilities
   - Commands: semanage, restorecon, setfiles
   - Installation: `sudo yum install policycoreutils`

2. **policycoreutils-python-utils**
   - Description: Python-based SELinux utilities
   - Commands: semanage, audit2allow, audit2why
   - Installation: `sudo yum install policycoreutils-python-utils`

3. **setroubleshoot-server**
   - Description: SELinux troubleshooting tools
   - Commands: sealert
   - Installation: `sudo yum install setroubleshoot-server`

4. **setools-console**
   - Description: SELinux policy analysis tools
   - Commands: sesearch, seinfo, sediff
   - Installation: `sudo yum install setools-console`

### GUI Tools
1. **system-config-selinux**
   - Description: Graphical SELinux configuration tool
   - Installation: `sudo yum install system-config-selinux`
   - Usage: GUI for managing SELinux settings

2. **setroubleshoot**
   - Description: SELinux troubleshooting GUI
   - Installation: `sudo yum install setroubleshoot`
   - Usage: Desktop notifications for SELinux denials

---

## Command Reference

### Quick Reference Cards
- **Red Hat SELinux Cheat Sheet**
  - URL: https://access.redhat.com/articles/2191331
  - Description: Quick reference for common SELinux commands

- **SELinux Command Reference**
  ```bash
  # Status and Mode
  sestatus                    # SELinux status
  getenforce                  # Current mode
  setenforce [0|1]           # Set mode temporarily
  
  # Contexts
  ls -Z                       # List with contexts
  ps -eZ                      # Process contexts
  chcon -t type file         # Change context
  restorecon -Rv /path       # Restore contexts
  
  # Booleans
  getsebool -a               # List all booleans
  setsebool -P bool on       # Set boolean permanently
  
  # Troubleshooting
  ausearch -m avc            # Search denials
  sealert -a /var/log/audit/audit.log  # Analyze denials
  audit2allow -a             # Generate policy
  
  # Policy Management
  semanage fcontext -l       # List file contexts
  semanage port -l           # List port contexts
  semodule -l                # List modules
  ```

---

## Books and Publications

### Recommended Books
1. **"SELinux System Administration" by Sven Vermeulen**
   - Publisher: Packt Publishing
   - Level: Intermediate to Advanced
   - Topics: Comprehensive SELinux administration

2. **"SELinux by Example" by Frank Mayer, Karl MacMillan, David Caplan**
   - Publisher: Prentice Hall
   - Level: Intermediate
   - Topics: Practical SELinux examples and use cases

3. **"The SELinux Notebook" (Free)**
   - Author: SELinux Project
   - URL: https://github.com/SELinuxProject/selinux-notebook
   - Level: All levels
   - Topics: Complete SELinux reference

---

## Community Resources

### Forums and Discussion
- **Red Hat Customer Portal**
  - URL: https://access.redhat.com/discussions
  - Description: Official Red Hat support forums
  - Topics: SELinux questions and discussions

- **Stack Overflow**
  - URL: https://stackoverflow.com/questions/tagged/selinux
  - Description: SELinux questions and answers
  - Topics: Troubleshooting, configuration, development

- **Unix & Linux Stack Exchange**
  - URL: https://unix.stackexchange.com/questions/tagged/selinux
  - Description: SELinux technical discussions
  - Topics: Advanced topics, troubleshooting

- **Reddit r/linuxadmin**
  - URL: https://www.reddit.com/r/linuxadmin/
  - Description: Linux administration community
  - Topics: SELinux discussions and help

### Mailing Lists
- **SELinux Mailing List**
  - URL: https://lore.kernel.org/selinux/
  - Description: Official SELinux development mailing list
  - Topics: Development, patches, discussions

---

## Practice Labs

### Online Labs
1. **KodeKloud SELinux Labs**
   - Platform: KodeKloud
   - Description: Hands-on SELinux practice
   - Topics: Installation, configuration, troubleshooting

2. **Katacoda SELinux Scenarios**
   - Platform: Katacoda (O'Reilly)
   - Description: Interactive SELinux tutorials
   - Topics: Practical scenarios and exercises

3. **Linux Academy Hands-on Labs**
   - Platform: A Cloud Guru
   - Description: SELinux practice environments
   - Topics: Real-world scenarios

### Local Practice
```bash
# Set up local practice environment
# 1. Install VirtualBox or VMware
# 2. Download CentOS/RHEL/Rocky Linux ISO
# 3. Create virtual machine
# 4. Install OS with SELinux enabled
# 5. Practice commands and scenarios
```

---

## Troubleshooting Resources

### Common Issues Database
- **Red Hat Knowledgebase**
  - URL: https://access.redhat.com/solutions
  - Description: Solutions for common SELinux issues
  - Search: "SELinux" + your issue

- **CentOS Wiki Troubleshooting**
  - URL: https://wiki.centos.org/HowTos/SELinux#Troubleshooting
  - Description: Common SELinux problems and solutions

### Log Analysis Tools
1. **ausearch**
   - Description: Search audit logs
   - Usage: `ausearch -m avc -ts recent`

2. **sealert**
   - Description: Analyze SELinux alerts
   - Usage: `sealert -a /var/log/audit/audit.log`

3. **audit2allow**
   - Description: Generate policy from denials
   - Usage: `audit2allow -a`

---

## Security Best Practices

### Security Guides
- **CIS Benchmarks for Linux**
  - URL: https://www.cisecurity.org/benchmark/red_hat_linux
  - Description: Security configuration benchmarks
  - Topics: SELinux hardening recommendations

- **NIST Security Guidelines**
  - URL: https://csrc.nist.gov/
  - Description: Federal security standards
  - Topics: Security controls including SELinux

- **DISA STIGs**
  - URL: https://public.cyber.mil/stigs/
  - Description: Security Technical Implementation Guides
  - Topics: SELinux security requirements

---

## Policy Development

### Policy Writing Resources
- **SELinux Policy Language Reference**
  - URL: https://github.com/SELinuxProject/selinux/wiki/PolicyLanguage
  - Description: Complete policy language documentation
  - Topics: Policy syntax, rules, macros

- **Reference Policy Documentation**
  - URL: https://github.com/SELinuxProject/refpolicy/wiki
  - Description: Reference policy documentation
  - Topics: Policy modules, interfaces, templates

### Development Tools
1. **sepolicy**
   - Description: SELinux policy inspection tool
   - Usage: `sepolicy generate`, `sepolicy network`

2. **sepolgen**
   - Description: SELinux policy generation tool
   - Usage: Generate policy templates

3. **audit2allow**
   - Description: Generate policy from audit logs
   - Usage: `audit2allow -a -M mypolicy`

---

## Certification and Training

### Red Hat Certifications
- **RHCSA (Red Hat Certified System Administrator)**
  - Topics: Basic SELinux administration
  - Level: Entry-level

- **RHCE (Red Hat Certified Engineer)**
  - Topics: Advanced SELinux configuration
  - Level: Intermediate

### Training Courses
- **Red Hat Training**
  - Course: RH124, RH134, RH254
  - Topics: SELinux fundamentals and administration

- **Linux Foundation Training**
  - Course: Various Linux administration courses
  - Topics: SELinux basics and configuration

---

## Additional Tools

### Monitoring Tools
1. **auditd**
   - Description: Linux audit daemon
   - Configuration: `/etc/audit/auditd.conf`
   - Logs: `/var/log/audit/audit.log`

2. **setroubleshoot**
   - Description: SELinux troubleshooting daemon
   - Service: `setroubleshootd`
   - Logs: `/var/log/messages`

### Analysis Tools
1. **sesearch**
   - Description: Search SELinux policy rules
   - Usage: `sesearch -A -s httpd_t`

2. **seinfo**
   - Description: Display SELinux policy information
   - Usage: `seinfo -t`

3. **sediff**
   - Description: Compare SELinux policies
   - Usage: `sediff policy1 policy2`

---

## Quick Links

### Essential Commands
```bash
# Documentation
man selinux
man sestatus
man semanage
man restorecon
man audit2allow

# Info pages
info selinux
```

### Configuration Files
```bash
# Main configuration
/etc/selinux/config

# Policy files
/etc/selinux/targeted/

# Audit logs
/var/log/audit/audit.log

# SELinux messages
/var/log/messages
```

---

## Updates and News

### Stay Updated
- **SELinux Project News**
  - URL: https://github.com/SELinuxProject/selinux/releases
  - Description: Latest SELinux releases and updates

- **Red Hat Security Blog**
  - URL: https://www.redhat.com/en/blog/channel/security
  - Description: Security updates including SELinux

- **Linux Security News**
  - URL: https://www.linux.com/topic/security/
  - Description: General Linux security news

---

## Summary

This resource collection provides:
- Official documentation and guides
- Tutorials for all skill levels
- Essential tools and utilities
- Community support channels
- Practice environments
- Troubleshooting resources
- Security best practices
- Policy development guides
- Certification information
- Monitoring and analysis tools

**Recommended Learning Path:**
1. Start with Red Hat SELinux Guide (basics)
2. Practice with DigitalOcean tutorials
3. Use KodeKloud labs for hands-on practice
4. Join community forums for support
5. Read SELinux Notebook for deep understanding
6. Consider RHCSA certification for validation

**Key Resources to Bookmark:**
- Red Hat SELinux Documentation
- SELinux Project Wiki
- Stack Overflow SELinux tag
- Red Hat Knowledgebase
- SELinux Notebook (GitHub)