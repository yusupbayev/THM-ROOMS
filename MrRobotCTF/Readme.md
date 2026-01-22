# THM: Mr. Robot CTF
**Difficulty:** Medium
**Focus:** Cryptography, Web Fuzzing, WordPress Exploitation, SUID PrivEsc

---

## 🛡️ Executive Summary
Based on the popular TV show, this room is a multi-stage CTF that tests a researcher's ability to find hidden data, exploit CMS vulnerabilities (WordPress), and perform privilege escalation through SUID binaries.

## 1. Reconnaissance & Information Gathering
### Nmap Scan
The scan revealed port 80 (HTTP) and 443 (HTTPS) open. Interestingly, SSH (port 22) was closed/filtered, suggesting initial access must be web-based.

### Hidden File Discovery
* **robots.txt:** Checking the standard `robots.txt` file revealed two entries:
  * `key-1-of-3.txt` -> **First Flag Captured.**
  * `fsocity.dic` -> A custom wordlist.
  
**Strategic Move:** I downloaded `fsocity.dic` immediately. Recognizing it contained many duplicates, I cleaned it to speed up future brute-force attacks:
`cat fsocity.dic | sort -u > cleaned_wordlist.txt`

## 2. Initial Access: WordPress Exploitation
Using `ffuf` for directory discovery, I located a `/wp-login.php` page.

### Brute Forcing Credentials
Using the cleaned wordlist and **Hydra** (or Intruder), I performed a username enumeration. WordPress's error messages ("Invalid username") allowed me to confirm the user `Elliot`.

Next, I brute-forced the password for `Elliot` using the same dictionary.
* **Credentials:** `Elliot:[REDACTED]`

### Gaining a Shell
Inside the WordPress Dashboard:
1. Navigated to the **Appearance > Editor**.
2. Modified the `404.php` template.
3. Injected a **PHP Reverse Shell**.
4. Set up a Netcat listener (`nc -lvnp 4444`) and triggered the 404 error to receive the shell.



## 3. Lateral Movement: User Robot
The second flag was located in `/home/robot/key-2-of-3.txt`, but it was only readable by the user `robot`. In the same directory, I found a file named `password.raw-md5`.

**Decryption:**
The file contained an MD5 hash. Using **CrackStation** or `hashcat`, I cracked the hash to reveal the password for the user `robot`.
* **Robot Pass:** `[REDACTED]`

`su robot` -> **Second Flag Captured.**

## 4. Privilege Escalation: SUID Nmap
To escalate to root, I searched for binaries with the **SUID bit** set:
`find / -perm -u=s -type f 2>/dev/null`

**Discovery:** `/usr/local/bin/nmap` was set to SUID. 



**Exploitation:**
Older versions of Nmap (like the one installed) have an "interactive mode" that can be abused to execute system commands.
```bash
nmap --interactive
!sh
