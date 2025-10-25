# 🛠️ Ansible Pre-flight Playbook

> **উদ্দেশ্য**: এক বা একাধিক Linux টার্গেট নোডকে Ansible ম্যানেজমেন্টের জন্য স্বয়ংক্রিয়ভাবে প্রস্তুত করা — SSH key-based authentication, sudo অ্যাক্সেস, সিকিউরিটি হার্ডেনিং এবং প্রয়োজনীয় প্যাকেজ ইনস্টল করে।

এই playbook টি বিশেষ করে **OpenStack (kolla-ansible)**, **Kubernetes**, বা যেকোনো DevOps ওয়ার্কফ্লোর আগে নোড প্রস্তুতির জন্য ডিজাইন করা হয়েছে।

---

## 📁 Directory Structure

```
ansible-prep/
├── README.md                     # এই ফাইল
├── inventory.ini                 # টার্গেট হোস্ট(গুলো) এবং প্রাথমিক ক্রেডেনশিয়াল
├── group_vars/
│   └── all.yml                   # গ্লোবাল ভ্যারিয়েবল (ইউজার, SSH key, UFW সেটিংস)
├── roles/
│   └── preflight/
│       ├── tasks/main.yml        # মূল কনফিগারেশন টাস্কসমূহ
│       └── handlers/main.yml     # সার্ভিস রিস্টার্ট হ্যান্ডলার (SSH)
└── site.yml                      # মূল playbook এন্ট্রি পয়েন্ট
```

### 🔍 ফাইল ব্যাখ্যা

| ফাইল | কাজ |
|------|------|
| `inventory.ini` | টার্গেট হোস্ট(গুলো) এবং প্রাথমিক SSH ক্রেডেনশিয়াল (শুধু প্রথমবারের জন্য) |
| `group_vars/all.yml` | কাস্টমাইজেবল সেটিংস: ইউজার নাম, SSH public key path, UFW enable/disable |
| `roles/preflight/tasks/main.yml` | মূল কাজ: প্যাকেজ ইনস্টল, ইউজার তৈরি, sudo অ্যাক্সেস, SSH key সেটআপ, sshd_config মডিফাই |
| `roles/preflight/handlers/main.yml` | SSH সার্ভিস রিস্টার্ট করে যখন sshd_config পরিবর্তন হয় |
| `site.yml` | playbook এর এন্ট্রি পয়েন্ট — `preflight` role কল করে |

---

## ⚙️ কাস্টমাইজেশন গাইড

### 1. **ইউজার নাম পরিবর্তন করতে**
- ফাইল: `group_vars/all.yml`
- পরিবর্তন:
  ```yaml
  preflight_user: your_custom_user
  ```

### 2. **SSH Public Key পরিবর্তন করতে**
- ডিফল্ট: `~/.ssh/id_rsa.pub` (control node থেকে)
- অন্য কী ব্যবহার করতে চাইলে:
  ```yaml
  preflight_ssh_pubkey: "{{ lookup('file', '/path/to/your/public.key') }}"
  ```
  অথবা সরাসরি স্ট্রিং দিন:
  ```yaml
  preflight_ssh_pubkey: "ssh-rsa AAAAB3NzaC1yc2E... user@host"
  ```

### 3. **UFW বন্ধ রাখতে চাইলে**
- ফাইল: `group_vars/all.yml`
- পরিবর্তন:
  ```yaml
  enable_ufw: false
  ```

### 4. **অতিরিক্ত পোর্ট অ্যালাউ করতে (UFW)**
- ফাইল: `roles/preflight/tasks/main.yml`
- নতুন টাস্ক যোগ করুন:
  ```yaml
  - name: Allow HTTP/HTTPS in UFW
    ufw:
      rule: allow
      port: "{{ item }}"
    loop: [80, 443]
    when: enable_ufw
  ```

---

## ▶️ Playbook রান করার গাইডলাইন

### প্রয়োজনীয় শর্ত
- **Control Node**: Ansible ইনস্টল থাকতে হবে (`sudo apt install ansible`)
- **Target Node**: শুধুমাত্র SSH পাসওয়ার্ড অ্যাক্সেস (প্রথমবারের জন্য)

### ধাপগুলো:

1. **SSH Key জেনারেট করুন** (যদি না থাকে):
   ```bash
   ssh-keygen -t rsa -b 4096
   ```

2. **Inventory আপডেট করুন** (`inventory.ini`):
   ```ini
   [target]
   192.168.0.94

   [target:vars]
   ansible_user=cloud3
   ansible_ssh_pass=your_actual_password  # শুধু প্রথমবার
   ```

3. **Playbook রান করুন**:
   ```bash
   cd ansible-prep
   ansible-playbook -i inventory.ini site.yml
   ```

4. **পরবর্তী ব্যবহারের জন্য**:
   - `inventory.ini` থেকে `ansible_ssh_pass` লাইন **মুছে ফেলুন**
   - এখন থেকে key-based auth কাজ করবে

> ✅ **সুরক্ষা টিপ**: প্রোডাকশনে `ansible_ssh_pass` ব্যবহার না করে `--ask-pass` অথবা `ansible-vault` ব্যবহার করুন।

---

## ✅ ভেরিফিকেশন: পরিবর্তনগুলো চেক করুন

Playbook সফলভাবে রান হওয়ার পর নিচের কমান্ডগুলো দিয়ে যাচাই করুন:

```bash
# 1. SSH key-based login কাজ করছে?
ssh cloud3@192.168.0.94 'echo "✅ Key auth works!"'

# 2. Password auth বন্ধ হয়েছে?
ssh cloud3@192.168.0.94 'grep "^PasswordAuthentication" /etc/ssh/sshd_config'
# আউটপুট: PasswordAuthentication no

# 3. NOPASSWD sudo কাজ করছে?
ssh cloud3@192.168.0.94 'sudo -n whoami'
# আউটপুট: root (কোনো পাসওয়ার্ড চাইবে না)

# 4. UFW চালু ও SSH অ্যালাউড?
ssh cloud3@192.168.0.94 'sudo ufw status verbose'

# 5. প্রয়োজনীয় প্যাকেজ ইনস্টল হয়েছে?
ssh cloud3@192.168.0.94 'dpkg -l | grep -E "openssh-server|git|htop"'
```

---

## ⚠️ সম্ভাব্য ত্রুটি ও সমাধান

| ত্রুটি | কারণ | সমাধান |
|--------|------|--------|
| **`UNREACHABLE! "Failed to connect to the host via ssh"`** | Target node-এ SSH চালু নেই, নেটওয়ার্ক ইস্যু, বা ভুল IP | - `ping 192.168.0.94` চেক করুন<br>- Target-এ `sudo systemctl status ssh` চালান<br>- Firewall/UFW চেক করুন |
| **`Authentication failed`** | ভুল পাসওয়ার্ড বা ইউজার নেই | - `inventory.ini`-এ সঠিক পাসওয়ার্ড দিন<br>- Target-এ `id cloud3` দিয়ে ইউজার আছে কিনা চেক করুন |
| **`python3 not found`** | Target-এ Python ইনস্টল নেই | Playbook প্রথমে `raw` মডিউল দিয়ে python3 ইনস্টল করে — তবে যদি তা ব্যর্থ হয়, তাহলে ম্যানুয়ালি ইনস্টল করুন: `sudo apt install -y python3` |
| **`sudo: a password is required`** | `cloud3` ইউজারের sudo অ্যাক্সেস নেই | Playbook স্বয়ংক্রিয়ভাবে sudoers ফাইল তৈরি করে — তবে যদি ব্যর্থ হয়, তাহলে ম্যানুয়ালি চেক করুন: `/etc/sudoers.d/cloud3_nopasswd` |
| **`Failed to restart ssh`** | `sshd_config` সিনট্যাক্স এরর | - `sudo sshd -t` দিয়ে config টেস্ট করুন<br>- Playbook অটো ব্যাকআপ নেয় (`/etc/ssh/sshd_config.bak`) — সেটি রিস্টোর করুন |
| **`lookup plugin (file) failed`** | Control node-এ `~/.ssh/id_rsa.pub` নেই | - `ssh-keygen` চালান<br>- অথবা `group_vars/all.yml`-এ সরাসরি public key দিন |

---

## 📌 নোট

- এই playbook **idempotent** — একাধিকবার চালালে কোনো ডুপ্লিকেট কনফিগারেশন তৈরি হবে না।
- প্রথমবার **password-based SSH** প্রয়োজন, পরে **key-based** হয়ে যায়।
- আপনার **OpenStack/kolla-ansible** ডিপ্লয়মেন্টের আগে এটি চালানো আদর্শ প্র্যাকটিস।

---

## 📬 সাপোর্ট

যদি আপনার ল্যাবে কোনো বিশেষ নেটওয়ার্ক সেটআপ (যেমন VLAN, ভার্চুয়াল IP, ইত্যাদি) থাকে, তাহলে `group_vars` বা `tasks`-এ সেগুলো যোগ করা যাবে।

> ✨ **Happy Automating!**  
> — আপনার DevOps ল্যাবের জন্য তৈরি করা হয়েছে।

--- 
