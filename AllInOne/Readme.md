# THM: All In One
**Difficulty:** Medium
**Focus:** WordPress Exploitation, LFI (Local File Inclusion), SUID Exploitation

---

## 🛡️ Executive Summary
All In One is a comprehensive machine that requires a systematic approach to web enumeration and exploit chaining. The path involves identifying a vulnerable WordPress plugin (Mail Masta) to leak sensitive configuration files, followed by leveraging a misconfigured SUID binary (`socat`) to elevate privileges to root.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The initial scan identified two open ports:
* **22/ssh** - OpenSSH 7.6p1
* **80/http** - Apache httpd 2.4.29

### Web Discovery & Fuzzing
Initial inspection of the homepage showed a standard Apache landing page. I used `ffuf` to find hidden directories:
`ffuf -u http://[IP]/FUZZ -w /usr/share/wordlists/dirb/common.txt`

**Discovery:**
* `/wordpress/` - A standard WordPress installation.
* `/hackme/` - A directory containing hints about potential vulnerabilities.

## 2. Initial Access: LFI via Mail Masta
By enumerating the WordPress plugins, I discovered **Mail Masta v1.0**. This plugin is known for a Local File Inclusion (LFI) vulnerability in its `rebuild_settings.php` file.



**Exploitation:**
I used the LFI to read the `wp-config.php` file to find database credentials:
`http://[IP]/wordpress/wp-content/plugins/mail-masta/inc/rebuild_settings.php?pl=/var/www/html/wordpress/wp-config.php`

**Credentials Found:**
* **User:** `elyana`
* **Password:** `[REDACTED]`

I successfully used these credentials to gain access to the system via **SSH**.
* **Flag Captured:** `user.txt`

## 3. Privilege Escalation: SUID Socat
During post-exploitation enumeration, I searched for binaries with the SUID bit set:
`find / -perm -u=s -type f 2>/dev/null`

**Discovery:**
The binary `/usr/bin/socat` had the SUID bit set. Socat is a powerful networking tool, and in this context, it can be abused to execute commands as the file owner (root).



**Exploitation:**
Using the GTFOBins methodology, I executed a command to spawn a shell that maintains root privileges:
```bash
socat exec:'/bin/sh -p',pty,stderr,setsid,sigint,sane
