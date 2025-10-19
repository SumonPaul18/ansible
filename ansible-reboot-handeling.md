### Reference:
- https://gemini.google.com/app/4d1db117c2616f46

আপনার নিজের লিনাক্স মেশিনে Ansible ব্যবহার করে কিছু ইনস্টল, কনফিগার এবং সেটআপ করার ধারণাটি চমৎকার\! একে **লোকাল প্রভিশনিং (Local Provisioning)** বলা হয়। এটি বেশ কয়েকটি কারণে কার্যকর হতে পারে:

1.  **ব্যক্তিগত ডেভেলপমেন্ট এনভায়রনমেন্ট সেটআপ:** আপনার নিজের ল্যাপটপ বা ওয়ার্কস্টেশনে বিভিন্ন ডেভেলপমেন্ট টুল, ভাষা বা ফ্রেমওয়ার্ক স্বয়ংক্রিয়ভাবে সেটআপ করতে।
2.  **সিস্টেম কনফিগারেশন ম্যানেজমেন্ট:** আপনার মেশিনের নির্দিষ্ট কনফিগারেশন ফাইল, সার্ভিস বা প্যাকেজগুলির স্টেট ট্র্যাক রাখতে এবং স্বয়ংক্রিয়ভাবে প্রয়োগ করতে।
3.  **Ansible শেখার জন্য:** রিমোট মেশিন সেটআপ করার আগে Ansible কিভাবে কাজ করে তা নিজের মেশিনে পরীক্ষা করে দেখতে।
4.  **পুনরাবৃত্তিমূলক সেটআপ:** নতুন ল্যাপটপ বা অপারেটিং সিস্টেম ইনস্টল করার সময় আপনার পছন্দের সফটওয়্যার এবং কনফিগারেশনগুলো দ্রুত প্রয়োগ করতে।

আপনার নিজের মেশিনে Ansible ব্যবহার করার জন্য আপনাকে যা করতে হবে:

**ধাপ 1: Ansible ইনস্টল করুন**

প্রথমে আপনার লিনাক্স মেশিনে Ansible ইনস্টল করুন। বেশিরভাগ লিনাক্স ডিস্ট্রিবিউশনের প্যাকেজ ম্যানেজারেই এটি পাওয়া যায়।

  * **Ubuntu/Debian-based:**
    ```bash
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible
    ```
  * **CentOS/RHEL-based:**
    ```bash
    sudo yum install epel-release
    sudo yum install ansible
    # Or for newer versions:
    # sudo dnf install ansible
    ```
  * **Arch Linux:**
    ```bash
    sudo pacman -S ansible
    ```
  * **Python pip ব্যবহার করে (যদি প্যাকেজ ম্যানেজার না থাকে বা নির্দিষ্ট সংস্করণ চান):**
    ```bash
    sudo apt install python3-pip # Or dnf/yum install python3-pip
    pip3 install ansible
    ```
    (যদি `pip3 install ansible` ব্যবহার করেন, তবে নিশ্চিত করুন যে `~/.local/bin` আপনার PATH এ আছে: `export PATH=$PATH:$HOME/.local/bin`)

ইনস্টলেশন যাচাই করতে:

```bash
ansible --version
```

**ধাপ 2: ইনভেন্টরি ফাইল তৈরি করুন (Local Host)**

Ansible কোন হোস্টের উপর কাজ করবে তা জানার জন্য একটি ইনভেন্টরি ফাইল ব্যবহার করে। নিজের মেশিনের জন্য, আপনি `localhost` ব্যবহার করবেন।

একটি ফাইল তৈরি করুন, যেমন `hosts.ini`:

```ini
# hosts.ini
localhost ansible_connection=local
```

  * `localhost`: Ansible কে বলছে এটি আপনার নিজের মেশিন।
  * `ansible_connection=local`: Ansible কে বলছে SSH ব্যবহার না করে সরাসরি আপনার লোকাল মেশিনের উপর কমান্ড চালাতে। এটি `sudo` ব্যবহার করে প্রিভিলেজ এসক্যালেশন পরিচালনা করবে।

**ধাপ 3: একটি প্লেবুক তৈরি করুন**

একটি প্লেবুক হল একটি YAML ফাইল যা আপনি কী করতে চান তা বর্ণনা করে। এটি টাস্কগুলির একটি সেট যা একটি নির্দিষ্ট হোস্ট গ্রুপে চালানো হবে।

উদাহরণস্বরূপ, চলুন একটি প্লেবুক তৈরি করি যা আপনার সিস্টেমে `htop` (একটি প্রসেস মনিটর) ইনস্টল করবে, একটি কাস্টম বার্তা সহ একটি টেক্সট ফাইল তৈরি করবে এবং একটি Apache ওয়েব সার্ভার ইনস্টল ও চালু করবে।

একটি ফাইল তৈরি করুন, যেমন `my_local_setup.yml`:

```yaml
# my_local_setup.yml
---
- name: Setup my local Linux machine
  hosts: localhost
  become: yes # This allows Ansible to use sudo for privileged tasks

  tasks:
    - name: Ensure htop is installed
      ansible.builtin.apt: # Or 'yum'/'dnf' for RHEL/CentOS
        name: htop
        state: present
        update_cache: yes

    - name: Create a custom message file
      ansible.builtin.copy:
        content: |
          Hello from your Ansible-configured local machine!
          This file was created by my_local_setup.yml playbook.
        dest: /tmp/ansible_message.txt
        owner: root
        group: root
        mode: '0644'

    - name: Ensure Apache web server is installed
      ansible.builtin.apt:
        name: apache2 # Or 'httpd' for RHEL/CentOS
        state: present

    - name: Ensure Apache web server is running and enabled at boot
      ansible.builtin.service:
        name: apache2 # Or 'httpd' for RHEL/CentOS
        state: started
        enabled: yes

    - name: Display a success message
      ansible.builtin.debug:
        msg: "Local machine setup complete! Check /tmp/ansible_message.txt and your Apache web server."
```

**প্লেবুকের ব্যাখ্যা:**

  * `---`: একটি YAML ফাইল শুরু নির্দেশ করে।
  * `- name: Setup my local Linux machine`: এই প্লেবুকের একটি নাম।
  * `hosts: localhost`: এই প্লেবুকটি `hosts.ini` ফাইলে সংজ্ঞায়িত `localhost` হোস্টের উপর চলবে।
  * `become: yes`: এটি Ansible কে `sudo` ব্যবহার করতে বলে, যা প্যাকেজ ইনস্টল বা সার্ভিস ম্যানেজ করার মতো প্রিভিলেজড টাস্কের জন্য প্রয়োজন।
  * `tasks:`: এটি টাস্কগুলির একটি তালিকা।
      * `ansible.builtin.apt`: এটি একটি Ansible মডিউল যা Debian/Ubuntu সিস্টেমে প্যাকেজ ম্যানেজ করে। আপনি যদি CentOS/RHEL ব্যবহার করেন তবে এটি `ansible.builtin.yum` বা `ansible.builtin.dnf` হবে।
          * `name: htop`: `htop` প্যাকেজ ইনস্টল করতে।
          * `state: present`: নিশ্চিত করে যে প্যাকেজটি ইনস্টল করা আছে।
          * `update_cache: yes`: প্যাকেজ ক্যাশে আপডেট করে।
      * `ansible.builtin.copy`: একটি মডিউল যা ফাইল কপি বা তৈরি করে।
          * `content`: ফাইলটির বিষয়বস্তু।
          * `dest`: গন্তব্য পাথ।
          * `owner`, `group`, `mode`: ফাইল পারমিশন সেট করে।
      * `ansible.builtin.service`: একটি মডিউল যা সিস্টেম সার্ভিসগুলো ম্যানেজ করে।
          * `state: started`: সার্ভিসটি চালু আছে তা নিশ্চিত করে।
          * `enabled: yes`: সিস্টেম বুট হওয়ার সময় সার্ভিসটি স্বয়ংক্রিয়ভাবে চালু হয় তা নিশ্চিত করে।
      * `ansible.builtin.debug`: কনসোল-এ মেসেজ প্রিন্ট করার জন্য।

**ধাপ 4: প্লেবুক রান করুন**

এখন আপনার ইনভেন্টরি ফাইল এবং প্লেবুক ব্যবহার করে Ansible রান করুন:

```bash
ansible-playbook -i hosts.ini my_local_setup.yml
```

  * `ansible-playbook`: Ansible প্লেবুক রান করার কমান্ড।
  * `-i hosts.ini`: আপনার ইনভেন্টরি ফাইল নির্দিষ্ট করে।
  * `my_local_setup.yml`: আপনার প্লেবুক ফাইল নির্দিষ্ট করে।

**আউটপুট:**

Ansible টাস্কগুলো একের পর এক চালাবে এবং আপনাকে প্রতিটি টাস্কের স্ট্যাটাস দেখাবে (`changed`, `ok`, `failed`)।

**ফলাফল যাচাই করুন:**

প্লেবুক রান হওয়ার পর:

  * `htop` ইনস্টল হয়েছে কিনা তা পরীক্ষা করুন: `htop` কমান্ড চালান।
  * ফাইলটি তৈরি হয়েছে কিনা তা পরীক্ষা করুন: `cat /tmp/ansible_message.txt`
  * Apache ওয়েব সার্ভার চলছে কিনা তা পরীক্ষা করুন: `sudo systemctl status apache2` (বা `httpd`) এবং আপনার ব্রাউজারে `http://localhost/` ভিজিট করুন।

**রিবুট হ্যান্ডলিং (আপনার পূর্ববর্তী প্রশ্নের সাথে লিঙ্ক):**

যদি আপনার প্লেবুকে এমন কোনো টাস্ক থাকে যা রিবুটের প্রয়োজন, যেমন কার্নেল আপডেট, তাহলে আপনি `reboot` মডিউল ব্যবহার করতে পারেন:

```yaml
- name: Setup my local Linux machine
  hosts: localhost
  become: yes

  tasks:
    - name: Update all packages that require reboot (e.g., kernel)
      ansible.builtin.apt:
        upgrade: dist
        update_cache: yes
      register: apt_update_result

    - name: Check if reboot is required after apt upgrade
      ansible.builtin.stat:
        path: /var/run/reboot-required
      register: reboot_required_file

    - name: Reboot if required
      ansible.builtin.reboot:
        reboot_timeout: 600 # Wait up to 10 minutes for reboot
      when: reboot_required_file.stat.exists

    - name: Continue with post-reboot tasks (e.g., install specific software)
      ansible.builtin.apt:
        name: my-post-reboot-software
        state: present
      # This task will only run after the potential reboot
```

এই প্লেবুকটি `apt` আপগ্রেড করার পর `/var/run/reboot-required` ফাইলটি চেক করবে। যদি ফাইলটি বিদ্যমান থাকে, তাহলে `ansible.builtin.reboot` মডিউলটি মেশিনকে রিবুট করবে এবং মেশিনটি আবার অনলাইনে না আসা পর্যন্ত Ansible অপেক্ষা করবে, তারপর প্লেবুকের বাকি টাস্কগুলো (যেমন `my-post-reboot-software` ইনস্টল করা) চালানো শুরু করবে।

এভাবে আপনি আপনার নিজের লিনাক্স মেশিনে Ansible ব্যবহার করে স্বয়ংক্রিয়ভাবে বিভিন্ন ইনস্টলেশন, কনফিগারেশন এবং সেটআপ টাস্ক সম্পন্ন করতে পারেন।