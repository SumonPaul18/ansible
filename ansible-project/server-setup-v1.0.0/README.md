এটি একটি চমৎকার এবং বাস্তবসম্মত অটোমেশন লক্ষ্য\! Ansible ব্যবহার করে আপনার বর্ণনা করা সম্পূর্ণ প্রক্রিয়াটি ধাপে ধাপে, স্বয়ংক্রিয় রিস্টার্ট সহ, একটি একক কমান্ডের মাধ্যমে সম্পন্ন করা সম্ভব।

এই সেটআপের জন্য আমরা একটি **Ansible প্রজেক্ট ডিরেক্টরি** তৈরি করব, যেখানে `inventory`, `playbook` এবং প্রয়োজনীয় `vars` ফাইল থাকবে। রিবুটের পরে প্লেবুকটি স্বয়ংক্রিয়ভাবে চালিয়ে যাওয়ার জন্য আমরা একটি উন্নত **স্টেট ট্র্যাকিং লজিক** ব্যবহার করব।

-----

### **Ansible প্রজেক্ট সেটআপ: সম্পূর্ণ গাইডলাইন (রিবুট হ্যান্ডলিং সহ)**

আমরা আপনার বেস মেশিনকে (যেখানে Ansible ইনস্টল করা আছে) এবং একটি রিমোট মেশিনকে (যদি থাকে) লক্ষ্য করব।

### **ধাপ ১: প্রজেক্ট ডিরেক্টরি তৈরি**

আপনার বেস মেশিনে একটি প্রজেক্ট ডিরেক্টরি তৈরি করুন:

```bash
mkdir my_ultimate_setup
cd my_ultimate_setup
mkdir vars
```

### **ধাপ ২: ইনভেন্টরি ফাইল (`inventory.ini`) তৈরি**

এই ফাইলটি আপনার টার্গেট মেশিনগুলি সংজ্ঞায়িত করবে। আমরা একটি গ্রুপ তৈরি করব, যেমন `all_servers`।

`inventory.ini` ফাইলটির বিষয়বস্তু:

```ini
# inventory.ini

# বেস মেশিন (যেখানে Ansible চলছে) - লোকাল কানেকশন ব্যবহার করে
[local_base]
localhost ansible_connection=local

# আপনার রিমোট সার্ভার/মেশিন
[remote_servers]
# remote_server_1 ansible_host=আপনার_রিমোট_আইপি_বা_হোস্টনেম ansible_user=ssh_ইউজারনেম

# সমস্ত টার্গেট মেশিনকে একই টাস্কে গ্রুপ করতে
[all_servers:children]
local_base
# remote_servers
```

> ⚠️ **দ্রষ্টব্য:** রিমোট সার্ভার ব্যবহারের জন্য `# remote_server_1` লাইনটি আনকমেন্ট করুন এবং আপনার আইপি/ইউজারনেম দিন। রিমোট মেশিনের জন্য SSH-পাসওয়ার্ডহীন লগইন সেটআপ করা আবশ্যক।

### **ধাপ ৩: ভ্যারিয়েবল ফাইল (`vars/config.yml`) তৈরি**

এই ফাইলটিতে আপনার মেশিনের জন্য কনফিগারেশন ভ্যালু থাকবে।

`vars/config.yml` ফাইলটির বিষয়বস্তু:

```yaml
# vars/config.yml

# --- গ্লোবাল কনফিগারেশন ---
new_hostname: "ansible-configured-host"
ssh_user: "আপনার_ssh_ইউজারনেম" # আপনার বেস মেশিন এবং রিমোট মেশিনের জন্য
reboot_flag_file: "/tmp/ansible_setup_in_progress" # স্টেট ট্র্যাকিং এর জন্য ফ্ল্যাগ ফাইল

# --- নেটওয়ার্ক কনফিগারেশন (netplan-এর জন্য) ---
# যদি আপনি static IP সেট করতে না চান, তাহলে এই অংশটি বাদ দিতে পারেন।
# মনে রাখবেন: Netplan শুধু Ubuntu-তে কাজ করে।
network_interface: "eth0" # আপনার নেটওয়ার্ক ইন্টারফেসের নাম দিন (যেমন: eth0, ens18, enp0s3)
static_ip_address: "192.168.1.10/24"
gateway_ip: "192.168.1.1"
dns_servers:
  - "8.8.8.8"
  - "8.8.4.4"
```

### **ধাপ ৪: হোস্ট ফাইল টেমপ্লেট (`hosts.j2`) তৈরি**

`/etc/hosts` ফাইলটি আপডেট করার জন্য একটি Jinja2 টেমপ্লেট ব্যবহার করব।

`hosts.j2` ফাইলটির বিষয়বস্তু:

```jinja2
127.0.0.1	localhost
127.0.1.1	{{ new_hostname }}

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Custom entries added by Ansible
{{ ansible_default_ipv4.address }}	{{ new_hostname }}
```

### **ধাপ ৫: প্রধান প্লেবুক (`setup.yml`) তৈরি**

এই প্লেবুকটি সমস্ত ধাপ পরিচালনা করবে, যার মধ্যে শর্তসাপেক্ষ রিবুট এবং পুনরায় কাজ শুরু করা অন্তর্ভুক্ত।

`setup.yml` ফাইলটির বিষয়বস্তু:

```yaml
---
# ==============================================================================
# Play 1: Pre-Reboot Tasks (Hostname, IP, OS Update/Upgrade)
# ==============================================================================
- name: Stage 1: Pre-Reboot Configuration and System Update
  hosts: all_servers # local_base এবং remote_servers
  become: yes
  vars_files:
    - vars/config.yml

  tasks:
    - name: Check if setup is already in Post-Reboot phase
      ansible.builtin.stat:
        path: "{{ reboot_flag_file }}"
      register: reboot_flag_status

    - name: Skip Stage 1 tasks if flag file exists (Go to Stage 2)
      ansible.builtin.meta: end_play
      when: reboot_flag_status.stat.exists

    # ------------------ 1.1 Hostname Change ------------------
    - name: Change hostname
      ansible.builtin.hostname:
        name: "{{ new_hostname }}"

    # ------------------ 1.2 /etc/hosts Edit ------------------
    - name: Edit /etc/hosts file with new hostname
      ansible.builtin.template:
        src: hosts.j2
        dest: /etc/hosts
        owner: root
        group: root
        mode: '0644'

    # ------------------ 1.3 Static IP Setup (Netplan) ------------------
    - name: Configure Static IP using Netplan (Ubuntu/Debian)
      ansible.builtin.template:
        src: netplan.j2
        dest: "/etc/netplan/01-static-net.yaml"
        owner: root
        group: root
        mode: '0600'
      when: ansible_distribution == 'Ubuntu' or ansible_distribution == 'Debian'
      # Notifying netplan to apply changes

    - name: Apply Netplan configuration
      ansible.builtin.command: netplan apply
      when: ansible_distribution == 'Ubuntu' or ansible_distribution == 'Debian'

    # ------------------ 1.4 System Update/Upgrade ------------------
    - name: OS Update and Upgrade (Kernel updates will require reboot)
      ansible.builtin.apt:
        upgrade: dist
        update_cache: yes
      register: apt_update_result

    # ------------------ 1.5 Reboot Handler ------------------
    - name: Create flag file to track post-reboot continuation
      ansible.builtin.copy:
        content: "STAGE_1_COMPLETE"
        dest: "{{ reboot_flag_file }}"
        mode: '0600'
      when: apt_update_result.changed

    - name: Initiate system reboot and wait for return
      ansible.builtin.reboot:
        reboot_timeout: 600
      when: apt_update_result.changed

    - name: If no reboot was needed, ensure flag file is removed for next run
      ansible.builtin.file:
        path: "{{ reboot_flag_file }}"
        state: absent
      when: not apt_update_result.changed

# ==============================================================================
# Play 2: Post-Reboot Tasks (Install Software)
# ==============================================================================
- name: Stage 2: Post-Reboot Software Installation (Python, Nginx, Docker)
  hosts: all_servers
  become: yes
  vars_files:
    - vars/config.yml

  # Clean up the flag file as the first task in the post-reboot stage
  pre_tasks:
    - name: Cleanup: Remove the reboot flag file
      ansible.builtin.file:
        path: "{{ reboot_flag_file }}"
        state: absent
      when: reboot_flag_status.stat.exists # Only remove if it was found in Stage 1

  tasks:
    - name: Install Python 3 and development headers
      ansible.builtin.apt:
        name:
          - python3
          - python3-pip
          - python3-dev
        state: present

    - name: Install Nginx web server
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Ensure Nginx service is running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes

    - name: Install Docker dependencies
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg-agent
          - software-properties-common
        state: present

    - name: Add Docker's official GPG key
      ansible.builtin.apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker APT repository
      ansible.builtin.apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present

    - name: Install Docker Engine
      ansible.builtin.apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present
        update_cache: yes

    - name: Add current user to the docker group
      ansible.builtin.user:
        name: "{{ ssh_user }}"
        groups: docker
        append: yes

    - name: Final Success Message
      ansible.builtin.debug:
        msg: "✅ All configurations, updates, and software installations (Python, Nginx, Docker) are complete!"
```

### **ধাপ ৬: নেটপ্ল্যান টেমপ্লেট (`netplan.j2`) তৈরি**

এটি `vars/config.yml`-এর ভেরিয়েবল ব্যবহার করে একটি `Netplan` কনফিগারেশন ফাইল তৈরি করবে।

`netplan.j2` ফাইলটির বিষয়বস্তু:

```jinja2
# netplan.j2

network:
  version: 2
  renderer: networkd
  ethernets:
    {{ network_interface }}:
      dhcp4: no
      addresses:
        - {{ static_ip_address }}
      gateway4: {{ gateway_ip }}
      nameservers:
          addresses: {{ dns_servers }}
```

### **ধাপ ৭: সম্পূর্ণ প্রজেক্ট স্ট্রাকচার**

আপনার প্রজেক্ট ডিরেক্টরি এখন দেখতে এমন হবে:

```
my_ultimate_setup/
├── inventory.ini
├── hosts.j2
├── netplan.j2
├── setup.yml
└── vars/
    └── config.yml
```

### **ধাপ ৮: একক কমান্ডের মাধ্যমে স্বয়ংক্রিয়ভাবে রান করা**

প্রথমবার প্লেবুকটি রান করার জন্য এই কমান্ডটি ব্যবহার করুন। এই কমান্ডটি যদি রিবুটের প্রয়োজন হয়, তবে এটি স্বয়ংক্রিয়ভাবে রিবুট করবে এবং প্রথম প্লেটি শেষ করবে।

```bash
# প্রথমবার প্লেবুক রান করুন
ansible-playbook -i inventory.ini setup.yml -k -K
```

  * `-i inventory.ini`: ইনভেন্টরি ফাইল নির্দিষ্ট করে।
  * `-k`: SSH পাসওয়ার্ড (রিমোট হোস্টের জন্য) জিজ্ঞাসা করে।
  * `-K`: `sudo` পাসওয়ার্ড (আপনার বেস মেশিন এবং রিমোট মেশিনে `become: yes` এর জন্য) জিজ্ঞাসা করে।

#### **রিবুটের পরে স্বয়ংক্রিয়ভাবে চালিয়ে যাওয়ার প্রক্রিয়া:**

1.  **Stage 1** রান হবে এবং OS আপডেট করবে।
2.  যদি কোনো কার্নেল আপডেট বা রিবুট-প্রয়োজনীয় আপডেট হয়, তাহলে `apt_update_result.changed` ট্রু হবে।
3.  Ansible একটি ফ্ল্যাগ ফাইল (`/tmp/ansible_setup_in_progress`) তৈরি করবে এবং **সিস্টেমকে রিবুট করবে**।
4.  রিবুটের পরে, আপনি **একই কমান্ডটি আবার** চালাবেন:

<!-- end list -->

```bash
# রিবুটের পর একই কমান্ড আবার চালান
ansible-playbook -i inventory.ini setup.yml -k -K
```

**কি ঘটবে:**

  * প্লেবুকটি রান হবে।
  * **Stage 1** এর শুরুতে এটি `/tmp/ansible_setup_in_progress` ফ্ল্যাগ ফাইলটি খুঁজে পাবে।
  * `ansible.builtin.meta: end_play` টাস্কটি ট্রিগার হবে, এবং **Stage 1 এর সব টাস্ক স্কিপ হয়ে যাবে**।
  * প্লেবুকটি **Stage 2** (Post-Reboot Tasks) এ চলে যাবে।
  * Stage 2 এর `pre_tasks` ফ্ল্যাগ ফাইলটি মুছে ফেলবে।
  * এরপর এটি Python, Nginx এবং Docker ইনস্টল করবে।
  * সমস্ত কাজ শেষ হবে।

### **একক কমান্ডে GitHub থেকে সোর্স ডাউনলোড ও রান (উচ্চ স্তরের)**

যদি আপনি এই সম্পূর্ণ সেটআপটি একটি একক কমান্ড দিয়ে GitHub থেকে ডাউনলোড করে চালাতে চান, তাহলে আপনি একটি ছোট **শেল স্ক্রিপ্ট** ব্যবহার করে পুরো প্রক্রিয়াটিকে একটি প্যাকেজে নিয়ে আসতে পারেন।

**`run_setup.sh` (উদাহরণ):**

```bash
#!/bin/bash
PROJECT_NAME="my_ultimate_setup"
REPO_URL="https://github.com/your_username/$PROJECT_NAME.git" # আপনার গিটহাব রিপোজিটরি দিন

echo "--- 1. Cloning Ansible Project from GitHub ---"
if [ ! -d "$PROJECT_NAME" ]; then
    git clone $REPO_URL
fi
cd $PROJECT_NAME

echo "--- 2. Starting Ansible Setup Playbook ---"
# -k, -K Flags: Prompts for SSH and sudo passwords if needed.
# If you use SSH keys and passwordless sudo, you can omit these.

ansible-playbook -i inventory.ini setup.yml -k -K

# Rerun the playbook to complete post-reboot tasks
if [ $? -eq 0 ] && [ -f /tmp/ansible_setup_in_progress ]; then
    echo "--- System requires reboot. Please reboot the machine (sudo reboot now) ---"
    echo "--- After reboot, run THIS SCRIPT AGAIN to continue Stage 2! ---"
    exit 0 # Exit the script, expecting a manual reboot

    # ALTERNATIVE: Initiate automatic reboot (Less recommended for complex scripts)
    # sudo reboot now
fi

echo "--- 3. Re-running Ansible for Post-Reboot tasks ---"
ansible-playbook -i inventory.ini setup.yml -k -K

echo "--- Setup Completed ---"
```

এই স্ক্রিপ্টটি ডাউনলোড, প্লেবুক রান, রিবুট এবং পুনরায় কাজ শুরু করার প্রক্রিয়াটি ম্যানুয়ালভাবে ধাপে ধাপে সম্পন্ন করার একটি নির্দেশিকা হিসাবে কাজ করবে। যদি `ansible-playbook` টি রিবুট করে, তবে ইউজারকে স্ক্রিপ্টটি আবার রান করার জন্য বলা হবে।

**Git clone এবং রান করার চূড়ান্ত কমান্ড:**

```bash
# Assuming you have git and ansible installed
git clone https://github.com/your_username/my_ultimate_setup.git
cd my_ultimate_setup
chmod +x run_setup.sh
./run_setup.sh
```

এই পদ্ধতিটি ব্যবহার করে, আপনি আপনার সমস্ত কাজ একটি সুসংগঠিত এবং স্বয়ংক্রিয় উপায়ে পরিচালনা করতে পারবেন।