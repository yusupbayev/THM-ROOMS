# THM: All In One (Write-up)

**Note for your GitHub:** This room is excellent for showcasing "Lateral Movement." You start with a web vulnerability, move to a local user, and finally escalate to root.

---

# 🛡️ Executive Summary
All In One is a comprehensive CTF that requires chaining multiple exploitation techniques. It covers WordPress vulnerability identification, manual credential discovery, and privilege escalation via SUID binaries.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The scan revealed standard web ports:
* **22/ssh**
* **80/http**

### Directory Fuzzing
Using `ffuf`, I discovered a `/wordpress/` directory. Further fuzzing revealed a hidden directory `/hackme/`, which contained a hint regarding a specific WordPress plugin vulnerability.

## 2. Initial Access: WordPress Plugin Exploit
The site was running a vulnerable version of the **Mail Masta** plugin. This plugin is susceptible to **Local File Inclusion (LFI)**.

**Exploitation:**
By leveraging the LFI vulnerability, I was able to read the `wp-config.php` file:
`http://[IP]/wordpress/wp-content/plugins/mail-masta/inc/rebuild_settings.php?pl=[PATH_TO_WP_CONFIG]`

This revealed the database credentials:
* **DB_USER:** `elyana`
* **DB_PASSWORD:** `[REDACTED]`

I used these credentials to log in via **SSH** as the user `elyana`.

## 3. Privilege Escalation: SUID Socat
After performing local enumeration, I searched for SUID binaries:
`find / -perm -u=s -type f 2>/dev/null`

**Discovery:** `/usr/bin/socat` was set to SUID. 

**Exploitation:**
Socat can be used to create a bidirectional byte stream. By running it with SUID permissions, I could spawn a shell that inherits root privileges.

**Command:**
```bash
./socat exec:'/bin/sh -p',pty,stderr,setsid,sigint,sane
