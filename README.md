# **Ansible: Configuration Management Tools বিস্তারিত গাইড**

![Ansible Logo](./src/images/ansible-1.png)
---
এই গাইডটি Ansible সম্পর্কে একটি বিস্তারিত বিবরণ দেয়, যা বেসিক থেকে অ্যাডভান্সড ব্যবহার পর্যন্ত সবকিছু কভার করে। এটি অনুসরণ করে Ansible এর সম্পূর্ণ সুবিধা নিতে পারবেন।
---

## **সূচীপত্র**

1. [Ansible কি?](#ansible-কি)
2. [Ansible কে তৈরি করেছে? এটি ওপেন সোর্স না পেইড?](#ansible-কে-তৈরি-করেছে)
3. [Ansible কেন ব্যবহার করব?](#ansible-কেন-ব্যবহার-করব)
4. [Ansible কখন ব্যবহার করব?](#ansible-কখন-ব্যবহার-করব)
5. [Ansible কিভাবে ব্যবহার করব?](#ansible-কিভাবে-ব্যবহার-করব)
6. [বাস্তব দৃষ্টান্ত: Ansible ব্যবহারের সুবিধা](#বাস্তব-দৃষ্টান্ত)
7. [Ansible এর মূল ধারণা](#ansible-এর-মূল-ধারণা)
8. [Ansible মডিউলের তালিকা](#ansible-মডিউলের-তালিকা)
9. [Ansible ইনস্টলেশন গাইডলাইন](#ansible-ইনস্টলেশন-গাইডলাইন)
10. [Ansible Configuration Files](#Ansible-Configuration-Files)
11. [Ansible এর বেসিক ব্যবহার](#ansible-এর-বেসিক-ব্যবহার)
12. [Ansible এর অ্যাডভান্সড ব্যবহার](#ansible-এর-অ্যাডভান্সড-ব্যবহার)

---

## **Ansible কি?** <a id="ansible-কি"></a>

Ansible হল একটি ওপেন-সোর্স অটোমেশন টুল যা কনফিগারেশন ম্যানেজমেন্ট, অ্যাপ্লিকেশন ডিপ্লয়মেন্ট, টাস্ক অটোমেশন এবং IT অর্কেস্ট্রেশনের জন্য ব্যবহৃত হয়। এটি আপনাকে একটি একক কন্ট্রোল মেশিন থেকে একাধিক সার্ভার (নোড) ম্যানেজ করতে সাহায্য করে যেখানে ম্যানেজ করা নোডগুলিতে অতিরিক্ত সফটওয়্যার ইনস্টল করার প্রয়োজন হয় না।

মূল বৈশিষ্ট্য:
- এজেন্টলেস: রিমোট সিস্টেমে এজেন্ট ইনস্টল করার প্রয়োজন নেই।
- SSH ব্যবহার করে যোগাযোগ।
- Python দিয়ে লেখা।
- টাস্ক ডিফাইন করার জন্য YAML সিনট্যাক্স ব্যবহার করা হয়।

---

## **Ansible কে তৈরি করেছে? এটি ওপেন সোর্স না পেইড?** <a id="ansible-কে-তৈরি-করেছে"></a>

Ansible তৈরি করেছেন Michael DeHaan ২০১২ সালে। এটি GNU General Public License (GPL) এর অধীনে ওপেন সোর্স। তবে Red Hat ২০১৫ সালে Ansible কে কিনে নেয় এবং একটি পেইড এন্টারপ্রাইজ সংস্করণ প্রদান করে, যার নাম **Ansible Tower** (বর্তমানে Red Hat Automation এর অংশ)।

---

## **Ansible কেন ব্যবহার করব?** <a id="ansible-কেন-ব্যবহার-করব"></a>

* **সরলতা:** এটি YAML ফাইল ব্যবহার করে কাজ করে, যা পড়া এবং লেখা খুব সহজ। জটিল স্ক্রিপ্টিং এর প্রয়োজন হয় না।
* **Agentless:** টার্গেট মেশিনে কোনো এজেন্ট ইনস্টল করার ঝামেলা নেই। শুধু SSH অ্যাক্সেস থাকলেই হয়।
* **ক্ষমতাশালী:** এটি দিয়ে ছোট টাস্ক থেকে শুরু করে জটিল অ্যাপ্লিকেশন ডেপ্লয়মেন্ট এবং পুরো সিস্টেম কনফিগারেশন অটোমেট করা যায়।
* **স্কেলেবিলিটি**: শত শত বা হাজার হাজার সার্ভার ম্যানেজ করা যায়।
* **আইডেম্পোটেন্সি**: প্লেবুক যতবারই চালানো হোক না কেন, কাঙ্খিত অবস্থা পৌঁছানো নিশ্চিত করে।
* **মডুলার:** এতে প্রচুর বিল্ট-ইন মডিউল আছে বিভিন্ন টাস্ক সম্পন্ন করার জন্য (যেমন: প্যাকেজ ইনস্টল করা, ফাইল কপি করা, সার্ভিস স্টার্ট/স্টপ করা ইত্যাদি)।
* **ক্রস-প্ল্যাটফর্ম**: Linux, Windows এবং ক্লাউড পরিবেশে কাজ করে।
* **সম্প্রদায় (Community):** এর একটি বড় এবং সক্রিয় কমিউনিটি রয়েছে, তাই সাহায্য সহজেই পাওয়া যায়।

---

## **Ansible কখন ব্যবহার করব?** <a id="ansible-কখন-ব্যবহার-করব"></a>

- প্যাচিং, ব্যাকআপ এবং ডিপ্লয়মেন্ট জাতীয় পুনরাবৃত্তিমূলক কাজ অটোমেট করতে।
- একাধিক সার্ভারে কনফিগারেশন ম্যানেজ করতে।
- ডেভেলপমেন্ট, টেস্টিং এবং প্রোডাকশন পরিবেশে অ্যাপ্লিকেশন ডিপ্লয় করতে।
- একাধিক সিস্টেম জড়িত জটিল ওয়ার্কফ্লো অর্কেস্ট্রেট করতে।
- Infrastructure as Code (IaC) বাস্তবায়নে।

---

## **Ansible কিভাবে ব্যবহার করব?** <a id="ansible-কিভাবে-ব্যবহার-করব"></a>

Ansible ব্যবহার করে **প্লেবুক** চালানো হয়, যা YAML ফরম্যাটে লেখা হয়। প্লেবুকে রিমোট হোস্টে কাজ সম্পাদন করার জন্য টাস্ক ডিফাইন করা হয়। এখানে একটি উচ্চ-স্তরের ওয়ার্কফ্লো:

1. **ইনভেন্টরি ফাইল**: টার্গেট হোস্টের তালিকা ডিফাইন করুন।
2. **প্লেবুক**: YAML ফরম্যাটে টাস্ক লিখুন।
3. **প্লেবুক চালানো**: `ansible-playbook` কমান্ড ব্যবহার করে প্লেবুক চালান।

---

## **বাস্তব দৃষ্টান্ত: Ansible ব্যবহারের সুবিধা** <a id="বাস্তব-দৃষ্টান্ত"></a>

### Ansible ছাড়া:
ধরুন, ৫০টি সার্ভার ম্যানুয়ালি ম্যানেজ করছেন:
- প্যাচ আপডেট করতে ঘণ্টার পর ঘণ্টা লাগবে।
- মানুষের ভুলের ঝুঁকি বেশি।
- সার্ভারগুলোর মধ্যে কোনো সামঞ্জস্যতা নেই।

### Ansible দিয়ে:
- সবগুলো সার্ভারে প্যাচ আপডেট করতে মিনিটের মধ্যে কাজ শেষ।
- সার্ভারের কনফিগারেশন সামঞ্জস্যপূর্ণ রাখা যায়।
- মানুষের ভুল কমায় এবং সময় বাঁচায়।

---

## **Ansible এর মূল ধারণা (Core Concepts)** <a id="ansible-এর-মূল-ধারণা"></a>

Ansible এর কিছু গুরুত্বপূর্ণ ধারণা বোঝা জরুরি:

1.  **Control Node:** এটি সেই সার্ভার যেখানে Ansible ইনস্টল করা থাকে এবং যেখান থেকে কমান্ড বা প্লেবুক চালানো হয়।
2.  **Managed Nodes:** এগুলো হলো সেই সার্ভার বা ডিভাইসগুলো যা Ansible দ্বারা ম্যানেজ করা হয়। এদেরকে "হোস্ট" বা "টার্গেট মেশিন"ও বলা হয়।
3.  **Inventory:** এটি একটি ফাইল যেখানে Managed Nodes এর লিস্ট এবং তাদের গ্রুপ সম্পর্কে তথ্য থাকে। Ansible জানতে পারে কোন মেশিনে কী কাজ করতে হবে Inventory ফাইলের মাধ্যমে। এটি INI বা YAML ফরম্যাটে হতে পারে।
    * উদাহরণ Inventory (`hosts` ফাইল):
        ```ini
        [webservers]
        web1.example.com
        web2.example.com

        [dbservers]
        db1.example.com

        [all:vars]
        ansible_python_interpreter=/usr/bin/python3
        ```
4.  **Modules:** এগুলো Ansible এর কার্যকারিতার ক্ষুদ্রতম ইউনিট। প্রতিটি মডিউল একটি নির্দিষ্ট কাজ করে (যেমন: `apt` প্যাকেজ ইনস্টল করার জন্য, `copy` ফাইল কপি করার জন্য, `file` ফাইল বা ডিরেক্টরি তৈরি/মুছে ফেলার জন্য)। Ansible Playbook বা Ad-hoc কমান্ড চালানোর সময় মডিউল ব্যবহার করা হয়। মডিউলগুলো Idempotent হয়, অর্থাৎ একই মডিউল বারবার চালালেও শেষ পর্যন্ত সার্ভার একই নির্দিষ্ট অবস্থায় পৌঁছাবে।
5.  **Tasks:** একটি টাস্ক হলো একটি মডিউল ব্যবহার করে কোনো নির্দিষ্ট কাজ করা। Playbook এ টাস্কগুলো লিস্ট আকারে লেখা থাকে।
    * উদাহরণ Task:
        ```yaml
        - name: Ensure apache is installed
          apt:
            name: apache2
            state: present
        ```
6.  **Playbooks:** এগুলো YAML ফাইল যা টাস্কের একটি তালিকা ধারণ করে। একটি প্লেবুক দ্বারা আপনি সার্ভারের একটি নির্দিষ্ট অবস্থা বর্ণনা করেন। Playbooks হলো Ansible এর মূল শক্তি, কারণ এগুলি একাধিক টাস্ককে একত্রে লজিক্যাল অর্ডারে এক্সিকিউট করতে দেয়।
    * উদাহরণ Playbook (`setup_webserver.yml`):
        ```yaml
        ---
        - name: Setup Web Server
          hosts: webservers
          become: yes # root প্রিভিলেজ ব্যবহার করার জন্য

          tasks:
            - name: Update apt cache
              apt:
                update_cache: yes

            - name: Install Apache web server
              apt:
                name: apache2
                state: present

            - name: Copy index.html
              copy:
                src: files/index.html # কন্ট্রোল নোডের ফাইল
                dest: /var/www/html/index.html # টার্গেট মেশিনের পাথ

            - name: Ensure Apache service is running
              service:
                name: apache2
                state: started
                enabled: yes
        ```
7.  **Roles:** জটিল প্লেবুকগুলোকে সংগঠিত করার জন্য Role ব্যবহার করা হয়। একটি Role এ Tasks, Handlers, Variables, Files, Templates ইত্যাদি আলাদা আলাদা ডিরেক্টরিতে রাখা হয়। এটি প্লেবুকগুলোকে আরও মডুলার, পুনঃব্যবহারযোগ্য এবং সহজে বোঝা যায়।

8. **Ad-Hoc Commands**: দ্রুত কাজ করার জন্য এক-লাইন কমান্ড।


---

## **Ansible মডিউলের তালিকা** <a id="ansible-মডিউলের-তালিকা"></a>

| **মডিউলের নাম** | **উদ্দেশ্য** |
|-------------------|--------------|
| `apt`            | Debian/Ubuntu সিস্টেমে প্যাকেজ ম্যানেজ করা। |
| `yum`            | CentOS/RHEL সিস্টেমে প্যাকেজ ম্যানেজ করা। |
| `copy`           | কন্ট্রোল নোড থেকে ম্যানেজড নোডে ফাইল কপি করা। |
| `file`           | ফাইল এবং ডিরেক্টরি ম্যানেজ করা। |
| `service`        | সার্ভিস শুরু/বন্ধ/রিস্টার্ট করা। |
| `user`           | ইউজার অ্যাকাউন্ট ম্যানেজ করা। |
| `ping`           | ম্যানেজড নোডের সাথে কানেক্টিভিটি টেস্ট করা। |
| `template`       | Jinja2 টেমপ্লেট ব্যবহার করে কনফিগারেশন ফাইল ডিপ্লয় করা। |
| `command`        | যেকোনো শেল কমান্ড চালানো। |
| `shell`          | পরিবেশ সমর্থন সহ শেল কমান্ড চালানো। |

---

## **Ansible ইনস্টলেশন গাইডলাইন** <a id="ansible-ইনস্টলেশন-গাইডলাইন"></a>

### **Ubuntu এ**
```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### **CentOS এ**
```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

ইনস্টলেশন যাচাই:
```bash
ansible --version
```

---

## **Ansible Configuration Files** <a id="Ansible-Configuration-Files"></a>

### **সেটআপ ফাইলের অবস্থান**
- মূল কনফিগারেশন ফাইল: `/etc/ansible/ansible.cfg`
- ডিফল্ট ইনভেন্টরি ফাইল: `/etc/ansible/hosts`

## **Ansible এর বেসিক ব্যবহার** <a id="ansible-এর-বেসিক-ব্যবহার"></a>

### **ইনভেন্টরি ফাইল কাস্টমাইজ করা**
`/etc/ansible/hosts` ফাইল সম্পাদনা করে আপনার ম্যানেজড নোড ডিফাইন করুন। উদাহরণ:
```ini
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```
---

## 🗂️ **Custom Inventory ফাইল তৈরি করা (custom inventory)**

একটি ফাইল তৈরি করুন `inventory` নামে:

```
nano inventory
```

```ini
[web]
192.168.56.101 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

- এখানে `192.168.56.101` হলো remote server এর IP
- `ansible_user` হলো SSH username (যেমন `ubuntu`)
- `ansible_ssh_private_key_file` হলো private key path

---

## 🔐 **Passwordless SSH Setup (ansible server থেকে Manage server-এ)**

### Step 1: SSH Keygen চালান:
```bash
ssh-keygen
```

Enter চেপে চেপে default সেটিং রাখুন।

### Step 2: Public key remote server-এ copy করুন:
```bash
ssh-copy-id ubuntu@192.168.56.101
```

এখন আপনি `ssh ubuntu@192.168.56.101` দিয়ে password ছাড়াই ঢুকতে পারবেন।

---

## ✅ **Connectivity টেস্ট করুন**

```bash
ansible -i inventory web -m ping
```

Output দেখতে এরকম হবে:
```json
192.168.56.101 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

```

---
## 🧠 **Ad-hoc Command Using**

Ansible Playbook না লিখেই, একবারের জন্য সরাসরি command line থেকে command run করাকে বলা হয় **Ad-hoc Command**।

👉 Syntax:
```bash
ansible <group/host> -i inventory -m <module_name> -a "<arguments>" [--become]
```

---

## 🔧 **1. ping module (check connection)**

```bash
ansible web -i inventory -m ping
```

---

## 🔧 **2. command module (default module)**

```bash
ansible web -i inventory -a "uptime"
```

অথবা স্পষ্ট করে:
```bash
ansible web -i inventory -m command -a "uptime"
```

✅ এটি remote server-এ command run করে।

---

## ⚠️ **3. shell module (command + shell features)**

```bash
ansible web -i inventory -m shell -a "echo $HOME"
```

> shell module দিয়ে আপনি shell feature (like variables, pipes) ব্যবহার করতে পারবেন।

---

## 📦 **4. apt module (Ubuntu/Debian package manager)**

```bash
ansible web -i inventory -m apt -a "name=nginx state=present update_cache=yes" --become
```

> `--become` ব্যবহার করতে হবে root access এর জন্য।

---

## 📋 **5. service module**

```bash
ansible web -i inventory -m service -a "name=nginx state=started" --become
```

---

## 📁 **6. copy module (local → remote file copy)**

```bash
ansible web -i inventory -m copy -a "src=./index.html dest=/var/www/html/index.html" --become
```

---

## 📂 **7. file module (permission, ownership, directory create)**

```bash
ansible web -i inventory -m file -a "path=/opt/testdir state=directory mode=0755 owner=ubuntu group=ubuntu" --become
```

---

## 👤 **8. user module (নতুন user তৈরি)**

```bash
ansible web -i inventory -m user -a "name=demo state=present" --become
```

---

## 🛠️ **9. Gathering System Info (setup module)**

```bash
ansible web -i inventory -m setup
```

> এটা পুরো system information আনবে: IP, hostname, memory, OS, etc.

---

## 🎓 **--become: Become root (sudo) permission**

যেসব module কাজ করতে root access লাগে, সেগুলোতে `--become` দিতে হয়।

```bash
ansible web -i inventory -m apt -a "name=htop state=present" --become
```

---

## 🧠 **Ansible Playbook Using**

Ansible Playbook হলো YAML ফরম্যাটে লেখা ফাইল যেখানে একাধিক task define করা হয়, sequentially run করার জন্য।

👉 এটা shell script এর automated version যা:
- Repeatable
- Human readable
- Git-এ রাখা যায়
- Documentation এর মতোই কাজ করে

---

## 🛠️ **Playbook Structure:**

```yaml
---
- name: Install and Start Nginx
  hosts: web
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true
```

---

## 📂 **Step-by-Step Breakdown:**

| লাইন | ব্যাখ্যা |
|------|----------|
| `---` | YAML ফাইল শুরু |
| `- name:` | এই Playbook বা Play এর নাম |
| `hosts:` | কোন host/inventory group এ কাজ হবে |
| `become: yes` | root access লাগবে |
| `tasks:` | এখানে শুরু হয় actual কাজ |
| `- name:` | প্রতিটা task এর readable নাম |
| `apt`, `service` | module ব্যবহার করে কাজ করা হয় |

---

## 🧪 **ব্যবহার করে দেখা (প্রথম Playbook):**

### 📝 `nginx_setup.yml` ফাইল তৈরি করুন:

```yaml
---
- name: Install and Configure Nginx
  hosts: web
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: true
```

---

### ▶️ Run করুন:

```bash
ansible-playbook -i inventory nginx_setup.yml
```

✅ আপনি দেখবেন ধাপে ধাপে সবকিছু হচ্ছে: apt update → nginx install → nginx start

---

## 🔥 **আরেকটা Real-Life Example: Static Website Deploy**

### 📝 `deploy_website.yml`:

```yaml
---
- name: Deploy Static Website
  hosts: web
  become: yes
  tasks:
    - name: Copy HTML file
      copy:
        src: ./index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---
## **Ansible এর অ্যাডভান্সড ব্যবহার** <a id="ansible-এর-অ্যাডভান্সড-ব্যবহার"></a>

## 🧠 **1. Ansible Variable Using**

Variable হলো কোনো reusable value, যেমন: `server_port`, `user_name`, `package_name` ইত্যাদি।  
Playbook বা inventory কে flexible করার জন্য এগুলো ব্যবহার করা হয়।

---

## ✅ **Variable Declare করার ৫টি জায়গা:**

| জায়গা                        | Priority |
|-----------------------------|----------|
| Playbook file (inline)      | Medium   |
| Inventory file (host/group) | Medium   |
| `host_vars/` & `group_vars/` | High     |
| Extra vars (`--extra-vars`) | Highest  |
| Role variables               | Medium   |

---

## 🛠️ **1. Playbook-এ Variable define ও ব্যবহার:**

```yaml
---
- name: Install a package using variable
  hosts: web
  become: yes
  vars:
    pkg_name: nginx
  tasks:
    - name: Install {{ pkg_name }}
      apt:
        name: "{{ pkg_name }}"
        state: present
```

---

## 🗂️ **2. Inventory ফাইল-এ Variable ব্যবহার:**

```ini
[web]
192.168.56.101 ansible_user=ubuntu http_port=8080
```

Playbook-এ ব্যবহার:
```yaml
- name: Show port number
  hosts: web
  tasks:
    - debug:
        msg: "HTTP Port is {{ http_port }}"
```

---

## 🗃️ **3. group_vars/host_vars ফোল্ডার:**

### Structure:
```
inventory/
├── hosts
├── group_vars/
│   └── web.yml
└── host_vars/
    └── 192.168.56.101.yml
```

📁 `group_vars/web.yml`:
```yaml
http_port: 80
server_admin: admin@example.com
```

📁 `host_vars/192.168.56.101.yml`:
```yaml
nginx_root: /var/www/html
```

---

## 🧠 **2. Facts কী?**

Ansible **automatically gather** করে যেকোনো host/server-এর hardware, software, OS, memory, IP address, CPU info ইত্যাদি।

📌 এগুলোকে বলা হয় **Facts**।

### Check করতে:
```bash
ansible web -m setup
```

---

## 🔍 **Example: Facts ব্যবহার করা**

```yaml
- name: Print IP Address using facts
  hosts: web
  tasks:
    - debug:
        msg: "The default IPv4 address is {{ ansible_default_ipv4.address }}"
```

আরো facts:
- `ansible_hostname`
- `ansible_os_family`
- `ansible_processor_cores`
- `ansible_memtotal_mb`

---

## 🧠 **3. Conditionals in Ansible**

যদি কোনো নির্দিষ্ট শর্ত থাকে, তাহলে সেই Task চলবে। না হলে skip করবে।

### ✅ Syntax:
```yaml
when: <condition>
```

### 📌 উদাহরণ ১: শুধুমাত্র Ubuntu হলে প্যাকেজ install করো

```yaml
- name: Install nginx only on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_facts['os_family'] == "Debian"
```

---

### 📌 উদাহরণ ২: Custom Variable Check

```yaml
vars:
  install_package: true

tasks:
  - name: Conditionally install htop
    apt:
      name: htop
      state: present
    when: install_package
```

---

## 🔁 **4. Loops in Ansible**

একটা task বারবার run করানো হয় **loops** দিয়ে।

### ✅ Syntax:
```yaml
with_items:
  - item1
  - item2
```
**বা এখনকার নতুন syntax:**
```yaml
loop:
  - item1
  - item2
```

---

### 📌 উদাহরণ ১: একাধিক প্যাকেজ ইনস্টল

```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - unzip
```

---

### 📌 উদাহরণ ২: একাধিক ফাইল কপি

```yaml
- name: Copy multiple files
  copy:
    src: "{{ item }}"
    dest: "/tmp/{{ item }}"
  loop:
    - file1.txt
    - file2.txt
```

---

## 🛎️ **5. Handlers in Ansible**

যখন কোনো Task "Changed" হয়, তখন এক বা একাধিক handler call করা হয়।

### ✅ Structure:
```yaml
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

> ✅ Notify only runs handler **if** that task made a **change**

---

## 🧪 **সম্পূর্ণ বাস্তব উদাহরণ: Condition + Loop + Handler**

```yaml
---
- name: Install apps & configure NGINX
  hosts: web
  become: yes
  vars:
    my_packages:
      - htop
      - tree
      - nginx
    install_nginx: true

  tasks:
    - name: Install all basic packages
      apt:
        name: "{{ item }}"
        state: present
      loop: "{{ my_packages }}"

    - name: Copy nginx config only if allowed
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      when: install_nginx
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```
---

## Templates (Jinja2) — বিশদ ও বাস্তব উদাহরণ 

Ansible Templates (Jinja2) হলো Ansible-এ dynamic configuration তৈরি করার সবথেকে শক্তিশালী উপায়। নিচে পুরো কনসেপ্ট, সিনট্যাক্স, advanced টেকনিক, best practices, এবং NGINX-এর পূর্ণ উদাহরণসহ দেয়া হলো — যেন আপনি প্র্যাকটিক্যাল প্রজেক্টে একে সরাসরি ব্যবহার করতে পারেন।

---

## 🔎 Template কেন ব্যবহার করবেন?

- একই কনফিগuration ফাইলটি ভিন্ন ভিন্ন host/role/env অনুযায়ী ভিন্ন মানে তৈরি করতে পারবেন।
    
- Playbook/role কে DI (dynamic & reusable) করে তোলে।
    
- Jinja2 templating দিয়ে loops, conditionals, filters ইত্যাদি ব্যবহার করে সহজে complex config বানানো যায়।
    

---

## 🧩 Jinja2—মূল ধারণা ও Syntaх

**Placeholders (Variable interpolation)**

```jinja
server_name {{ domain_name }};
```

**Expression / Filter**

```jinja
{{ some_var | default('value') | lower }}
```

**Condition**

```jinja
{% if ssl_enabled %}
listen 443 ssl;
{% else %}
listen 80;
{% endif %}
```

**Loop**

```jinja
upstream backend {
  {% for host in backend_servers %}
  server {{ host }};
  {% endfor %}
}
```

**Comments**

```jinja
{# This is a comment and won't appear in output #}
```

**Escape (to show `{{` literally)**

```jinja
{{ '{{ not_a_var }}' }}
```

---

## ✅ Where to put templates in a role/project

Best practice (role structure):

```
roles/
└── nginx/
    ├── templates/
    │   └── nginx.conf.j2
    ├── tasks/
    │   └── main.yml
    ├── defaults/
    │   └── main.yml
    └── handlers/
        └── main.yml
```

---

## 🔧 Example: `roles/nginx/templates/nginx.conf.j2` (complete, production-ready style)

```nginx
# Generated by Ansible - DO NOT EDIT MANUALLY
user www-data;
worker_processes {{ worker_processes | default(2) }};
pid /run/nginx.pid;

events {
    worker_connections 768;
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ## Gzip
    gzip {{ 'on' if gzip_enabled | default(false) else 'off' }};

    {% if upstream_servers is defined and upstream_servers | length > 0 %}
    upstream backend {
        {% for h in upstream_servers %}
        server {{ h }};
        {% endfor %}
    }
    {% endif %}

    server {
        listen {{ http_port | default(80) }}{% if ssl_enabled %} default_server{% endif %};
        {% if ssl_enabled %}
        listen {{ ssl_port | default(443) }} ssl;
        ssl_certificate {{ ssl_certificate_path }};
        ssl_certificate_key {{ ssl_certificate_key_path }};
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;
        {% endif %}

        server_name {{ server_name | default('_') }};
        root {{ document_root | default('/var/www/html') }};
        index index.php index.html index.htm;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass {{ php_fpm_socket | default('unix:/run/php/php7.4-fpm.sock') }};
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        access_log /var/log/nginx/{{ server_name | default('access') }}.log;
        error_log /var/log/nginx/{{ server_name | default('error') }}.log;
    }
}
```

**বর্ণনা:** উপরের টেমপ্লেটটি variable-driven: `worker_processes`, `gzip_enabled`, `upstream_servers` (list), `ssl_enabled`, `ssl_certificate_path`, `server_name`, `document_root`, `php_fpm_socket` ইত্যাদি ব্যবহার করে।

---

## 🔁 Playbook / Role: Template apply করার টাস্ক

`roles/nginx/tasks/main.yml` (টাস্ক অংশের উদাহরণ):

```yaml
---
- name: Ensure nginx package is installed
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Ensure document root exists
  file:
    path: "{{ document_root | default('/var/www/html') }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'

- name: Deploy nginx config from template
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Reload nginx

- name: Ensure nginx is enabled and started
  service:
    name: nginx
    state: started
    enabled: yes
```

`roles/nginx/handlers/main.yml`:

```yaml
---
- name: Reload nginx
  service:
    name: nginx
    state: reloaded
```

**বিঃদ্রঃ** Template টাস্ক `notify` ব্যবহার করলে কনফিগ পরিবর্তন হলে handler চালায় — এটা container/process restart/ reload control করতে খুবই দরকারি।

---

## 🔎 Variable management — কোথা থেকে কি আসবে

- `defaults/main.yml` — low-priority defaults
    
- `vars/main.yml` — higher priority (রোল-স্পেসিফিক)
    
- `group_vars/*` ও `host_vars/*` — environment / host specific
    
- Playbook inline `vars:` ব্লকে override করবেন
    
- CLI দিয়ে `--extra-vars` (`-e`) দিয়ে highest priority তে সেট করতে পারবেন
    

উদাহরণ `roles/nginx/defaults/main.yml`:

```yaml
worker_processes: 2
gzip_enabled: false
http_port: 80
ssl_port: 443
document_root: /var/www/html
php_fpm_socket: unix:/run/php/php7.4-fpm.sock
```

উপরের তালিকা environment অনুযায়ী `group_vars/production.yml`-এ override করুন (যেমন `gzip_enabled: true`, `worker_processes: 4`).

---

## 🔐 Secrets & Vault Integration

SSL key বা DB password ইত্যাদি টেমপ্লেটে আর সরাসরি লাগাবেন না — vault-এ রাখুন এবং `vars_files:` দিয়ে লোড করুন।

Playbook-এ:

```yaml
vars_files:
  - vault.yml     # encrypted with ansible-vault
```

টেমপ্লেটে:

```jinja
ssl_certificate {{ ssl_certificate_path }};
ssl_certificate_key {{ ssl_certificate_key_path }};
```

আর `ssl_certificate_path`/`ssl_certificate_key_path` থাকতে পারে vault-এ অথবা সেগুলো আপনার host-specific path এ থাকতে পারে (S3/secret manager থেকে deploy করলে path দিন)।

---

## 🧪 টেস্টিং ও ডিবাগিং

1. **Render locally (dry-run template rendering)**  
    আপনি Ansible দিয়ে template rendering test করতে পারেন:
    
    ```bash
    ansible localhost -m template -a "src=roles/nginx/templates/nginx.conf.j2 dest=/tmp/nginx.conf" -c local -e "server_name=example.com upstream_servers=['10.0.0.1:8080']"
    ```
    
    এর ফলে `/tmp/nginx.conf` পাবেন — সেখান থেকে আউটপুট পরীক্ষা করুন।
    
2. **`ansible-playbook --check --diff`**  
    Check-mode + diff দিয়ে দেখা যাবে কি পরিবর্তন হবে এবং কিভাবে টেমপ্লেট রেন্ডার হচ্ছে।
    
3. **`debug` module**  
    Playbook-এ `debug: var=some_var` ব্যবহার করে variable values চেক করুন।
    

---

## 🔧 Advanced techniques (practical scenarios)

### 1) Multiple server blocks (domains) — loop in template

```jinja
{% for v in vhosts %}
server {
    listen {{ v.port }};
    server_name {{ v.server_name }};
    root {{ v.root }};
    ...
}
{% endfor %}
```

`vhosts` হবে list of dicts:

```yaml
vhosts:
  - { server_name: "site1.example.com", port: 80, root: "/var/www/site1" }
  - { server_name: "site2.example.com", port: 80, root: "/var/www/site2" }
```

### 2) Conditional inclusion of snippets

```jinja
{% if enable_rate_limit %}
include /etc/nginx/snippets/rate_limit.conf;
{% endif %}
```

### 3) Use `lookup` to fetch file contents (e.g., include SSL cert inline — not recommended usually)

```jinja
ssl_certificate_data: "{{ lookup('file', '/etc/ssl/certs/example.crt') }}"
```

### 4) Filters & complex expressions

- `| default()` — set default if var missing
    
- `| join(',')` — join list to string
    
- `| ipaddr('public')` — (requires netaddr filter plugin) — for IP handling
    
- Use `| to_nice_json` for debugging complex data
    

---

## ⚠️ Common pitfalls & Troubleshooting

- **YAML / Jinja syntax errors**: spacing and indentation critical. Always validate with `ansible-lint` / `yamllint`.
    
- **Undefined variables**: use `{{ var | default('value') }}` or guard with `if var is defined`.
    
- **Permissions**: template dest `/etc/nginx/nginx.conf` owner & mode ঠিক রাখতে হবে; otherwise nginx fail।
    
- **Handlers not firing**: handler only fires if task reports `changed: true`. If template produces same content, handler won't run — that's desired.
    
- **Line endings**: Windows CRLF সমস্যা হলে `.gitattributes` দিয়ে LF force করুন।
    
- **Large templates**: break into smaller snippets and `include` them to keep readable.
    

---

## ✔︎ Best practices (practical checklist)

- Place templates under `roles/<role>/templates/`.
    
- Keep templates idempotent and deterministic.
    
- Use `defaults/main.yml` for safe defaults.
    
- Avoid embedding secrets; use Vault or external secret managers.
    
- Test templates locally with `ansible -m template` or `--check --diff`.
    
- Version-control templates; keep them small and modular (snippets).
    
- Use comments and header lines indicating generated-by-Ansible.
    

---

## 🔁 Full minimal example (quick copy-paste)

**1) roles/nginx/templates/nginx.conf.j2** — use previous nginx template.

**2) roles/nginx/tasks/main.yml**

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Reload nginx

- name: Ensure document root exists
  file:
    path: "{{ document_root | default('/var/www/html') }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
```

**3) roles/nginx/handlers/main.yml**

```yaml
- name: Reload nginx
  service:
    name: nginx
    state: reloaded
```

**4) playbook.yml**

```yaml
- name: Deploy nginx with template
  hosts: web
  become: yes
  vars:
    server_name: example.com
    document_root: /var/www/example
    upstream_servers:
      - 10.0.0.5:8080
      - 10.0.0.6:8080
    gzip_enabled: true
  roles:
    - nginx
```

Run:

```bash
ansible-playbook -i inventory playbook.yml
```

---

## 🧠 **6. Ansible Role কী?**

Role হলো Ansible-এর একটি best practice structure  
👉 যা task, variable, template, file, handler — সবকিছুকে **modular & reusable** করে তোলে।

🎯 আপনি একবার Role বানাবেন, তারপর সেটা যে কোনো project-এ import করে চালাতে পারবেন!

---

## 📁 **Role Structure:**

```bash
roles/
└── nginx_setup/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── files/
    │   └── index.html
    ├── vars/
    │   └── main.yml
    └── defaults/
        └── main.yml
```

---

## 🛠️ **Role তৈরি করার কমান্ড:**

```bash
ansible-galaxy init roles/nginx_setup
```

---

## 📝 **roles/nginx_setup/tasks/main.yml**

```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Copy index.html
  copy:
    src: index.html
    dest: /var/www/html/index.html

- name: Copy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx
```

---

## 🛎️ **roles/nginx_setup/handlers/main.yml**

```yaml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

---

## 📜 **Playbook থেকে Role Call:**

```yaml
---
- name: Deploy Web Server using Role
  hosts: web
  become: yes
  roles:
    - nginx_setup
```

📁 Playbook এর পাশেই `roles/` ফোল্ডার থাকবে।

---

## 🎯 **7. Variable & Template Support**

📁 `roles/nginx_setup/templates/nginx.conf.j2`:
```nginx
server {
    listen {{ http_port }};
    server_name localhost;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

📁 `roles/nginx_setup/defaults/main.yml`:
```yaml
http_port: 80
```

---

## 🔥 **Production Tips:**

| Tip | ব্যাখ্যা |
|-----|----------|
| `defaults/` | low-priority variables (can override later) |
| `vars/` | high-priority variables |
| `handlers/` | config পরিবর্তনে service restart/notify |
| `files/` | static file copy |
| `templates/` | dynamic file with Jinja2 variables |
| `meta/` | dependencies, author info (optional) |

---

## 💡 **Mini Project Idea:**

✅ একটি Role বানান `apache_setup` নামে  
✅ যা Apache install করে  
✅ index.html দেয়  
✅ template config দেয়  
✅ restart handler দেয়

---

## 🧠 **8. Ansible Vault কী?**

Ansible Vault ব্যবহার করে আপনি যেকোনো ফাইল (বিশেষ করে variables) কে encrypt করে রাখতে পারেন।  
👉 যাতে কেউ YAML ফাইল খুললেও sensitive data না দেখতে পায়।

---

## 🔐 **Vault ব্যবহার করার সময়ের উদাহরণ:**

- `db_password: mysecretpass`
- `api_token: 3fk2r23kfd...`
- `smtp_auth_password: yourpass123`

---

## 🛠️ **Vault ফাইল তৈরি (encrypt):**

```bash
ansible-vault create secrets.yml
```

➡️ এটি আপনাকে একটি পাসওয়ার্ড চাইবে  
➡️ এরপর vi editor-এ লিখুন:

```yaml
db_user: root
db_pass: VerySecret123
```

সেভ করলে পুরো ফাইল encrypted থাকবে!

---

## 🔓 **Vault ফাইল খোলা (decrypt করে read):**

```bash
ansible-vault view secrets.yml
```

---

## ✏️ **Vault ফাইল এডিট করা (edit):**

```bash
ansible-vault edit secrets.yml
```

---

## 🔁 **Existing ফাইল Encrypt করা:**

```bash
ansible-vault encrypt vars.yml
```

---

## 🔓 **Vault Remove করে Normal file করা (decrypt):**

```bash
ansible-vault decrypt secrets.yml
```

---

## 📜 **Playbook-এ Vault file ব্যবহার:**

📁 `secrets.yml` (encrypted)  
📁 `main.yml` playbook:

```yaml
---
- name: Use secure DB credentials
  hosts: db
  become: yes
  vars_files:
    - secrets.yml
  tasks:
    - debug:
        msg: "My DB password is {{ db_pass }}"
```

➡️ রান করার সময় আপনাকে পাসওয়ার্ড দিতে হবে:

```bash
ansible-playbook main.yml --ask-vault-pass
```

---

## 🔐 **Vault Password File (auto auth):**

পাসওয়ার্ড বারবার দিতে না চাইলে একটা text ফাইলে রেখে দিন:

📁 `vault_pass.txt`:
```
MySuperVaultPass
```

Then রান করুন:

```bash
ansible-playbook main.yml --vault-password-file vault_pass.txt
```

> ⚠️ কিন্তু এই ফাইল `gitignore` করে রাখতে ভুলবেন না!

---

## ✅ **Best Practices (Security Guide):**

| Practice | ব্যাখ্যা |
|----------|---------|
| Use `vault` for all passwords | Database, Token, Email password |
| Use `.gitignore` for `*.txt`, `*.vault` files | যাতে কখনও গিটে না যায় |
| Separate secret file per environment | ex: `prod_secrets.yml`, `dev_secrets.yml` |
| Use Ansible Roles with `vars_files` | Role-ভিত্তিক গোপন তথ্য ব্যবহার |
| Rotate your vault password | নিয়মিত পাসওয়ার্ড পরিবর্তন করুন |

---

# 📘 **Dynamic Inventory & Real-Time Use Cases **

---

## 🎯 **উদ্দেশ্য:**
- Static vs Dynamic Inventory
- Dynamic Inventory কী ও কেন দরকার
- AWS EC2 Dynamic Inventory Example
- Ansible Inventory Script বা Plugin
- বাস্তব উদাহরণে ব্যবহার

---

## 🧠 **1. Static Inventory vs Dynamic Inventory**

### 📄 Static Inventory Example:
```ini
[web]
192.168.1.10
192.168.1.11

[db]
192.168.1.20
```

👉 IP বা Hostname ম্যানুয়ালি লিখতে হয়।

---

### 🌐 Dynamic Inventory:
- Dynamic inventory script/প্লাগিন ব্যবহার করে সার্ভার লিস্ট **স্বয়ংক্রিয়ভাবে fetch** করা হয়।
- আপনার অনলাইন সার্ভার (AWS, Azure, GCP, VMware, etc.) থেকে live inventory নিয়ে কাজ করে।

---

## ⚙️ **2. AWS EC2 Dynamic Inventory Example (Plugin Method)**

### 🔧 ধাপ ১: প্লাগইন ইনস্টল
```bash
pip install boto boto3 botocore
```

### 🔧 ধাপ ২: AWS Credential সেট করুন:
```bash
export AWS_ACCESS_KEY_ID=YOUR_KEY
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET
```

### 🔧 ধাপ ৩: Inventory Plugin config ফাইল (aws_ec2.yml)

```yaml
plugin: aws_ec2
regions:
  - ap-south-1
keyed_groups:
  - key: tags.Name
    prefix: tag
filters:
  instance-state-name: running
```

> 🔐 `aws_ec2.yml` ফাইলটি Ansible 2.9+ ভার্সনে Plug-and-play!

---

## 🔎 **Run inventory check:**

```bash
ansible-inventory -i aws_ec2.yml --list
```

🔄 এটি আপনার AWS থেকে EC2 instance গুলোর তথ্য নিয়ে আসবে!

---

## 🔥 **3. Custom Dynamic Inventory Script**

Ansible আপনাকে Python/Bash দিয়ে নিজের custom script লেখার সুবিধা দেয়।

### ✅ স্ক্রিপ্ট শর্ত:
- JSON format output দিতে হবে
- `--list` দিয়ে inventory দেখাতে হবে

### উদাহরণ (Python):
```python
#!/usr/bin/env python3
import json

inventory = {
    "web": {
        "hosts": ["192.168.1.10", "192.168.1.11"],
    },
    "_meta": {
        "hostvars": {}
    }
}
print(json.dumps(inventory))
```

### রান করার সময়:
```bash
ansible -i my_inventory.py all -m ping
```

---

## 📦 **4. Real-Time Use Case: Deploying Code to AWS**

```yaml
---
- name: Deploy to dynamic AWS EC2
  hosts: tag_WebServer
  become: yes
  tasks:
    - name: Pull latest code
      git:
        repo: https://github.com/myapp/repo.git
        dest: /var/www/html
```

📌 এখানে `tag_WebServer` হচ্ছে EC2-এর tag থেকে বানানো group।

---

# 📘 **9. Ansible with CI/CD Pipelines (GitHub Actions & Jenkins)**

---

## 🎯 **উদ্দেশ্য:**
- CI/CD কীভাবে Ansible ব্যবহার করে
- GitHub Actions থেকে Ansible চালানো
- Jenkins থেকে Ansible job চালানো
- বাস্তব উদাহরণে deploy automation

---

## 🧠 **1. CI/CD + Ansible কীভাবে কাজ করে?**

CI/CD Pipeline যখন কোড push করে:
- Ansible automatically সার্ভারে গিয়ে config/app deploy করে
- Deployment becomes **fully automated** & repeatable
- No manual login needed anymore!

---

## 🚀 **2. GitHub Actions থেকে Ansible চালানো**

### 📁 `.github/workflows/deploy.yml`

```yaml
name: Deploy with Ansible

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repo
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: 3.10

    - name: Install Ansible
      run: |
        pip install ansible

    - name: Run Ansible Playbook
      run: |
        ansible-playbook playbook.yml -i inventory.ini --extra-vars "env=prod"
      env:
        ANSIBLE_HOST_KEY_CHECKING: "False"
```

✅ যদি আপনার সার্ভারে SSH key set করা থাকে, তাহলে workflow থেকেই deploy হবে!

---

### 🔐 **Security Tip:**
- Use **GitHub Secrets** for:
  - SSH_PRIVATE_KEY
  - VAULT_PASSWORD
- Inject those into GitHub Actions securely.

---

## 🧰 **3. Jenkins থেকে Ansible চালানো**

### 🔧 ধাপ ১: Jenkins Server-এ:
- Install Ansible: `sudo apt install ansible`
- Add SSH credentials via Jenkins UI

### 🔧 ধাপ ২: Job তৈরি করুন:
- Freestyle Project ➜ Build Steps ➜ Execute Shell

```bash
cd $WORKSPACE
ansible-playbook playbook.yml -i inventory.ini
```

### 📦 Optional:
- GitHub Webhook দিয়ে auto-trigger করুন

---

## 💡 **4. বাস্তব উদাহরণ: Auto Deploy Node.js App**

📁 `deploy_app.yml`
```yaml
---
- name: Deploy Node.js App
  hosts: app
  become: yes

  tasks:
    - name: Pull code
      git:
        repo: 'https://github.com/yourname/node-app.git'
        dest: /opt/app

    - name: Install dependencies
      npm:
        path: /opt/app

    - name: Restart app
      systemd:
        name: nodeapp
        state: restarted
```

---

# 📘 **End-to-End Ansible Automation Project**

---

## 🎯 উদ্দেশ্য:
- একটি end-to-end real-life automation use case
- Project structure কেমন হবে
- Inventory + Variables + Roles + Playbook use
- GitHub/GitLab/Jenkins থেকে deploy
- নিরাপত্তা ও স্কেলেবলতা নিশ্চিত করা

---

## 🧱 **Project Scenario: Deploy LEMP Stack (Linux, Nginx, MySQL, PHP) with Laravel App**

> **উদ্দেশ্য:**  
Ubuntu সার্ভারে LEMP stack configure করে, Laravel app deploy করা হবে — পুরোটাই Ansible দিয়ে, automated.

---

## 📁 **1. Project Folder Structure**

```
ansible-lemp-project/
├── inventory/
│   └── production.ini
├── group_vars/
│   └── all.yml
├── roles/
│   ├── nginx/
│   │   ├── tasks/
│   │   └── templates/
│   ├── mysql/
│   ├── php/
│   └── laravel/
├── playbook.yml
└── vault_pass.txt
```

---

## 📜 **2. Inventory File: `production.ini`**

```ini
[web]
192.168.56.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## 🔐 **3. Group Variables: `group_vars/all.yml`**

```yaml
db_user: root
db_pass: "{{ vault_db_pass }}"
app_repo: "https://github.com/yourname/laravel-app.git"
```

**Vault-encrypted file:** `vault.yml`  
```yaml
vault_db_pass: MySecretPass123
```

Encrypt it:
```bash
ansible-vault encrypt vault.yml
```

---

## 🧩 **4. Roles Overview**

### ✅ Role: `nginx`

- Install Nginx
- Configure VirtualHost
- Start service

### ✅ Role: `mysql`

- Install MySQL
- Create Database
- Secure root password

### ✅ Role: `php`

- Install PHP & extensions

### ✅ Role: `laravel`

- Clone app from GitHub
- Install composer dependencies
- Set file permissions
- Run `php artisan migrate`

---

## ▶️ **5. Main Playbook: `playbook.yml`**

```yaml
---
- name: Setup LEMP and deploy Laravel
  hosts: web
  become: yes
  vars_files:
    - group_vars/all.yml
    - vault.yml

  roles:
    - nginx
    - mysql
    - php
    - laravel
```

---

## 🚀 **6. Run the Project**

```bash
ansible-playbook -i inventory/production.ini playbook.yml --vault-password-file vault_pass.txt
```

---

## 🛡️ **7. Bonus: GitHub Actions Integration**

📁 `.github/workflows/deploy.yml`:

```yaml
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: pip install ansible
      - run: ansible-playbook -i inventory/production.ini playbook.yml --vault-password-file vault_pass.txt
```

---

# 📘  Docker Automation with Ansible

---

## 🎯 উদ্দেশ্য:
- Docker install & configure with Ansible
- Docker container deploy (nginx, redis etc.)
- Custom Docker image build করা
- Docker volume, network, env manage
- বাস্তব উদাহরণ: Laravel অ্যাপ Docker container-এ deploy

---

## 🧰 1. প্রয়োজনীয়তা:
- একটি Ubuntu Server (local/VM/cloud)
- Ansible installed on control machine
- Target server-এ SSH access

---

## 🧱 2. Project Structure:

```bash
docker-ansible/
├── inventory.ini
├── playbook.yml
└── roles/
    └── docker/
        ├── tasks/
        │   └── main.yml
        └── templates/
```

---

## 📄 3. Inventory File: `inventory.ini`

```ini
[servers]
192.168.56.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## 📝 4. Docker Role Task File: `roles/docker/tasks/main.yml`

```yaml
---
- name: Install required packages
  apt:
    name: [apt-transport-https, ca-certificates, curl, gnupg, lsb-release]
    update_cache: yes

- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Add Docker repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
    state: present

- name: Install Docker
  apt:
    name: docker-ce
    state: latest
    update_cache: yes

- name: Start and enable Docker
  service:
    name: docker
    state: started
    enabled: yes
```

---

## 🚀 5. Main Playbook: `playbook.yml`

```yaml
---
- name: Install Docker on Remote Server
  hosts: servers
  become: yes

  roles:
    - docker
```

---

## ▶️ 6. রান করুন:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

## 🧪 7. Practical Example: Run nginx container

```yaml
- name: Run Nginx Container
  hosts: servers
  become: yes
  tasks:
    - name: Pull nginx image
      docker_image:
        name: nginx
        source: pull

    - name: Run nginx container
      docker_container:
        name: nginx_web
        image: nginx
        state: started
        ports:
          - "80:80"
```

✅ Save above in a new playbook `nginx.yml` and run it:
```bash
ansible-playbook -i inventory.ini nginx.yml
```

---

## 💡 আরও Practical Ideas:
| Use Case              | Module           | What it does                          |
|-----------------------|------------------|---------------------------------------|
| Run Redis container   | `docker_container` | Run redis in background               |
| Build image from Dockerfile | `docker_image`    | Build custom Laravel app image        |
| Volume mount         | `docker_container` | Mount data from host                  |
| Docker Compose (later) | `community.docker.docker_compose` | Multi-container deployment           |

---

