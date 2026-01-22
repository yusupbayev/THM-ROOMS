# THM: CyberHeroes
**Difficulty:** Easy
**Focus:** Web Authentication Bypass, JavaScript Deobfuscation, Client-Side Logic Exploitation

---

## 🛡️ Executive Summary
CyberHeroes is a challenge that highlights the critical danger of relying on client-side security for authentication. In this scenario, the login logic is handled entirely within the user's browser via JavaScript. By analyzing and "reversing" the JavaScript code, I was able to identify the required credentials and bypass the login portal without any server-side interaction or database exploitation.

## 1. Reconnaissance & Enumeration
### Nmap Scan
The scan revealed a single primary point of entry:
* **80/http** - Apache httpd 2.4.41

### Web Interface
The landing page features a "CyberHeroes" login portal. Upon testing the form, I noticed that there were no network requests being sent to a backend server (no XHR or Fetch requests in the browser DevTools). This indicated that the authentication check was being performed locally by the browser.

## 2. Initial Access: JavaScript Deobfuscation
I inspected the page source (`Ctrl+U`) and found a JavaScript function responsible for the login logic.



### Analyzing the Code
The script contained a `CheckPassword()` function. The code used a series of string manipulations to verify the input. I observed that the username was hardcoded as a simple string, while the password was built using an array of characters or a reversed string.

**The "Aha!" Moment:**
By reading the script, I could see the exact string the code was looking for. 
1. **Username:** Identified as `h3ck3r1337`.
2. **Password:** I manually reconstructed the password by following the string concatenation logic in the script.

**Credentials Found:**
* **Username:** `h3ck3r1337`
* **Password:** `[REDACTED_PASSWORD_STRING]`

## 3. Flag Capture
Entering the discovered credentials into the login form triggered the client-side `success` condition. The browser then executed the logic to display the hidden flag.

---
**Flag Captured:** `THM{...}`
