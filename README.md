# Multi-Stage Penetration Test — From Reconnaissance to Root

### **Overview**
In this repository, I've documented a multi-stage security assessment I performed in a controlled virtual environment. The project demonstrates a realistic attack chain I followed, beginning with initial network reconnaissance and culminating in full system compromise (root access).

### **Scope**
- **Environment**: I performed this test in a virtual lab (VirtualBox) using a Kali Linux attacker VM against an intentionally vulnerable target VM.
- **Target services**: My focus was on NFS (port 2049), Apache (ports 80 & 8080), and SSH (port 22).

### **Tools & Techniques I Used**
- **Nmap & Netdiscover**: For network and service discovery.
- **ffuf**: To enumerate web content and directories.
- **fcrackzip**: For offline password cracking of a protected archive I found.
- **Searchsploit**: To research public exploits for identified software.
- **GTFOBins**: To identify privilege escalation vectors.

### **My Key Findings**
- **Insecure NFS Share**: I found an improperly configured NFS share that exposed sensitive files, including a password-protected archive containing an SSH private key.
- **Information Disclosure**: I discovered a misconfigured web server that exposed plaintext database credentials (`bolt:i_love_java`) in a configuration file.
- **Local File Inclusion (LFI)**: I identified and exploited an LFI vulnerability in a second web application (BoltWire) to confirm a valid system username (`jeanpaul`).
- **Password Reuse**: I found that the password discovered in the web application's configuration file was reused for the `jeanpaul` user account, which allowed me to gain initial access via SSH.
- **Privilege Escalation via Sudo**: I discovered a misconfigured sudo rule that allowed my user to run the `zip` command as root, providing me with a direct path to full system compromise.

### **Impact**
- In a real-world scenario, the attack path I followed would allow an attacker to gain complete control over the server. This could lead to sensitive data theft, lateral movement into the network, or the server being used for malicious activities.

### **Remediation & Hardening (High-Level)**
- **Secure NFS**: Restrict NFS shares to authorized IP addresses and enforce strong permissions.
- **Credential Management**: Avoid storing plaintext credentials in configuration files; use secrets management tools instead.
- **Prevent Password Reuse**: Enforce policies that prevent the use of the same password across different services and users.
- **Principle of Least Privilege**: Sudo rules should be strictly limited. Never grant permissions for binaries that can easily be used to spawn a shell (e.g., `zip`, `vim`, `find`).
- **Patch Management**: Keep all web application software and server components up-to-date to mitigate known vulnerabilities like LFI.

### **Responsible Use & Authorization**
I performed this work solely in a controlled lab environment. All technical details enabling exploitation have been removed. Do not use these materials to target systems without explicit, written authorization.

### **Files in this Repository**
- `detailed-report.md` — The step-by-step walkthrough I wrote for the lab exercise.
- `images/` — A folder containing the screenshots I took during the assessment.
- `responsible_use.md` — Authorization and disclosure guidance.
