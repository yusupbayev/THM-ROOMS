# THM: Publisher
**Difficulty:** Medium
**Focus:** ImageMagick (CVE-2016-3714), AppArmor Bypass, SSH Key Manipulation

---

## 🛡️ Executive Summary
Publisher is a machine that demonstrates the dangers of using outdated third-party libraries for media processing. The exploit path involves leveraging the "ImageTragick" vulnerability to gain initial execution, followed by a sophisticated privilege escalation that requires bypassing AppArmor profiles by exploiting path-based rule logic.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The scan revealed the following services:
* **22/ssh** - OpenSSH 8.2p1
* **80/http** - Apache httpd 2.4.41

### Web Analysis
The website is a publishing platform. Using `ffuf`, I discovered a directory that allowed users to upload images for processing. Through version detection and header analysis, I identified that the server was using an older version of **ImageMagick**.

## 2. Initial Access: ImageTragick (CVE-2016-3714)
ImageMagick's `mvg` (Magick Vector Graphics) format was vulnerable to command injection. By crafting a malicious `.mvg` file, I could force the server to execute a reverse shell when it attempted to "process" the image.



**Exploitation:**
1. Created a file named `exploit.mvg` with a payload designed to trigger a bash reverse shell.
2. Uploaded the file via the web interface.
3. Caught the incoming shell on a Netcat listener.

**Flag Captured:** `user.txt`

## 3. Privilege Escalation: AppArmor Bypass
During enumeration, I found a custom binary with the SUID bit set. However, standard exploitation attempts (like spawning a shell) were blocked by **AppArmor**, which was enforcing a strict profile on that specific binary.

### The Bypass Strategy
AppArmor profiles are often tied to the **absolute path** of a binary (e.g., `/usr/sbin/binary`). If a user has permission to move or copy that binary to a new location, the AppArmor profile may no longer apply.



**Exploitation:**
1. Copied the SUID binary to the `/dev/shm` directory (a world-writable shared memory space).
2. Executed the binary from the new path.
3. Because the path changed, AppArmor did not trigger its restrictions, allowing the SUID binary to successfully spawn a root shell.

---
**Flag Captured:** `root.txt`
