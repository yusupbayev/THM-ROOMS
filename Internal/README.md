# Internal - TryHackMe Writeup
**Target IP:** 10.10.x.x  
**Difficulty:** Hard  
**Focus:** WordPress Exploitation, SSH Tunneling, Jenkins RCE, Docker Escape

---

## 1. Reconnaissance
Initial Nmap scans revealed Port 22 (SSH) and Port 80 (HTTP).

### Directory Brute Forcing
Fuzzing directories with `gobuster` revealed several interesting paths including `/blog` and `/phpmyadmin`.
* **Findings:** The `/blog` directory contained a WordPress installation. Further enumeration with `wpscan` identified a valid user: `admin`.

## 2. Initial Access
Brute-forcing the WordPress login provided administrative access.
* **Credentials:** `admin : my2boys`
* **Exploit:** Navigated to the Theme Editor and injected a PHP reverse shell into the `header.php` file.
* **Result:** Obtained a shell as the `www-data` user.

## 3. Lateral Movement: www-data -> aubreanna
Conducted local enumeration to find a path for horizontal escalation.
* **Vulnerability:** Sensitive information stored in a world-readable file.
* **Findings:** Discovered `/opt/wp-save.txt` which contained credentials for the system user `aubreanna`.
* **Credentials:** `aubreanna : bubb13guM!@#123`
* **Result:** Authenticated via SSH as `aubreanna`.



## 4. Privilege Escalation: aubreanna -> jenkins (Docker)
Identified an internal Jenkins service running on `127.0.0.1:8080`.
* **Vulnerability:** Unauthenticated/Weakly authenticated internal service.
* **Pivoting:** Established an SSH tunnel to forward the internal Jenkins port to my local machine:
  `ssh -L 9000:127.0.0.1:8080 aubreanna@internal.thm`
* **Exploit:** 1. Accessed Jenkins at `http://127.0.0.1:9000` and fuzzed the login to find `admin : spongebob`.
    2. Navigated to **Manage Jenkins > Script Console**.
    3. Executed a Groovy reverse shell script to gain access to the underlying Docker container.
* **Result:** Obtained a shell as the `jenkins` user inside the container.

## 5. Privilege Escalation: jenkins -> Root
Exploited poor credential management within the containerized environment.
* **Findings:** Found a file named `note.txt` in the `/opt` directory of the container.
* **The Twist:** The note contained the plaintext password for the **root** user of the host machine.
* **Credentials:** `root : tr0ub13guM!@#123`
* **Exploit:**
  ```bash
  ssh aubreanna@internal.thm
  su root
  # Entered password found in the container note
