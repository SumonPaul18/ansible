
## ✅ **উন্নত ও ডাইনামিক প্লেবুক স্ট্রাকচার**

### 📁 Directory Structure (সরল ও মডুলার)
```
ansible-preflight/
├── README.md
├── inventory.ini
├── group_vars/
│   └── all.yml          # 👈 একমাত্র কনফিগ ফাইল — সব ভেরিয়েবল এখানে
├── site.yml
└── roles/
    └── preflight/
        └── tasks/
            └── main.yml
```

> 💡 **ভেরিয়েবল এক ফাইলে রাখাই ভালো** — কারণ:
> - সহজে কাস্টমাইজ করা যায়
> - এক জায়গায় সব কনফিগ
> - CI/CD বা টিম শেয়ারিংয়ে সুবিধা

---

## 📄 **1. `group_vars/all.yml` — সব কাস্টমাইজেশন এখানে**

```yaml
---
# ───────────────────────────────────────
# 🛠️  Preflight Playbook Configuration
# ───────────────────────────────────────

# 🔹 Target user to manage
preflight_user: cloud3

# 🔹 SSH public key (auto-loads from control node)
preflight_ssh_pubkey: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

# 🔹 Install packages? Set to false to skip
install_essential_packages: true
essential_packages:
  - openssh-server
  - net-tools
  - curl
  - wget
  - git
  - htop
  - vim
  # ➕ Add or remove packages here as needed

# 🔹 Configure UFW?
configure_ufw: false        # 👈 Set to true if you want UFW
ufw_rules:
  - name: OpenSSH
  # ➕ Add more rules: e.g., { port: 80, proto: tcp }

# 🔹 Enable passwordless sudo?
enable_passwordless_sudo: true

# 🔹 Disable password authentication after key setup?
disable_password_auth: true

# 🔹 Ensure python3 is installed (required for Ansible)?
ensure_python3: true
```

> ✅ এখন আপনি শুধু `all.yml` ফাইলটি এডিট করে:
> - UFW চালু/বন্ধ করতে পারবেন
> - প্যাকেজ যোগ/বাদ দিতে পারবেন
> - sudo, SSH auth সেটিংস টগল করতে পারবেন

---

## 📄 **2. `roles/preflight/tasks/main.yml` — কন্ডিশনাল টাস্কস**

```yaml
---
# ───────────────────────────────────────
# 📦 Ensure Python3 (minimal bootstrap)
# ───────────────────────────────────────
- name: Install python3 if missing (required for Ansible modules)
  raw: sudo apt update && sudo apt install -y python3
  when: ensure_python3 and (ansible_python_version is not defined)
  become: yes

# ───────────────────────────────────────
# 📥 Install essential packages (optional)
# ───────────────────────────────────────
- name: Install essential packages
  apt:
    name: "{{ essential_packages }}"
    state: present
    update_cache: yes
  when: install_essential_packages
  become: yes

# ───────────────────────────────────────
# 👤 Ensure target user exists
# ───────────────────────────────────────
- name: Ensure user exists
  user:
    name: "{{ preflight_user }}"
    shell: /bin/bash
    create_home: yes
  become: yes

# ───────────────────────────────────────
# 🔑 Set up SSH key-based access
# ───────────────────────────────────────
- name: Ensure .ssh directory exists
  file:
    path: "/home/{{ preflight_user }}/.ssh"
    state: directory
    owner: "{{ preflight_user }}"
    group: "{{ preflight_user }}"
    mode: '0700'

- name: Add SSH public key for Ansible access
  authorized_key:
    user: "{{ preflight_user }}"
    state: present
    key: "{{ preflight_ssh_pubkey }}"

# ───────────────────────────────────────
# 🛡️ Configure sudo (optional)
# ───────────────────────────────────────
- name: Set up passwordless sudo for user
  copy:
    content: "{{ preflight_user }} ALL=(ALL) NOPASSWD: ALL\n"
    dest: "/etc/sudoers.d/{{ preflight_user }}_nopasswd"
    owner: root
    group: root
    mode: '0440'
  when: enable_passwordless_sudo
  become: yes

# ───────────────────────────────────────
# 🔐 Harden SSH config (optional password auth disable)
# ───────────────────────────────────────
- name: Configure SSH daemon securely
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
    backup: yes
  loop:
    - { regexp: '^#?PermitRootLogin', line: 'PermitRootLogin no' }
    - { regexp: '^#?PubkeyAuthentication', line: 'PubkeyAuthentication yes' }
    - { regexp: '^#?AuthorizedKeysFile', line: 'AuthorizedKeysFile .ssh/authorized_keys' }
    - { regexp: '^#?UsePAM', line: 'UsePAM yes' }
    - { regexp: '^#?PasswordAuthentication', line: "PasswordAuthentication {{ 'no' if disable_password_auth else 'yes' }}" }
  notify: restart ssh
  become: yes

# ───────────────────────────────────────
# 🧱 Configure UFW (optional)
# ───────────────────────────────────────
- name: Configure UFW rules
  ufw:
    rule: allow
    name: "{{ item.name }}"
    port: "{{ item.port | default(omit) }}"
    proto: "{{ item.proto | default(omit) }}"
  loop: "{{ ufw_rules }}"
  when: configure_ufw
  become: yes

- name: Enable and start UFW
  ufw:
    state: enabled
    policy: deny
  when: configure_ufw
  become: yes

# ───────────────────────────────────────
# 🔄 Handlers
# ───────────────────────────────────────
- name: Restart SSH service
  systemd:
    name: ssh
    state: restarted
  become: yes
  listen: "restart ssh"
```

> ⚠️ **দ্রষ্টব্য**: `handlers` এখন inline (Ansible 2.10+ সাপোর্ট করে)।  
> অথবা আলাদা `handlers/main.yml` বানাতে চাইলে বলুন — কিন্তু এই স্ট্রাকচার ছোট প্লেবুকের জন্য আরও সহজ।

---

## 📄 **3. `site.yml`**
```yaml
---
- name: 🔧 Prepare target nodes for Ansible management
  hosts: all
  become: yes
  roles:
    - preflight
```

---

## 📄 **4. `inventory.ini` (সহজ উদাহরণ)**
```ini
[targets]
192.168.0.94

[targets:vars]
ansible_user=cloud3
ansible_ssh_pass=temp123  # Only needed for first run
```

> 🔄 প্রথম রানের পর `ansible_ssh_pass` মুছে ফেলুন।

---

# 📚 **README.md**

```markdown
# 🛠️ Ansible Preflight Playbook

এই playbook টি আপনার **target Linux নোডগুলোকে Ansible-এর জন্য প্রস্তুত** করবে।  
SSH key setup, sudo access, প্যাকেজ ইনস্টল, UFW কনফিগারেশন — সবকিছু **এক জায়গা থেকে কাস্টমাইজ** করা যাবে।

---

## 📁 Directory Structure

```
ansible-preflight/
├── README.md                 # এই ফাইল
├── inventory.ini             # টার্গেট নোডের IP ও লগিন তথ্য
├── group_vars/all.yml        # 👈 সব কনফিগারেশন এখানে (এডিট করুন এখানে)
├── site.yml                  # মূল playbook
└── roles/preflight/tasks/main.yml  # সব টাস্ক (পরিবর্তনের দরকার নেই)
```

### 🔍 ফাইলগুলোর কাজ:

| ফাইল | কাজ |
|------|------|
| `group_vars/all.yml` | **সব কাস্টমাইজেশন এখানে**: প্যাকেজ, UFW, sudo, SSH সেটিংস |
| `inventory.ini` | টার্গেট নোডের IP, ইউজার, পাসওয়ার্ড (শুধু প্রথমবার) |
| `site.yml` | playbook entrypoint — পরিবর্তনের দরকার নেই |
| `roles/preflight/tasks/main.yml` | লজিক — সাধারণত এডিট করার দরকার নেই |

---

## ⚙️ কীভাবে কাস্টমাইজ করবেন?

সব পরিবর্তন **শুধুমাত্র `group_vars/all.yml`** ফাইলে করুন:

### 1. **UFW বন্ধ রাখতে চান?**
```yaml
configure_ufw: false  # 👈 ডিফল্টে false
```

### 2. **প্যাকেজ যোগ/বাদ দিতে চান?**
```yaml
essential_packages:
  - openssh-server
  - curl
  - git
  # - htop    # কমেন্ট আউট করুন বাদ দিতে
  - nginx     # নতুন প্যাকেজ যোগ করুন
```

### 3. **Passwordless sudo চান না?**
```yaml
enable_passwordless_sudo: false
```

### 4. **পাসওয়ার্ড অথেন্টিকেশন বন্ধ করবেন না?**
```yaml
disable_password_auth: false
```

> ✅ কোনো কোড পরিবর্তন ছাড়াই সব কনফিগারেশন সম্ভব!

---

## ▶️ কীভাবে রান করবেন?

### প্রয়োজনীয় শর্ত:
- Control node-এ **Ansible** ইনস্টল থাকতে হবে
- Target node-এ **SSH password login** সক্রিয় থাকতে হবে (অস্থায়ী)

### ধাপসমূহ:

1. **SSH key জেনারেট করুন** (যদি না থাকে):
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **`inventory.ini` এ টার্গেট IP ও পাসওয়ার্ড দিন**:
   ```ini
   [targets]
   192.168.0.94

   [targets:vars]
   ansible_user=cloud3
   ansible_ssh_pass=your_temp_password
   ```

3. **`group_vars/all.yml` এ আপনার পছন্দমতো কনফিগ করুন**

4. **Playbook রান করুন**:
   ```bash
   ansible-playbook -i inventory.ini site.yml
   ```

5. **প্রথম রান শেষে**, `inventory.ini` থেকে `ansible_ssh_pass` লাইনটি **মুছে ফেলুন**।  
   এখন থেকে **SSH key** দিয়েই লগিন হবে।

---

## ✅ ভেরিফাই করুন: কী কী পরিবর্তন হয়েছে?

রান শেষে নিচের কমান্ডগুলো দিয়ে চেক করুন:

```bash
# 1. SSH key দিয়ে লগিন হয় কিনা?
ssh cloud3@192.168.0.94

# 2. sudo পাসওয়ার্ড ছাড়া কাজ করে কিনা?
ssh cloud3@192.168.0.94 "sudo -n whoami"  # আউটপুট: root

# 3. Password auth বন্ধ হয়েছে কিনা?
ssh cloud3@192.168.0.94 "grep 'PasswordAuthentication' /etc/ssh/sshd_config"

# 4. UFW চালু হয়েছে কিনা? (যদি configure_ufw=true হয়)
ssh cloud3@192.168.0.94 "sudo ufw status verbose"
```

---

## ❌ সম্ভাব্য ত্রুটি ও সমাধান

| ত্রুটি | কারণ | সমাধান |
|--------|------|--------|
| `Authentication failed` | ভুল পাসওয়ার্ড বা ইউজার নেই | `inventory.ini` চেক করুন; target-এ ইউজার আছে কিনা নিশ্চিত করুন |
| `python3 not found` | Target-এ Python নেই | `ensure_python3: true` রাখুন (ডিফল্টে true) |
| `Failed to connect to UFW` | UFW ইনস্টল নেই | `essential_packages`-এ `ufw` যোগ করুন অথবা `install_essential_packages: true` রাখুন |
| `Permission denied (publickey)` | SSH key সঠিকভাবে সেট হয়নি | Control node-এ `~/.ssh/id_rsa.pub` আছে কিনা চেক করুন |
| `sudo: a password is required` | sudoers ফাইল সঠিক নয় | `enable_passwordless_sudo: true` রাখুন এবং playbook রান করুন |

> 💡 **টিপ**: প্রথমবার `--ask-pass` ব্যবহার করে পাসওয়ার্ড ইনপুট নিন:
> ```bash
> ansible-playbook -i inventory.ini site.yml --ask-pass
> ```

---

## 🎯 ব্যবহারের উদাহরণ

### উদাহরণ 1: শুধু SSH key + sudo, কোনো প্যাকেজ/UFW নয়
```yaml
install_essential_packages: false
configure_ufw: false
enable_passwordless_sudo: true
```

### উদাহরণ 2: OpenStack নোডের জন্য (আপনার কেস)
```yaml
essential_packages:
  - openssh-server
  - python3
  - vim
  - git
  - curl
  - net-tools
configure_ufw: false   # OpenStack নিজে নেটওয়ার্ক ম্যানেজ করে
```

---

> ✨ **এই playbook টি আপনার DevOps/Cloud ল্যাব, OpenStack ডিপ্লয়মেন্ট, বা যেকোনো অটোমেশন প্রজেক্টের জন্য আদর্শ শুরুর পয়েন্ট!**
```

---

## 🎁 বোনাস: প্রথম রানের জন্য সেফটি টিপ

যদি `cloud3` ইউজার না থাকে, প্রথমে একটি সাদামাটা স্ক্রিপ্ট দিয়ে তৈরি করুন:

```bash
ssh root@192.168.0.94 "useradd -m -s /bin/bash cloud3 && echo 'cloud3:temp123' | chpasswd && usermod -aG sudo cloud3"
```

তারপর playbook রান করুন।

---

এই স্ট্রাকচার আপনার **8+ বছরের DevOps/Cloud অভিজ্ঞতার** সাথে পুরোপুরি ম্যাচ করবে — সহজ, নমনীয়, এবং পুনঃব্যবহারযোগ্য।  
চাইলে এটিকে **GitHub template** বা **Ansible Collection** হিসেবেও প্যাকেজ করে দিতে পারি।