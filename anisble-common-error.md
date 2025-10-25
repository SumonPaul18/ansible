```
root@UbuntuDesktop1:~/ansible-prep# ansible-playbook -i inventory.ini site.yml PLAY [Prepare target nodes for Ansible management] ************************************************************* TASK [Gathering Facts] ***************************************************************************************** fatal: [192.168.0.52]: FAILED! => {"msg": "Missing sudo password"} PLAY RECAP ***************************************************************************************************** 192.168.0.52 : ok=0 changed=0 unreachable=0 failed=1 skipped=0 rescued=0 ignored=0 root@UbuntuDesktop1:~/ansible-prep#

```

আপনার এখনকার এররটি:

1

fatal: [192.168.0.52]: FAILED! => {"msg": "Missing sudo password"}

### 🔍 **সমস্যার কারণ**

আপনার playbook-এ `become: yes` (অর্থাৎ `sudo`) ব্যবহার করা হচ্ছে, কিন্তু:

- আপনার **target নোডের `cloud3` ইউজারের `sudo` অ্যাক্সেস নেই**, **অথবা**
- `sudo` করতে **পাসওয়ার্ড লাগে**, কিন্তু Ansible-কে সেটি দেওয়া হয়নি।

> 📌 মনে রাখুন: আপনার playbook **প্রথমেই `sudo` অ্যাক্সেস চায়**, কিন্তু এখনো **`cloud3` ইউজারকে `NOPASSWD` sudo দেওয়া হয়নি** — কারণ সেটি তো এই playbook-ই করবে!

অর্থাৎ: **আপনি এখন একটি "chicken-and-egg" সমস্যায় আছেন**।

---

## ✅ **সমাধান: প্রথমবারের জন্য `--ask-become-pass` ব্যবহার করুন**

যেহেতু আপনার `cloud3` ইউজারের `sudo` করতে পাসওয়ার্ড লাগে (এবং এখনো `NOPASSWD` সেট হয়নি), তাই আপনাকে Ansible-কে **sudo পাসওয়ার্ড দিতে হবে**।

### 🔧 ধাপগুলো:

#### 1. **নিশ্চিত করুন যে `cloud3` ইউজারের `sudo` অ্যাক্সেস আছে**

SSH করে চেক করুন:

bash

1

ssh cloud3@192.168.0.52

তারপর:

bash

1

sudo whoami

> যদি এটি কাজ করে (পাসওয়ার্ড দিয়ে), তাহলে আপনার ইউজারের `sudo` অ্যাক্সেস আছে — শুধু পাসওয়ার্ড লাগে।

> ❌ যদি `cloud3 is not in the sudoers file` এরর আসে, তাহলে আপনাকে **প্রথমে target মেশিনে root হিসেবে লগইন করে `cloud3`-কে sudo গ্রুপে যোগ করতে হবে** (নিচে দেখানো হয়েছে)।

---

#### 2. **Playbook রান করুন `--ask-become-pass` অপশন সহ**

bash

1

ansible-playbook -i inventory.ini site.yml --ask-become-pass

> এটি আপনাকে **sudo পাসওয়ার্ড** চাইবে (যা আপনার `cloud3` ইউজারের পাসওয়ার্ড নিজেই)।

✅ একবার এই কমান্ড চালানোর পর, playbook:

- `cloud3`-কে `NOPASSWD` sudo দেবে
- SSH key সেট করবে
- Password-based SSH বন্ধ করবে

তারপর থেকে আপনি **কোনো পাসওয়ার্ড ছাড়াই** playbook রান করতে পারবেন!

---

## ❗ যদি `cloud3` ইউজারের `sudo` অ্যাক্সেস না থাকে?

তাহলে আপনাকে **target মেশিনে root হিসেবে লগইন করে** নিচের কাজগুলো করতে হবে:

### ধাপ 1: root হিসেবে SSH করুন

bash

1

ssh root@192.168.0.52

### ধাপ 2: `cloud3` ইউজারকে `sudo` গ্রুপে যোগ করুন

- **Ubuntu/Debian**:
    
    bash
    
    1
    
    usermod -aG sudo cloud3
    
- **CentOS/Rocky/AlmaLinux**:
    
    bash
    
    1
    
    usermod -aG wheel cloud3
    

### ধাপ 3: `cloud3` এর পাসওয়ার্ড সেট করুন (যদি না থাকে)

bash

1

passwd cloud3

### ধাপ 4: লগআউট করুন

bash

1

exit

এখন আবার `--ask-become-pass` দিয়ে playbook রান করুন।

---

## 📌 সারসংক্ষেপ

|সমস্যা|সমাধান|
|---|---|
|`Missing sudo password`|`ansible-playbook ... --ask-become-pass`|
|`cloud3`-এর`sudo`নেই|root হিসেবে লগইন →`usermod -aG sudo cloud3`|
|পরবর্তী রানে পাসওয়ার্ড লাগবে না|কারণ playbook নিজেই`NOPASSWD`sudo সেট করবে|

---

## ▶️ এখন কী করবেন?

1. নিশ্চিত করুন `cloud3` এর `sudo` অ্যাক্সেস আছে (SSH করে `sudo whoami` চালান)
2. তারপর এই কমান্ড রান করুন:
    
    bash
    
    1
    
    ansible-playbook -i inventory.ini site.yml --ask-become-pass
    
3. প্রথমে **SSH পাসওয়ার্ড** (যদি `sshpass` ব্যবহার করেন), তারপর **sudo পাসওয়ার্ড** চাইবে — দুটোই আপনার `cloud3` ইউজারের পাসওয়ার্ড।

✅ একবার সফল হলে, পরবর্তী সবকিছু অটোমেটিক হবে!

---

যদি আবারও কোনো এরর আসে (যেমন: `Authentication failure`), তখন জানান — আমি আপনাকে সাহায্য করব। 🛠️