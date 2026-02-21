
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


---

## 🎁 বোনাস: প্রথম রানের জন্য সেফটি টিপ

যদি `cloud3` ইউজার না থাকে, প্রথমে একটি সাদামাটা স্ক্রিপ্ট দিয়ে তৈরি করুন:

```bash
ssh root@192.168.0.94 "useradd -m -s /bin/bash cloud3 && echo 'cloud3:temp123' | chpasswd && usermod -aG sudo cloud3"
```

তারপর playbook রান করুন।

---
