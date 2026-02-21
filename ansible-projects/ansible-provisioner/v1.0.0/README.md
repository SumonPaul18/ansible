
# 🚀 Ansible Provisioner v1.0.0

> **AAutomated Server Provisioning & Configuration with Ansible**  
> *Prepare, secure, and manage Linux target nodes with Ansible — fast, repeatable, and production-ready.*

[![Ansible](https://img.shields.io/badge/Ansible-2.9+-red.svg)](https://ansible.com)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Stable-green.svg)]()

---

## 🎯 Purpose

This repository contains Ansible playbooks and configurations for automated provisioning and configuration of target Linux servers.
It helps you:

- ✅ Provision SSH access and key-based authentication
- ✅ Harden security with UFW firewall and SSH config
- ✅ Install essential tools (`htop`, `curl`, `git`, `vim`, etc.)
- ✅ Create dedicated users with passwordless `sudo` access
- ✅ Validate network connectivity between control and target nodes

Ideal for:  
🔹 Home labs & testing environments  
🔹 Staging/production server onboarding  
🔹 DevOps engineers scaling infrastructure  
🔹 Learning Ansible with real-world examples

---

## ✨ Features

| Feature                 | Description                                                 |
| ----------------------- | ------------------------------------------------------------------- |
| 🔐 SSH Hardening        | Configures secure SSH settings (key-based auth, disable root login) |
| 🛡️ Firewall Setup      | Enables and configures UFW with essential rules                     |
| 👤 User Management      | Creates users with passwordless sudo access                         |
| 📦 Package Management   | Installs essential tools (htop, vim, git, curl, etc.)               |
| 🔄 Idempotent Execution | Safe to run multiple times without side effects                     |
| 🌐 Bilingual Docs       | Full documentation in English and Bangla                            |
| ⚙️ Configurable         | Easy to modify inventory, variables, and tasks                      |

---

## ⚙️ Prerequisites

Before you begin, ensure:

### Control Node (Ansible Controller)
```bash
# ✅ Ubuntu/Debian-based system
# ✅ Python 3.8+
# ✅ Ansible 2.9+
# ✅ SSH access to target nodes
# ✅ sudo privileges on control node
```

### Target Nodes (Managed Servers)
```bash
# ✅ Ubuntu/Debian Linux (20.04 LTS recommended)
# ✅ SSH server enabled
# ✅ Python 3 installed (for Ansible modules)
# ✅ Network connectivity from control node
# ✅ User account with sudo privileges
```

### Install Ansible (Ubuntu/Debian)

#### Update system packages
```
sudo apt update && sudo apt upgrade -y
```
#### Install Ansible
```
sudo apt install ansible -y
```
#### Verify installation
```
ansible --version
ansible-playbook --version
```
---

## 📁 Project Structure

```
ansible-target-provisioner-v1.0.0/
├── inventory.ini          # Define your target servers (IPs, users, SSH keys)
├── ansible.cfg            # Ansible runtime configuration (SSH, privileges, etc.)
├── install_htop.yml       # Simple demo playbook: install htop package
├── setup_server.yaml      # Main playbook: full server provisioning workflow
├── README.md              # This guide
└── (optional) roles/      # Extend with reusable roles in future versions

```

---

## 📄 File Descriptions | ফাইল বিবরণ

### 1️⃣ `inventory.ini`
```ini
[target_servers]
192.168.0.63 ansible_user=cloud3
```
| Property | Description |
|----------|-------------|
| **Purpose** | Defines target hosts and connection parameters |
| **Modify When** | Adding new servers, changing SSH user, or using custom SSH keys |
| **Key Fields** | `ansible_user`, `ansible_ssh_private_key_file`, `ansible_port` |

> 💡 **Tip**: For multiple hosts, add each on a new line under the group name.

### 2️⃣ `ansible.cfg`
```ini
[defaults]
inventory = ./inventory.ini
remote_user = pd
host_key_checking = False
ansible_private_key_file = ~/.ssh/id_rsa

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```
| Section | Purpose |
|---------|---------|
| `[defaults]` | Global Ansible behavior settings |
| `[privilege_escalation]` | Controls sudo/elevation behavior |

> ⚠️ **Warning**: `host_key_checking = False` is for lab use only. Enable in production!

### 3️⃣ `install_htop.yml` (Demo Playbook)
```yaml
---
- name: Install htop on managed node
  hosts: paulco-desk
  become: yes
  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
    - name: Install htop package
      ansible.builtin.apt:
        name: htop
        state: present
```
| Use Case | Description |
|----------|-------------|
| 🧪 Learning | Understand basic playbook structure |
| 🧪 Testing | Verify Ansible connectivity and sudo access |
| 🧪 Template | Base for creating new simple task playbooks |

### 4️⃣ `setup_server.yaml` (Main Playbook) 🚀
A comprehensive playbook that performs:

```yaml
✅ System update & upgrade
✅ Install essential packages (openssh-server, net-tools, curl, wget, git, htop, vim)
✅ Enable & start SSH service
✅ Configure UFW firewall (allow OpenSSH)
✅ Prepare SSH key authentication (~/.ssh directory, authorized_keys)
✅ Harden sshd_config (disable root login, enable pubkey auth)
✅ Create/manage target user with passwordless sudo
✅ Network connectivity verification
```

| Section | Tasks Covered |
|---------|--------------|
| 🔧 System Prep | apt update, upgrade, package installation |
| 🔐 SSH Setup | Service enablement, config hardening, key auth |
| 🛡️ Security | UFW configuration, sshd security settings |
| 👤 User Mgmt | User creation, sudoers configuration |
| 🌐 Verification | IP check, ping test to control node |

---

## 🚀 Quick Start | দ্রুত শুরু

### Step 1: Clone the Repository

 Clone via HTTPS
```
git clone https://github.com/SumonPaul18/ansible.git
```
Show Directory List
```
ll
```
 Navigate to project directory
```
cd ansible-projects/ansible-provisioner/v1.0.0/
```

### Step 2: Configure Inventory
Edit `inventory.ini` with your target server details:
```
nano inventory.ini
```
```ini
[target_servers]
192.168.0.63 ansible_user=cloud3
# Add more servers as needed:
# 192.168.0.64 ansible_user=admin ansible_port=2222
```

### Step 3: Configure SSH Access

Generate SSH key (if not already done)
```
ssh-keygen -t ed25519 -C "ansible-control"
```
Copy public key to target node
```
ssh-copy-id cloud3@192.168.0.63
```
Test passwordless SSH
```
ssh cloud3@192.168.0.63 "echo 'SSH connection successful!'"
```

### Step 4: Verify Ansible Connectivity
Test connection to all hosts
```
ansible all -m ping
```
Expected output:
```
# 192.168.0.63 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### Step 5: Run the Main Playbook

Execute server setup playbook
```
ansible-playbook setup_server.yaml
```
 Run with verbose output for debugging
```
ansible-playbook setup_server.yaml -v
```
 Run with extra verbosity (show task details)
```
ansible-playbook setup_server.yaml -vvv
```
Run with Extra Variables

Override defaults at runtime:

```bash
ansible-playbook -i inventory.ini setup_server.yaml -e "target_user=admin control_node_ip=192.168.0.93"
```
Dry Run (Check Mode)

See what *would* change without applying:

```bash
ansible-playbook -i inventory.ini setup_server.yaml --check
```
Target Specific Hosts
```bash
ansible-playbook -i inventory.ini setup_server.yaml --limit 192.168.0.63
```

### Step 6: 🧪 Testing & Verification Checklist

After running `setup_server.yaml`, verify:

| Check | Command | Expected Result |
|-------|---------|----------------|
| SSH Access | `ssh cloud3@192.168.0.63` | Login without password |
| Sudo Access | `sudo whoami` | Returns `root` |
| Firewall | `sudo ufw status` | `Status: active`, `OpenSSH ALLOW` |
| Tools Installed | `which htop curl git` | Paths returned for each |
| SSH Config | `sudo grep -E "PermitRootLogin|PasswordAuth" /etc/ssh/sshd_config` | `PermitRootLogin no`, `PasswordAuthentication yes` (or `no` if hardened) |

#### Another way to verify

Check if htop is installed
```
ansible target_servers -a "which htop"
```
Check SSH service status
```
ansible target_servers -a "systemctl status ssh"
```
Check UFW status
```
ansible target_servers -a "ufw status verbose"
```
Verify sudo access for user
```
ansible target_servers -a "sudo whoami" -b
```

---
## ⚙️ Configuration Guide

### 🔧 Customizing Variables
Edit `setup_server.yaml` vars section:
```
nano setup_server.yaml
```
```yaml
vars:
  target_user: cloud3              # Change to your desired username
  control_node_ip: 192.168.0.93    # Update with your control node IP
```

### 🔐 SSH Key Configuration
If using a custom SSH key:
```
nano inventory.ini
```
```ini
# In inventory.ini
192.168.0.63 ansible_user=cloud3 ansible_ssh_private_key_file=~/.ssh/custom_key

# OR in ansible.cfg
ansible_private_key_file = ~/.ssh/custom_key
```

### 🌐 Multi-Environment Setup
Use group_vars for environment-specific configurations:
```
group_vars/
├── production.yml    # Production server settings
├── staging.yml       # Staging server settings
└── development.yml   # Dev/Lab server settings
```

Example `group_vars/production.yml`:
```yaml
---
# Production-specific variables
security_level: high
enable_auditd: true
backup_enabled: true
```

### 🔄 Running Specific Tasks Only
```bash
# List all tasks without executing
ansible-playbook setup_server.yaml --list-tasks

# Run only tasks tagged 'ssh'
ansible-playbook setup_server.yaml --tags ssh

# Skip tasks tagged 'firewall'
ansible-playbook setup_server.yaml --skip-tags firewall

# Start from a specific task
ansible-playbook setup_server.yaml --start-at-task="Configure UFW"
```

---

## 🧪 Usage Examples

### Example 1: Install a New Package
```
nano setup_server.yaml
```
```yaml
# Add to setup_server.yaml or create new playbook
- name: Install docker.io
  ansible.builtin.apt:
    name: docker.io
    state: present
    update_cache: yes
```

### Example 2: Deploy Configuration File
```yaml
- name: Deploy custom sshd_config
  ansible.builtin.template:
    src: templates/sshd_config.j2
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: '0600'
  notify: Restart SSH
```

### Example 3: Conditional Execution
```yaml
- name: Install monitoring tools (only for production)
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - prometheus-node-exporter
    - netdata
  when: environment == "production"
```

### Example 4: Run Ad-hoc Commands
```bash
# Check disk usage on all targets
ansible target_servers -a "df -h"

# Restart a service
ansible target_servers -m systemd -a "name=nginx state=restarted" -b

# Gather system facts
ansible target_servers -m setup | grep -i ansible_os_family
```

---

## 🛠️ Troubleshooting 
| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| ❌ `UNREACHABLE!` | SSH connection failed | Verify IP, user, SSH key, and firewall rules |
| ❌ `Authentication failed` | Wrong credentials or key | Check `ansible_user`, run `ssh-copy-id` again |
| ❌ `Missing sudo password` | become_ask_pass=True | Set `become_ask_pass: False` or use SSH key for sudo |
| ❌ `Module not found` | Python missing on target | Install Python: `sudo apt install python3-minimal` |
| ❌ `Permission denied` | Insufficient privileges | Ensure user has sudo access with NOPASSWD |
| ❌ Playbook runs but no changes | Idempotency working | Check `changed_when` and task conditions |

### Debug Mode
```bash
# Enable debug output
ansible-playbook setup_server.yaml -vvv

# Test connection with detailed output
ansible all -m ping -vvv

# Check Ansible configuration
ansible --version
ansible-config dump --only-changed
```

### Common Fixes
```bash
# Fix SSH key permissions
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# Reset known_hosts if host key changed
ssh-keygen -R 192.168.0.63

# Validate playbook syntax
ansible-playbook setup_server.yaml --syntax-check

# Test a single host
ansible-playbook setup_server.yaml --limit 192.168.0.63
```

---

## 🔒 Security Best Practices

### ✅ Do's
```yaml
# Use SSH keys instead of passwords
# Store secrets in Ansible Vault
# Limit sudo access to specific commands when possible
# Regularly update Ansible and system packages
# Use tags to control playbook execution scope
```

### ❌ Don'ts
```yaml
# Never commit plaintext passwords or private keys to Git
# Avoid using 'become: yes' globally if not needed
# Don't disable host_key_checking in production
# Avoid hardcoding IPs; use variables or dynamic inventory
```

### Using Ansible Vault for Secrets
```bash
# Create encrypted vault file
ansible-vault create secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Run playbook with vault password
ansible-playbook setup_server.yaml --ask-vault-pass

# OR use vault password file
ansible-playbook setup_server.yaml --vault-password-file ~/.vault_pass
```

Example `secrets.yml`:
```yaml
---
db_password: "{{ vault_db_password }}"
api_key: "{{ vault_api_key }}"
```

---

## 🤝 Contributing 

Contributions are welcome! Please follow these steps:

1. 🍴 Fork the repository
2. 🌿 Create a feature branch (`git checkout -b feature/amazing-feature`)
3. 💾 Commit your changes (`git commit -m 'Add some amazing feature'`)
4. 📤 Push to the branch (`git push origin feature/amazing-feature`)
5. 🔓 Open a Pull Request

### Contribution Guidelines
- ✅ Write clear, descriptive commit messages
- ✅ Add comments for complex logic
- ✅ Test playbooks in a lab environment first
- ✅ Update documentation for new features
- ✅ Follow YAML best practices (indentation, quotes, etc.)

---

## 👨‍💻 Author 

<div align="center">

**Sumon Paul**  
🔧 DevOps Engineer | Cloud & Infrastructure Automation  
🌐 [GitHub](https://github.com/SumonPaul18) • 💼 [LinkedIn](#) • ✉️ [Email](#)

*Building scalable infrastructure with code, one playbook at a time.*  

</div>

---