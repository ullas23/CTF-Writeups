# VulnHub: Empire: LupinOne (Medium) Walkthrough

A summary of my methodology for compromising the **Empire: LupinOne** machine. Both the user and root flags were captured successfully.

## 👉 **Looking for the deep dive?** Read the full report with complete command outputs and screenshots here: [(./LupinOne-Writeup.pdf)](https://github.com/ullas23/CTF-Writeups/blob/main/VulnHub-Empire-LupinOne/LupinOne-Writeup.pdf)

---

## Machine Metadata
* **Machine Name:** LupinOne
* **Series:** Empire
* **Release Date:** 21 Oct 2021
* **Author:** icex64 & Empire Cybersecurity
* **Difficulty:** 🟠 Medium
* **Challenge Link:** [VulnHub - Empire: LupinOne](https://www.vulnhub.com/entry/empire-lupinone,750/)

---

## Attack Lifecycle Summary

### 1. Enumeration & Initial Foothold (icex64)
* **Discovery:** Conducted directory fuzzing to reveal an encoded credential payload.
* **Access:** Cracked a protected SSH private key using **John the Ripper** to gain initial access as the user `icex64`.

### 2. Lateral Movement (arsene)
* **SUID Abuse:** Discovered a misconfigured binary execution flaw in `/tmp/bash`.
* **Privilege Transition:** Executed `/tmp/bash -p` to maintain the effective user privileges of `arsene`.
* **Credential Recovery:** Inspected `/home/arsene/.secret` to recover the plain-text password and fully switch to the `arsene` user account.

### 3. Privilege Escalation (Root)
* **Sudo Rights:** Checking `sudo -l` showed that `arsene` could run `/usr/bin/pip` as `root` without a password.
* **Exploitation:** Crafted a malicious Python installation file (`setup.py`) designed to spawn a reverse/interactive shell during module setup.
* **Execution:** Ran `sudo pip install` to execute the code with administrative rights, capturing the final **root flag**.

---

## Final Attack Chain

```bash
SSH as icex64
       │
       ▼
SUID Enumeration
       │
       ▼
Exploit /tmp/bash -p (Effective UID = arsene)
       │
       ▼
Enumerate /home/arsene/.secret
       │
       ▼
Recover arsene password & 'su arsene'
       │
       ▼
Check 'sudo -l' -> (root) NOPASSWD: /usr/bin/pip
       │
       ▼
Create malicious setup.py
       │
       ▼
Execute 'sudo pip install .'
       │
       ▼
Root Shell Obtained
       │
       ▼
Read /root/root.txt (ROOT FLAG)
```

---

## 💡 Key Takeaways
* **Check Effective UIDs:** SUID binaries can allow horizontal movement between users, not just vertical movement to root.
* **Secure Package Managers:** Granting `sudo` permissions over system package installers like `pip` is highly dangerous, as they natively execute custom setup scripts with elevated system context.
