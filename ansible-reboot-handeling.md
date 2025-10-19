### Reference:
- https://gemini.google.com/app/4d1db117c2616f46

Ansible-এ রিবুট হ্যান্ডলিং একটি অত্যন্ত গুরুত্বপূর্ণ বিষয়, বিশেষ করে যখন আপনি সার্ভার আপগ্রেড, কার্নেল আপডেট বা অন্যান্য কনফিগারেশন পরিবর্তন করছেন যার জন্য সিস্টেম রিস্টার্ট করা প্রয়োজন। এটি নিশ্চিত করে যে আপনার অটোমেশন প্রক্রিয়াটি রিবুটের পরেও নির্বিঘ্নে চলতে পারে।

চলুন, Ansible-এ রিবুট হ্যান্ডলিং সম্পর্কে একদম প্রাথমিক স্তর থেকে শুরু করে উচ্চ স্তরের কৌশল পর্যন্ত ধাপে ধাপে গাইডলাইন দেওয়া যাক।

-----

### **Ansible রিবুট হ্যান্ডলিং: প্রাথমিক স্তর থেকে উচ্চ স্তর পর্যন্ত গাইডলাইন**

**লক্ষ্য:** একটি Ansible প্লেবুক তৈরি করা যা প্রয়োজনীয় ক্ষেত্রে সিস্টেমকে রিবুট করবে এবং রিবুটের পরে প্লেবুকের বাকি কাজগুলো স্বয়ংক্রিয়ভাবে চালিয়ে যাবে।

-----

### **ধাপ 0: পূর্বশর্ত এবং মৌলিক ধারণা**

  * **Ansible ইনস্টলেশন:** আপনার কন্ট্রোল নোডে Ansible ইনস্টল করা থাকতে হবে।
  * **ইনভেন্টরি ফাইল:** আপনার টার্গেট হোস্ট (যে মেশিন/সার্ভার রিবুট হবে) ইনভেন্টরি ফাইলে সংজ্ঞায়িত করা থাকতে হবে।
      * উদাহরণ: `hosts.ini`
        ```ini
        [web_servers]
        webserver1 ansible_host=your_server_ip ansible_user=your_ssh_user
        ```
  * **SSH অ্যাক্সেস:** কন্ট্রোল নোড থেকে টার্গেট হোস্টে SSH অ্যাক্সেস থাকতে হবে (পাসওয়ার্ড-লেস SSH কী প্রস্তাবিত)।
  * **`become: yes`:** প্লেবুকে `become: yes` ব্যবহার করতে হবে যদি রিবুটের জন্য রুট প্রিভিলেজ (sudo) প্রয়োজন হয়।

-----

### **ধাপ 1: মৌলিক রিবুট - `ansible.builtin.reboot` মডিউল ব্যবহার**

এটি Ansible-এ রিবুট হ্যান্ডলিংয়ের সবচেয়ে প্রাথমিক এবং সহজ উপায়। `ansible.builtin.reboot` মডিউলটি সিস্টেমকে রিবুট করে এবং এটি পুনরায় অনলাইন না হওয়া পর্যন্ত Ansible অপেক্ষা করে।

**উদাহরণ প্লেবুক (`basic_reboot.yml`):**

```yaml
---
- name: Perform a basic system reboot
  hosts: web_servers # Replace with your host group
  become: yes # Required for rebooting

  tasks:
    - name: Display a message before reboot
      ansible.builtin.debug:
        msg: "System will now reboot. Ansible will wait for it to come back online."

    - name: Initiate system reboot and wait for it to return
      ansible.builtin.reboot:
        reboot_timeout: 600 # Wait up to 10 minutes (600 seconds) for the system to come back
        # Other optional parameters:
        # post_reboot_delay: 10 # Wait 10 seconds after system is online before proceeding
        # connect_timeout: 60 # SSH connection timeout after reboot
        # test_command: "uptime" # A command to test if the system is fully up (optional, defaults to SSH check)

    - name: Display a message after successful reboot
      ansible.builtin.debug:
        msg: "System has successfully rebooted and is online. Proceeding with next tasks."

    - name: Example task after reboot
      ansible.builtin.command: uptime
      register: uptime_output
    - ansible.builtin.debug:
        msg: "System uptime after reboot: {{ uptime_output.stdout }}"
```

**কিভাবে রান করবেন:**

```bash
ansible-playbook -i hosts.ini basic_reboot.yml
```

**ব্যাখ্যা:**

  * `ansible.builtin.reboot`: এই মডিউলটি রিবুট ট্রিগার করে।
  * `reboot_timeout`: এটি সর্বাধিক সময় (সেকেন্ডে) যা Ansible রিবুট শেষ হওয়ার এবং টার্গেট হোস্টের সাথে পুনরায় সংযোগ স্থাপনের জন্য অপেক্ষা করবে। ডিফল্ট হল 600 সেকেন্ড (10 মিনিট)। যদি এই সময়ের মধ্যে সংযোগ স্থাপন না হয়, তাহলে টাস্কটি ব্যর্থ হবে।
  * `post_reboot_delay`: রিবুট হওয়ার পর পুনরায় সংযোগ স্থাপন হলে, পরবর্তী টাস্ক শুরু করার আগে অতিরিক্ত কত সেকেন্ড অপেক্ষা করবে। এটি নিশ্চিত করে যে সিস্টেম পুরোপুরি স্থিতিশীল হয়েছে।

-----

### **ধাপ 2: শর্তসাপেক্ষে রিবুট - যখন প্রয়োজন হয় তখনই রিবুট**

সাধারণত, আপনি তখনই রিবুট করতে চান যখন এটি প্রয়োজন হয় (যেমন কার্নেল আপডেট বা নির্দিষ্ট সার্ভিস আপডেটের পর)। এটি অপ্রয়োজনীয় রিবুট এড়িয়ে সময় বাঁচায়।

**লিনাক্সে রিবুট প্রয়োজনের চিহ্ন:**

  * **`/var/run/reboot-required` ফাইল:** Debian/Ubuntu-ভিত্তিক সিস্টেমে প্যাকেজ আপডেটের পর যদি রিবুট প্রয়োজন হয়, তাহলে এই ফাইলটি তৈরি হয়।
  * **`/var/run/reboot-required.pkgs` ফাইল:** এটিও Debian/Ubuntu তে, কোন প্যাকেজগুলোর জন্য রিবুট প্রয়োজন তা তালিকাভুক্ত করে।
  * **নির্দিষ্ট প্যাকেজ ম্যানেজার কমান্ড:** কিছু ডিস্ট্রিবিউশনে প্যাকেজ ম্যানেজার রিবুটের প্রয়োজন আছে কিনা তা নির্দেশ করতে পারে (যেমন Red Hat-এ `needs-restarting -r`)।

**উদাহরণ প্লেবুক (`conditional_reboot.yml`):**

```yaml
---
- name: Update system and reboot if necessary
  hosts: web_servers
  become: yes

  tasks:
    - name: Update all packages that require reboot (e.g., kernel)
      ansible.builtin.apt: # Use 'yum' or 'dnf' for RHEL/CentOS
        upgrade: dist
        update_cache: yes
      register: apt_update_result # Register output to check for changes

    - name: Check if /var/run/reboot-required file exists (Debian/Ubuntu)
      ansible.builtin.stat:
        path: /var/run/reboot-required
      register: reboot_required_file

    # For RHEL/CentOS, you might use 'command: needs-restarting -r' and check its return code
    # - name: Check if reboot is required (RHEL/CentOS)
    #   ansible.builtin.command: needs-restarting -r
    #   register: needs_restarting_result
    #   changed_when: needs_restarting_result.rc != 0
    #   failed_when: false # Don't fail if the command exits with non-zero

    - name: Initiate system reboot if required
      ansible.builtin.reboot:
        reboot_timeout: 600
      when: reboot_required_file.stat.exists
      # For RHEL/CentOS: when: needs_restarting_result.rc != 0

    - name: Continue with post-reboot tasks
      ansible.builtin.debug:
        msg: "System is ready for post-reboot tasks."

    - name: Install a package that should only be installed after potential reboot
      ansible.builtin.apt:
        name: sysstat # Example package
        state: present
```

**ব্যাখ্যা:**

  * `ansible.builtin.stat`: এই মডিউলটি একটি ফাইলের অবস্থা চেক করে। আমরা `/var/run/reboot-required` ফাইলটি বিদ্যমান আছে কিনা তা পরীক্ষা করছি।
  * `register: reboot_required_file`: `stat` মডিউলের আউটপুট `reboot_required_file` ভেরিয়েবলে সংরক্ষণ করা হয়।
  * `when: reboot_required_file.stat.exists`: `reboot` টাস্কটি তখনই চলবে যখন `reboot_required_file.stat.exists` সত্য হবে (অর্থাৎ, ফাইলটি বিদ্যমান)।
  * RHEL/CentOS এর জন্য `needs-restarting -r` কমান্ড ব্যবহার করা হয়েছে, যা রিবুটের প্রয়োজন হলে একটি নন-জিরো এক্সিট কোড (`rc != 0`) দেয়।

-----

### **ধাপ 3: হ্যান্ডলার ব্যবহার করে রিবুট ট্রিগার করা (উন্নত)**

হ্যান্ডলার (Handlers) হল Ansible-এর একটি শক্তিশালী ফিচার যা টাস্ক সম্পন্ন হওয়ার পরে চালানো হয়, যদি সেই টাস্কটি `changed` (কিছু পরিবর্তন) হয়ে থাকে এবং হ্যান্ডলারকে `notify` করা হয়। এটি নিশ্চিত করে যে রিবুটের মতো ব্যয়বহুল অপারেশন শুধুমাত্র তখনই ট্রিগার হয় যখন সত্যিই প্রয়োজন হয়।

**উদাহরণ প্লেবুক (`reboot_with_handler.yml`):**

```yaml
---
- name: Update system and reboot using a handler
  hosts: web_servers
  become: yes

  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: yes
      changed_when: false # This task won't report "changed" unless it actually updates

    - name: Upgrade all installed packages
      ansible.builtin.apt:
        upgrade: dist
      notify: Check and reboot if necessary # This will trigger the handler if any packages are upgraded

    - name: Install Nginx web server
      ansible.builtin.apt:
        name: nginx
        state: present
      # This task will run BEFORE the handler, but the handler will run after all tasks are evaluated.

  handlers:
    - name: Check and reboot if necessary
      ansible.builtin.debug:
        msg: "Handler 'Check and reboot if necessary' is triggered. Checking for reboot-required file."

    - name: Initiate system reboot (handler)
      ansible.builtin.reboot:
        reboot_timeout: 600
      # The 'when' condition makes sure the handler actually reboots only if the file exists
      when: ansible_facts.default_ipv4.address is defined and '/var/run/reboot-required' in lookup('fileglob', '/var/run/reboot-required') # More robust check for handler
      # OR, for simplicity: when: '/var/run/reboot-required' in lookup('fileglob', '/var/run/reboot-required')
      # Note: 'ansible_facts' might not be fully populated immediately after reboot in some edge cases within handlers
      # The fileglob lookup is a more direct way to check for the file's existence in handlers.
```

**কিভাবে রান করবেন:**

```bash
ansible-playbook -i hosts.ini reboot_with_handler.yml
```

**ব্যাখ্যা:**

  * `notify: Check and reboot if necessary`: `upgrade: dist` টাস্কটি যদি কোনো প্যাকেজ আপগ্রেড করে (অর্থাৎ `changed` হয়), তাহলে এটি `Check and reboot if necessary` নামের হ্যান্ডলারকে ট্রিগার করবে।
  * **হ্যান্ডলার্স (Handlers):** হ্যান্ডলারগুলি প্লেবুকের `tasks` সেকশনের সব টাস্ক চলার পর এবং `changed` টাস্ক দ্বারা `notify` হওয়ার পরই চলে। এর মানে হল, যদি আপনি একাধিক টাস্ক থেকে একটি হ্যান্ডলারকে `notify` করেন, তবে হ্যান্ডলারটি একবারই রান করবে।
  * হ্যান্ডলারে `when` কন্ডিশন যোগ করা হয়েছে যাতে এটি শুধুমাত্র তখনই রিবুট করে যখন `/var/run/reboot-required` ফাইলটি আসলেই বিদ্যমান থাকে। এটি অতিরিক্ত সুরক্ষা স্তর যোগ করে।
  * **গুরুত্বপূর্ণ:** হ্যান্ডলারের মধ্যে `/var/run/reboot-required` এর অস্তিত্ব পরীক্ষা করার জন্য `lookup('fileglob', '/var/run/reboot-required')` ব্যবহার করা হয়েছে। হ্যান্ডলারের মধ্যে `ansible_facts` সম্পূর্ণরূপে রিফ্রেশ না হওয়ার সম্ভাব্য সমস্যার জন্য এটি আরও নির্ভরযোগ্য।

-----

### **ধাপ 4: রিবুটের পরে ভিন্ন টাস্ক সেট - ফ্লেক্সিবল ওয়ার্কফ্লো (উচ্চ স্তরের কৌশল)**

কিছু পরিস্থিতিতে, আপনি রিবুটের আগে এক সেট টাস্ক চালাতে চান এবং রিবুটের পরে অন্য সেট টাস্ক চালাতে চান। এটি অর্জন করতে আপনি একাধিক প্লেবুক বা একই প্লেবুকের মধ্যে লজিক ব্যবহার করতে পারেন।

**কৌশল 1: দুটি পৃথক প্লেবুক ব্যবহার (ম্যানুয়াল বা কাস্টম স্ক্রিপ্ট দিয়ে চেইন করা)**

  * **`pre_reboot_tasks.yml`:** রিবুটের আগে চালানো হবে এমন টাস্ক।
  * **`post_reboot_tasks.yml`:** রিবুটের পরে চালানো হবে এমন টাস্ক।

<!-- end list -->

```yaml
# pre_reboot_tasks.yml
---
- name: Pre-reboot setup
  hosts: web_servers
  become: yes
  tasks:
    - name: Apply critical updates
      ansible.builtin.apt:
        upgrade: dist
      notify: Request reboot

  handlers:
    - name: Request reboot
      ansible.builtin.reboot:
        reboot_timeout: 600
      # You might want to save a flag file here if you need to know later that a reboot happened
      # - name: Mark reboot requested
      #   ansible.builtin.copy:
      #     content: "reboot_requested\n"
      #     dest: /tmp/reboot_flag.txt
      #     mode: '0600'
```

```yaml
# post_reboot_tasks.yml
---
- name: Post-reboot setup
  hosts: web_servers
  become: yes
  tasks:
    - name: Ensure specific services are running after reboot
      ansible.builtin.service:
        name: my_application_service
        state: started
        enabled: yes

    - name: Final configurations
      ansible.builtin.copy:
        src: files/post_reboot_config.conf
        dest: /etc/my_app/config.conf
```

**কিভাবে রান করবেন:**

1.  `ansible-playbook -i hosts.ini pre_reboot_tasks.yml`
2.  Ansible রিবুট শেষ হওয়ার জন্য অপেক্ষা করবে।
3.  ম্যানুয়ালি বা একটি কাস্টম স্ক্রিপ্ট দিয়ে, `ansible-playbook -i hosts.ini post_reboot_tasks.yml` চালান।

**কৌশল 2: একই প্লেবুকের মধ্যে লজিক (উচ্চ স্তরের)**

এই কৌশলটি `ansible.builtin.meta: end_play` বা কাস্টম ফ্ল্যাগ ফাইল (যেমন পূর্বে আলোচনা করা হয়েছিল) ব্যবহার করে রিবুটের পরে প্লেবুকের বাকি অংশ কার্যকর করা নিশ্চিত করে।

**উদাহরণ প্লেবুক (`advanced_reboot_workflow.yml`):**

```yaml
---
- name: Stage 1: Pre-reboot tasks and reboot if necessary
  hosts: web_servers
  become: yes
  vars:
    # A flag to check if the playbook has already run up to the reboot point
    reboot_flag_file: "/tmp/ansible_reboot_in_progress"

  tasks:
    - name: Check if playbook is continuing after a reboot
      ansible.builtin.stat:
        path: "{{ reboot_flag_file }}"
      register: reboot_flag_status

    - name: Skip pre-reboot tasks if flag file exists
      ansible.builtin.meta: end_play # Stop this play
      when: reboot_flag_status.stat.exists # If flag exists, we are in post-reboot phase

    - name: Initial system update (Stage 1)
      ansible.builtin.apt:
        upgrade: dist
        update_cache: yes
      register: apt_update_result

    - name: Create reboot flag file if update changed something
      ansible.builtin.copy:
        content: "rebooting\n"
        dest: "{{ reboot_flag_file }}"
        mode: '0600'
      when: apt_update_result.changed

    - name: Initiate system reboot (Stage 1.5)
      ansible.builtin.reboot:
        reboot_timeout: 600
      when: apt_update_result.changed # Only reboot if updates were actually applied

# --- This marks the start of the post-reboot phase in the SAME PLAYBOOK ---
- name: Stage 2: Post-reboot tasks
  hosts: web_servers
  become: yes
  vars:
    reboot_flag_file: "/tmp/ansible_reboot_in_progress" # Same flag file as above

  pre_tasks: # Tasks to run before roles or main tasks
    - name: Remove reboot flag file
      ansible.builtin.file:
        path: "{{ reboot_flag_file }}"
        state: absent
      # Only remove the flag if it exists, meaning we've successfully passed the reboot point
      # This task is critical to ensure the playbook doesn't re-run pre-reboot tasks next time

  tasks:
    - name: Install and configure post-reboot software
      ansible.builtin.apt:
        name:
          - docker.io
          - docker-compose
        state: present

    - name: Add current user to docker group
      ansible.builtin.user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Final cleanup
      ansible.builtin.apt:
        autoremove: yes
      when: True # Always run, assuming we are post-reboot

    - name: Verify setup
      ansible.builtin.debug:
        msg: "Setup complete! Docker installed and user {{ ansible_user }} added to docker group."
```

**কিভাবে রান করবেন:**

```bash
ansible-playbook -i hosts.ini advanced_reboot_workflow.yml
```

**ব্যাখ্যা:**

  * **দুটি প্লে (Play):** এই প্লেবুকে দুটি ভিন্ন `play` রয়েছে। প্রথম প্লেটি রিবুটের আগে কাজ করে, এবং দ্বিতীয় প্লেটি রিবুটের পরে কাজ করে।
  * `reboot_flag_file`: একটি টেম্পোরারি ফ্ল্যাগ ফাইল যা নির্দেশ করে যে রিবুট প্রক্রিয়া চলছে এবং প্লেবুকটি রিবুটের পরে পুনরায় শুরু হচ্ছে।
  * `meta: end_play`: এটি একটি বিশেষ Ansible মডিউল যা বর্তমান প্লেটি বন্ধ করে দেয়।
      * প্রথম প্লেতে, যদি `reboot_flag_file` বিদ্যমান থাকে (যার অর্থ আমরা ইতিমধ্যে একবার রান করে রিবুটের পরে ফিরে এসেছি), তবে `end_play` টাস্কটি ট্রিগার হবে এবং প্রথম প্লেটি শেষ হয়ে যাবে। এরপর দ্বিতীয় প্লে শুরু হবে।
      * যদি `reboot_flag_file` বিদ্যমান না থাকে, তবে প্রথম প্লেটি তার সব টাস্ক চালাবে, প্রয়োজন হলে রিবুট করবে এবং `reboot_flag_file` তৈরি করবে।
  * **রিবুটের পর:** যখন সিস্টেম রিবুট হয় এবং আপনি আবার প্লেবুকটি চালান (ম্যানুয়ালি বা `systemd` এর মাধ্যমে), তখন প্রথম প্লেটি `reboot_flag_file` খুঁজে পাবে এবং `meta: end_play` দ্বারা স্কিপ হয়ে যাবে।
  * **দ্বিতীয় প্লে (`pre_tasks`):** দ্বিতীয় প্লেটি শুরু হবে এবং `pre_tasks` সেকশনে প্রথমে `reboot_flag_file` মুছে ফেলবে। এটি অত্যন্ত গুরুত্বপূর্ণ, কারণ এটি নিশ্চিত করে যে পরেরবার প্লেবুকটি আবার শুরু হলে এটি আবার প্রথম প্লে থেকে শুরু হবে (যদি আপনি আবার শুরু করতে চান)।
  * এরপরে দ্বিতীয় প্লেয়ের বাকি টাস্কগুলো চলবে।

**এই উন্নত কৌশলটি নিশ্চিত করে যে:**

  * প্লেবুকটি রিবুটের আগে এবং পরে ভিন্ন ভিন্ন লজিক চালাতে পারে।
  * রিবুটের পরে, প্লেবুক স্বয়ংক্রিয়ভাবে সঠিক ধাপে চলে যাবে।
  * এটি একটি শক্তিশালী এবং ফ্লেক্সিবল পদ্ধতি যা জটিল ওয়ার্কফ্লোর জন্য ব্যবহৃত হয়।

-----

### **গুরুত্বপূর্ণ বিবেচনা এবং সেরা অভ্যাস:**

  * **Idempotence:** নিশ্চিত করুন যে আপনার টাস্কগুলি Idempotent। অর্থাৎ, একই টাস্ক বারবার চালালেও সিস্টেমের অবস্থা কাঙ্ক্ষিত অবস্থায়ই থাকবে এবং অপ্রয়োজনীয় পরিবর্তন হবে না। Ansible মডিউলগুলি সাধারণত Idempotent হয়।
  * **Error Handling:** নেটওয়ার্ক সমস্যা, SSH সমস্যা, বা টাস্ক ব্যর্থ হলে কিভাবে হ্যান্ডেল করবেন তা বিবেচনা করুন। `ignore_errors` বা `block/rescue` ব্যবহার করতে পারেন।
  * **SSH Connection Timeout:** রিবুটের পর `ansible.builtin.reboot` মডিউলের `reboot_timeout` পর্যাপ্ত বড় রাখুন। খুব বেশি ছোট হলে সংযোগ পুনরায় স্থাপন হওয়ার আগেই টাইমআউট হতে পারে।
  * **Persistent Connections (Optional):** যদি আপনার ঘন ঘন রিবুট করার প্রয়োজন হয় এবং SSH সংযোগের পারফরম্যান্স গুরুত্বপূর্ণ হয়, তবে `ansible.cfg` এ `control_path` বা `control_master` সেট করে SSH Persistent Connections ব্যবহার করতে পারেন।
  * **লগিং:** `ansible.cfg` এ লগিং সেটআপ করুন যাতে আপনি প্রতিটি রান এবং রিবুট ইভেন্টের লগ দেখতে পারেন।
  * **Testing:** ছোট পরিবেশে পুঙ্খানুপুঙ্খভাবে আপনার রিবুট লজিক পরীক্ষা করুন, যাতে প্রোডাকশন সিস্টেমে কোনো অপ্রত্যাশিত আচরণ না হয়।
  * **`systemd` সার্ভিস দিয়ে স্বয়ংক্রিয়ভাবে প্লেবুক পুনরায় শুরু:** যদি আপনি চান যে সিস্টেম রিবুট হওয়ার পরে প্লেবুকটি স্বয়ংক্রিয়ভাবে পুনরায় শুরু হোক (বিশেষ করে যদি এটি রিবুটের সময় মাঝপথে থেমে যায়), তাহলে আপনি পূর্বে আলোচনা করা `systemd` সার্ভিস পদ্ধতি ব্যবহার করতে পারেন। `ExecStart` কমান্ডে আপনার `ansible-playbook` কমান্ডটি রাখুন।
    ```ini
    [Service]
    Type=oneshot
    ExecStart=/usr/bin/ansible-playbook -i /path/to/hosts.ini /path/to/my_playbook.yml
    RemainAfterExit=yes
    ```
    এবং `my_playbook.yml` ফাইলটি `advanced_reboot_workflow.yml` এর মতো করে তৈরি করুন যা তার নিজের অবস্থা ট্র্যাক করে।

এই গাইডলাইনগুলো আপনাকে Ansible-এ রিবুট হ্যান্ডলিংয়ের বিভিন্ন স্তর এবং কৌশল বুঝতে সাহায্য করবে। আপনার নির্দিষ্ট প্রয়োজনের উপর ভিত্তি করে সঠিক পদ্ধতি নির্বাচন করুন।