## Ansible Lab Setup and Basic Usage

### Table of Contents

  - [Ansible Lab Setup and Basic Usage](https://www.google.com/search?q=%23ansible-lab-setup-and-basic-usage)
      - [Table of Contents](https://www.google.com/search?q=%23table-of-contents)
      - [1. Initial System Setup](https://www.google.com/search?q=%231-initial-system-setup)
      - [2. Ansible Installation](https://www.google.com/search?q=%232-ansible-installation)
      - [3. SSH Key Generation and Configuration](https://www.google.com/search?q=%233-ssh-key-generation-and-configuration)
      - [4. Ansible Project Setup](https://www.google.com/search?q=%234-ansible-project-setup)
      - [5. Inventory and Configuration Files](https://www.google.com/search?q=%235-inventory-and-configuration-files)
          - [5.1 `inventory.ini` (formerly `ansible.ini`)](https://www.google.com/search?q=%2351-inventoryini-formerly-ansibleini)
          - [5.2 `ansible.cfg`](https://www.google.com/search?q=%2352-ansiblecfg)
      - [6. SSH Key Copy to Managed Node](https://www.google.com/search?q=%236-ssh-key-copy-to-managed-node)
      - [7. Verifying Connectivity](https://www.google.com/search?q=%237-verifying-connectivity)
      - [8. Running a Playbook](https://www.google.com/search?q=%238-running-a-playbook)
          - [8.1 `install_htop.yml`](https://www.google.com/search?q=%2381-install_htopyml)
      - [9. Verification on Managed Node](https://www.google.com/search?q=%239-verification-on-managed-node)

-----

## 1\. Initial System Setup

Before installing Ansible, it's good practice to update your system's package lists and upgrade existing packages.

```bash
sudo apt update
apt upgrade
```

## 2\. Ansible Installation

Install Ansible using `apt`. The `-y` flag automatically answers "yes" to prompts.

```bash
apt install ansible -y
```

After installation, verify the installed version:

```bash
ansible --version
```

## 3\. SSH Key Generation and Configuration

Ansible relies on SSH for communication with managed nodes. Generate an SSH key pair.

```bash
ssh-keygen
```

(Press Enter for default locations and no passphrase unless you have specific security requirements.)

Confirm your current directory after key generation:

```bash
pwd
```

## 4\. Ansible Project Setup

Create a dedicated directory for your Ansible lab and navigate into it.

```bash
mkdir ~/ansible-lab
cd ~/ansible-lab/
```

Verify your current working directory:

```bash
pwd
```

## 5\. Inventory and Configuration Files

Ansible uses an inventory file to define the managed nodes and an `ansible.cfg` file for overall configuration.

### 5.1 `inventory.ini` (formerly `ansible.ini`)

Initially created as `ansible.ini`, this file was later renamed to `inventory.ini` for clarity, as it typically defines the inventory of hosts.

```bash
nano ansible.ini
mv ansible.ini inventory.ini
```

An example `inventory.ini` (content not shown in history, but implied by `ssh-copy-id` to `192.168.0.91`):

```ini
[paulco-lab]
192.168.0.91
```

### 5.2 `ansible.cfg`

This file configures Ansible's behavior. An example entry might point to your inventory file.

```bash
nano ansible.cfg
```

An example `ansible.cfg` might look like this:

```ini
[defaults]
inventory = ./inventory.ini
remote_user = pd
```

## 6\. SSH Key Copy to Managed Node

Copy your public SSH key to the managed node (`192.168.0.91` with user `pd`) to enable passwordless SSH authentication for Ansible.

```bash
ssh-copy-id pd@192.168.0.91
```

You can view your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

## 7\. Verifying Connectivity

Test connectivity to all hosts defined in your inventory using the `ping` module.

```bash
ansible all -m ping
```

Then, test connectivity to a specific host group (e.g., `paulco-lab`).

```bash
ansible paulco-lab -m ping
```

If you encounter issues, ensure you are in the correct directory where your `ansible.cfg` and `inventory.ini` (or `ansible.ini`) files are located. If you moved the files, navigate back to the correct path:

```bash
cd /root/ansible-lab
```

## 8\. Running a Playbook

Playbooks are YAML files that define a set of tasks to be executed on managed nodes.

### 8.1 `install_htop.yml`

This playbook installs `htop` on the `paulco-lab` group.

```bash
nano install_htop.yml
```

Example `install_htop.yml` content:

```yaml
---
- name: Install htop
  hosts: paulco-lab
  become: yes
  tasks:
    - name: Ensure htop is installed
      apt:
        name: htop
        state: present
```

Execute the playbook:

```bash
ansible-playbook install_htop.yml
```

You might run the playbook multiple times to ensure idempotency (running it again should show no changes if the software is already installed).

## 9\. Verification on Managed Node

After running the playbook, log into the managed node (`192.168.0.91`) and verify that `htop` is installed and executable.

```bash
ssh pd@192.168.0.91
htop

---
exit
```

-----

This `README.md` provides a good overview of the steps involved in setting up and using Ansible for basic system management.
