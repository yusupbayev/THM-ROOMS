# THM: Tomghost
**Difficulty:** Easy/Medium
**Focus:** Ghostcat (CVE-2020-1938), PGP Cracking, Sudo Privilege Escalation

---

## 🛡️ Executive Summary
Tomghost is a classic example of how a single unpatched service (Apache Tomcat) can lead to full system compromise. The attack involves exploiting the AJP (Apache JServ Protocol) to read sensitive configuration files, followed by lateral movement via PGP decryption and vertical escalation through binary misconfiguration.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The initial scan revealed the following open ports:
* **22/ssh** - OpenSSH 7.2p2
* **53/dns** - ISC BIND 9.10.3
* **8080/http** - Apache Tomcat 9.0.30
* **8009/ajp13** - Apache JServ Protocol v1.3

### Web Discovery
Navigating to port 8080 showed a default Tomcat installation. Direct access to the `/manager/html` directory was restricted (403 Forbidden), requiring valid administrative credentials.

## 2. Initial Access: Ghostcat (CVE-2020-1938)
The Tomcat version (9.0.30) is famously vulnerable to **Ghostcat**. This vulnerability allows an attacker to read any file from the `WEB-INF` or `META-INF` directories via the AJP connector on port 8009.

**Exploitation:**
Using a modified Python exploit script, I successfully read the `WEB-INF/web.xml` file, which contained cleartext credentials:
* **User:** `skyfuck`
* **Pass:** `[REDACTED]`

I used these credentials to gain initial access via **SSH**.

## 3. Lateral Movement: PGP Decryption
Once inside as `skyfuck`, I discovered two interesting files in the home directory:
* `tryhackme.asc` (PGP Private Key)
* `credential.pgp` (Encrypted File)

**The Crack:**
1. Exported the key to my attack machine.
2. Used `gpg2john` to convert the key to a hash.
3. Cracked the hash with **John the Ripper** and `rockyou.txt`:
   * **Passphrase:** `alexandru`

**The Result:**
Decrypting `credential.pgp` revealed credentials for the user **merlin**:
`merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

## 4. Privilege Escalation: ZIP Sudo Exploit
Switching to user `merlin`, I checked sudo permissions with `sudo -l`:
`(root : root) NOPASSWD: /usr/bin/zip`

**Exploitation:**
The `zip` binary can be abused to spawn a shell via the `--unzip-command` parameter.
```bash
sudo /usr/bin/zip /tmp/test.zip /tmp/test -T --unzip-command="sh -c /bin/sh"
