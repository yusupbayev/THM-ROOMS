# Wonderland - TryHackMe Writeup
**Target IP:** 10.10.x.x  
**Difficulty:** Medium  
**Focus:** Steganography, Library Hijacking, PATH Injection, Linux Capabilities

---

## 1. Reconnaissance
Initial Nmap scans revealed Port 22 (SSH) and Port 80 (HTTP). 

### Directory Brute Forcing
Fuzzing directories revealed `/img` and `/r`. Following a trail of hidden directories based on the theme (r/a/b/b/i/t) led to a hidden page.
* **Findings:** Inspected the source code of `http://wonderland.thm/r/a/b/b/i/t/` and discovered hidden SSH credentials for the user `alice`.

## 2. Initial Access
Authenticated via SSH as `alice`. 
* **The Twist:** Found `root.txt` in Alice's home directory (unreadable) and `user.txt` in the `/root` directory (readable).

## 3. Privilege Escalation: Alice -> Rabbit
Checked `sudo -l` and found that Alice could run a Python script as `rabbit`.
* **Vulnerability:** Python Library Hijacking.
* **Exploit:** Created a malicious `random.py` in the local directory containing a bash spawn command.
* **Result:** Successfully pivoted to the `rabbit` user.



## 4. Privilege Escalation: Rabbit -> Hatter
Discovered a SUID binary in `/home/rabbit` named `teaParty`.
* **Vulnerability:** PATH Injection.
* **Analysis:** Using `strings`, I found the binary called the `date` command without an absolute path.
* **Exploit:** 1. Created a fake `date` script in `/home/rabbit`.
    2. Exported the local path to the front of the $PATH variable.
    3. Executed `teaParty`, triggering my script as the user `hatter`.
* **Result:** Obtained a password for `hatter` in their home directory and stabilized via SSH.

## 5. Privilege Escalation: Hatter -> Root
Searched for special capabilities assigned to binaries.
* **Command:** `getcap -r / 2>/dev/null`
* **Findings:** `/usr/bin/perl` had `cap_setuid+ep` enabled.
* **Exploit:**
  ```bash
  perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
