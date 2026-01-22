# THM: Root Me
**Difficulty:** Easy
**Focus:** Web Fuzzing, File Upload Bypass, SUID Privilege Escalation

---

## 🛡️ Executive Summary
Root Me is a beginner-friendly CTF that focuses on the fundamental "Web-to-Shell" pipeline. It demonstrates how a lack of file-extension validation on an upload form can lead to Remote Code Execution (RCE) and how SUID binaries can be abused for privilege escalation.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The scan revealed two open services:
* **22/ssh** - OpenSSH 7.6p1
* **80/http** - Apache httpd 2.4.29

### Directory Fuzzing
Using `ffuf` (as learned in the HTB Academy), I searched for hidden directories:
`ffuf -u http://10.10.x.x/FUZZ -w /usr/share/wordlists/dirb/common.txt`

**Discovery:**
* `/panel/` - A hidden upload page.
* `/uploads/` - The directory where uploaded files are stored.

## 2. Initial Access: File Upload Bypass (RCE)
The `/panel/` page allowed for file uploads. I initially attempted to upload a standard `.php` reverse shell, but it was blocked by a basic extension filter.

### Bypassing the Filter
As covered in the **File Upload Attacks** module, many servers only block `.php` but forget about alternative extensions that the server can still execute. 
* Attempted: `.php5`, `.phtml`, `.php3`.
* **Result:** `.phtml` was accepted by the server.



**Exploitation:**
1. Renamed my PHP reverse shell to `shell.phtml`.
2. Uploaded it via `/panel/`.
3. Started a Netcat listener: `nc -lvnp 1234`.
4. Navigated to `http://10.10.x.x/uploads/shell.phtml` to trigger the execution.

**Flag Captured:** `user.txt` (Found using `find / -name user.txt 2>/dev/null`)

## 3. Privilege Escalation: SUID Python
To escalate to root, I looked for binaries with the SUID bit set:
`find / -perm -u=s -type f 2>/dev/null`

**Discovery:** `/usr/bin/python` was set to SUID. 



This is a critical misconfiguration. If Python has the SUID bit, it can be used to run a sub-shell that inherits the file owner's (root) permissions.

**Exploitation:**
Using the GTFOBins methodology:
```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
