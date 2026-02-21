
# 🚀 Ansible Provisioner

> **Automated server setup and configuration toolkit**  
> *Prepare, secure, and manage Linux target nodes with Ansible — fast, repeatable, and production-ready.*

[![Ansible](https://img.shields.io/badge/Ansible-2.9+-red.svg)](https://ansible.com)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Stable-green.svg)]()

---

## 🎯 Purpose

This project automates the initial setup of Linux target servers using Ansible. It helps you:

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

## ⚙️ Prerequisites

Before you begin, ensure:

| Requirement | Details |
|-------------|---------|
| **Control Node** | Linux/macOS with Python 3.8+ and Ansible 2.9+ installed |
| **Target Nodes** | Ubuntu/Debian-based Linux servers with SSH access |
| **SSH Key** | Generated key pair (`~/.ssh/id_rsa`) on control node |
| **Network** | Control node can reach target nodes via IP/hostname |

### Install Ansible (Ubuntu/Debian)
```bash
sudo apt update && sudo apt install ansible -y
ansible --version  # Verify installation
```

### Generate SSH Key (if not already done)
```bash
ssh-keygen -t ed25519 -C "ansible-control"
# Press Enter for defaults (no passphrase for automation)
```

---

## 🚀 Quick Start Guide

### 1️⃣ Clone the Repository
```bash
git clone https://github.com/SumonPaul18/ansible.git
cd ansible/ansible-target-provisioner-v1.0.0
```

### 2️⃣ Configure Your Inventory
Edit `inventory.ini` to match your environment:
```ini
[target_servers]
192.168.0.63 ansible_user=cloud3 ansible_ssh_private_key_file=~/.ssh/id_rsa
# Add more servers like:
# 192.168.0.64 ansible_user=admin
# web-servers:
#   hosts:
#     192.168.0.100:
#     192.168.0.101:
```

### 3️⃣ Copy SSH Key to Target (One-Time Setup)
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub cloud3@192.168.0.63
```

### 4️⃣ Test Connectivity
```bash
ansible all -i inventory.ini -m ping
# Expected output:
# 192.168.0.63 | SUCCESS => { "changed": false, "ping": "pong" }
```

### 5️⃣ Run the Main Provisioning Playbook
```bash
ansible-playbook -i inventory.ini setup_server.yaml
```

### 6️⃣ (Optional) Run Demo Playbook: Install htop
```bash
# ⚠️ First, update install_htop.yml to use 'target_servers' instead of 'paulco-desk'
ansible-playbook -i inventory.ini install_htop.yml
```

### 7️⃣ Verify on Target Node
```bash
ssh cloud3@192.168.0.63
htop          # Should launch successfully
sudo ufw status  # Should show firewall active with OpenSSH allowed
exit
```

---

## 📄 File Descriptions & Customization Guide

### 🔹 `inventory.ini`
**Purpose**: Lists target servers and connection details.  
**Key fields**:
- `ansible_user`: SSH username on target
- `ansible_ssh_private_key_file`: Path to your private key
- Group names (e.g., `[target_servers]`) used in playbooks

✅ **Modify when**: Adding/removing servers, changing users, or using password auth.

---

### 🔹 `ansible.cfg`
**Purpose**: Sets Ansible defaults for this project.  
**Critical settings**:
```ini
[defaults]
inventory = ./inventory.ini          # Auto-loads your inventory
remote_user = pd                     # Fallback SSH user
host_key_checking = False            # ⚠️ Disable for labs; ENABLE in production!
ansible_private_key_file = ~/.ssh/id_rsa

[privilege_escalation]
become = True                        # Auto-use sudo
become_method = sudo
become_user = root
```

✅ **Modify when**: Changing SSH user, enabling host key checking, or adjusting parallelism (`forks`).

---

### 🔹 `install_htop.yml` (Demo Playbook)
**Purpose**: Simple example to install a single package.  
**Current issue**: Targets `paulco-desk` — update to match your inventory group:
```yaml
# Before:
hosts: paulco-desk

# After (recommended):
hosts: target_servers
```

✅ **Use for**: Learning playbook syntax, testing connectivity, or quick package deployment.

---

### 🔹 `setup_server.yaml` (Main Playbook)
**Purpose**: Full server provisioning workflow.  
**Key tasks**:
1. 🔄 Update & upgrade system packages
2. 🔐 Install & configure OpenSSH + essential tools
3. 🛡️ Enable UFW firewall with SSH access
4. 🔑 Prepare `~/.ssh` and `authorized_keys` for key-based auth
5. ⚙️ Harden `sshd_config` (disable root login, enable pubkey auth)
6. 👤 Ensure `cloud3` user exists with passwordless `sudo`
7. 🌐 Validate network connectivity (IP check + ping to control node)

✅ **Customize by editing**:
- `vars:` section → Change `target_user`, `control_node_ip`
- Package list in `apt: name:` → Add/remove tools
- `lineinfile` loops → Adjust SSH security policies
- Firewall rules → Open additional ports (e.g., `80`, `443`)

💡 **Pro Tip**: Uncomment `debug:` blocks to see task outputs during testing.

---

## 🔧 Advanced Configuration

### Run with Extra Variables
Override defaults at runtime:
```bash
ansible-playbook -i inventory.ini setup_server.yaml \
  -e "target_user=admin control_node_ip=192.168.0.93"
```

### Dry Run (Check Mode)
See what *would* change without applying:
```bash
ansible-playbook -i inventory.ini setup_server.yaml --check
```

### Target Specific Hosts
```bash
ansible-playbook -i inventory.ini setup_server.yaml --limit 192.168.0.63
```

### Verbose Output for Debugging
```bash
ansible-playbook -i inventory.ini setup_server.yaml -vvv
```

---

## 🧪 Testing & Verification Checklist

After running `setup_server.yaml`, verify:

| Check | Command | Expected Result |
|-------|---------|----------------|
| SSH Access | `ssh cloud3@192.168.0.63` | Login without password |
| Sudo Access | `sudo whoami` | Returns `root` |
| Firewall | `sudo ufw status` | `Status: active`, `OpenSSH ALLOW` |
| Tools Installed | `which htop curl git` | Paths returned for each |
| SSH Config | `sudo grep -E "PermitRootLogin|PasswordAuth" /etc/ssh/sshd_config` | `PermitRootLogin no`, `PasswordAuthentication yes` (or `no` if hardened) |

---

## 🛠️ Troubleshooting

| Issue | Solution |
|-------|----------|
| ❌ `UNREACHABLE!` | Check IP, SSH key, firewall, and `ansible_user` in `inventory.ini` |
| ❌ `Permission denied` | Ensure `become: yes` and user has `sudo` rights |
| ❌ `Failed to update apt cache` | Run `sudo apt update` manually on target first |
| ❌ SSH connection hangs | Temporarily set `host_key_checking = False` in `ansible.cfg` |
| ❌ Playbook syntax error | Validate with `ansible-playbook --syntax-check playbook.yml` |

🔍 **Enable debug output**: Uncomment `debug:` tasks in `setup_server.yaml` to inspect variables.

---

## 🤝 Contributing

Contributions welcome! To propose improvements:
1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-idea`
3. Commit changes with clear messages
4. Push and open a Pull Request

💡 Ideas: Add roles for Docker/Nginx, support RHEL/CentOS, add Molecule tests.

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for details.  
*Free to use, modify, and distribute — even in commercial projects.*

---

## 🙏 Acknowledgements

- Built with ❤️ for the DevOps community
- Inspired by Ansible best practices and idempotent design
- Special thanks to contributors testing in home labs & production

> 🌟 **Pro Tip**: Always test playbooks in a staging environment before production deployment!

---

*Made by [Sumon Paul](https://github.com/SumonPaul18) • [Report Issue](https://github.com/SumonPaul18/ansible/issues)*
```

---

## ✅ Why This README Works

| Feature | Benefit |
|---------|---------|
| 🎯 Clear purpose statement | Instantly tells users *what* and *why* |
| 📁 Visual project tree | Helps navigate files quickly |
| 🚀 Step-by-step quick start | Reduces onboarding friction |
| 🔧 Customization guidance | Empowers users to adapt safely |
| 🧪 Verification checklist | Builds confidence in results |
| 🛠️ Troubleshooting table | Saves time debugging common issues |
| 🤝 Contributing + License | Encourages collaboration & compliance |
| 🌐 Badges + emojis | Modern, scannable, professional look |

---

## ▶️ Next Steps for You

1. Rename folder:  
   ```bash
   mv target-node-ready-v1.0.0 ansible-target-provisioner-v1.0.0
   ```
2. Replace `README.md` with the content above
3. Fix `install_htop.yml`: change `hosts: paulco-desk` → `hosts: target_servers`
4. Commit & push:
   ```bash
   git add .
   git commit -m "docs: improve README + standardize folder name"
   git push origin main
   ```

Let me know if you'd like me to generate a Bengali version of this README for your tutorials or documentation! 🇧🇩✨