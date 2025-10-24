# Ansible: Zero to Hero Guide

এই গাইডলাইনে, Ansible-এর সম্পূর্ণ বিশদ আলোচনা করা হবে। Ansible কি, কেন এবং কখন ব্যবহার করবো, তা থেকে শুরু করে কিভাবে ইনস্টল করবেন, কনফিগার করবেন এবং কিভাবে বেসিক থেকে অ্যাডভান্সড লেভেলে ব্যবহার করবেন - সবকিছু ধাপে ধাপে বিশদভাবে আলোচনা করা হবে। এছাড়াও বাস্তব উদাহরণ দেওয়া হবে যাতে আপনি সহজেই Ansible ব্যবহার করতে পারেন।

---

## Table of Contents
1. [What is Ansible?](#what-is-ansible)
2. [Why Use Ansible?](#why-use-ansible)
3. [When to Use Ansible?](#when-to-use-ansible)
4. [How to Install Ansible?](#how-to-install-ansible)
   - [On Ubuntu](#on-ubuntu)
   - [On CentOS](#on-centos)
5. [Basic Configuration](#basic-configuration)
6. [Basic Usage of Ansible](#basic-usage-of-ansible)
7. [Advanced Usage of Ansible](#advanced-usage-of-ansible)
8. [Real-Life Examples](#real-life-examples)

---

### What is Ansible? <a name="what-is-ansible"></a>
Ansible is an open-source automation tool used for configuration management, application deployment, task automation, and IT orchestration. It allows you to manage multiple servers from a single control machine without requiring any agents on the managed nodes.

Key Features:
- **Agentless**: No need to install additional software on the remote machines.
- **Idempotent**: Ensures that running the same playbook multiple times produces the same result.
- **YAML-Based Playbooks**: Easy to write and understand.
- **Inventory Management**: Define groups of hosts (servers) for targeted actions.

---

### Why Use Ansible? <a name="why-use-ansible"></a>
1. **Simplicity**: Ansible uses YAML for playbooks, which is human-readable and easy to learn.
2. **Scalability**: Manage thousands of servers with minimal effort.
3. **Automation**: Automate repetitive tasks like software installation, configuration changes, and deployments.
4. **Cross-Platform**: Works across Linux, Windows, and cloud environments.
5. **No Agents Required**: Uses SSH for communication, reducing overhead.

---

### When to Use Ansible? <a name="when-to-use-ansible"></a>
- To automate server provisioning and configuration.
- To deploy applications across multiple environments (Dev, QA, Prod).
- For managing cloud infrastructure (AWS, Azure, GCP).
- For orchestrating complex workflows involving multiple systems.
- When you need to enforce consistent configurations across servers.

---

### How to Install Ansible? <a name="how-to-install-ansible"></a>

#### On Ubuntu <a name="on-ubuntu"></a>
```bash
# Update package lists
sudo apt update

# Install software-properties-common for adding PPAs
sudo apt install software-properties-common

# Add Ansible PPA repository
sudo add-apt-repository --yes --update ppa:ansible/ansible

# Install Ansible
sudo apt install ansible
```

#### On CentOS <a name="on-centos"></a>
```bash
# Enable EPEL repository
sudo yum install epel-release

# Install Ansible
sudo yum install ansible
```

Verify Installation:
```bash
ansible --version
```

---

### Basic Configuration <a name="basic-configuration"></a>
After installing Ansible, configure it by editing the inventory file and setting up SSH keys.

1. **Inventory File** (`/etc/ansible/hosts`):
   Define your managed nodes here. Example:
   ```ini
   [webservers]
   192.168.1.10
   192.168.1.11

   [databases]
   192.168.1.20
   ```

2. **SSH Key Setup**:
   Ensure passwordless SSH access to all managed nodes.
   ```bash
   ssh-keygen
   ssh-copy-id user@192.168.1.10
   ```

3. **Test Connectivity**:
   ```bash
   ansible all -m ping
   ```

---

### Basic Usage of Ansible <a name="basic-usage-of-ansible"></a>
1. **Ad-Hoc Commands**:
   Run commands directly on remote hosts.
   ```bash
   ansible webservers -m command -a "uptime"
   ```

2. **Playbooks**:
   Create YAML files to define tasks. Example (`playbook.yml`):
   ```yaml
   - name: Install Nginx
     hosts: webservers
     become: yes
     tasks:
       - name: Install Nginx package
         apt:
           name: nginx
           state: present

       - name: Start Nginx service
         service:
           name: nginx
           state: started
   ```

Run the playbook:
```bash
ansible-playbook playbook.yml
```

---

### Advanced Usage of Ansible <a name="advanced-usage-of-ansible"></a>
1. **Roles**:
   Organize playbooks into reusable components called roles.
   Directory structure:
   ```
   roles/
   ├── common/
   │   ├── tasks/
   │   ├── handlers/
   │   ├── templates/
   │   └── vars/
   └── webserver/
       ├── tasks/
       ├── handlers/
       └── vars/
   ```

   Example `roles/webserver/tasks/main.yml`:
   ```yaml
   - name: Install Apache
     apt:
       name: apache2
       state: present
   ```

2. **Variables and Templates**:
   Use variables and Jinja2 templates for dynamic configurations.
   Example template (`nginx.conf.j2`):
   ```nginx
   server {
       listen {{ http_port }};
       server_name {{ domain_name }};
       root /var/www/html;
   }
   ```

3. **Handlers**:
   Trigger specific tasks only when notified.
   Example:
   ```yaml
   handlers:
     - name: Restart Nginx
       service:
         name: nginx
         state: restarted
   ```

4. **Vault**:
   Encrypt sensitive data using Ansible Vault.
   ```bash
   ansible-vault create secrets.yml
   ```

---

### Real-Life Examples <a name="real-life-examples"></a>
1. **Automated Server Provisioning**:
   Deploy a LAMP stack across multiple servers using a single playbook.

2. **Cloud Infrastructure Management**:
   Use Ansible modules to provision AWS EC2 instances or Azure VMs.

3. **CI/CD Pipeline Integration**:
   Automate application deployment using Ansible in Jenkins pipelines.

4. **Configuration Enforcement**:
   Ensure all servers comply with security policies (e.g., installing patches).

Example: Deploy a Python app on AWS EC2:
```yaml
- name: Deploy Python App
  hosts: aws_servers
  become: yes
  tasks:
    - name: Install Python
      apt:
        name: python3
        state: present

    - name: Copy App Files
      copy:
        src: ./app.py
        dest: /opt/app.py

    - name: Start App
      command: python3 /opt/app.py
```

---

This guide provides a comprehensive overview of Ansible, from installation to advanced usage. Practice these steps to become proficient in automating your infrastructure efficiently!

--- 

If you have further questions or need clarification, feel free to ask!