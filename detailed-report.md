Project Overview
This report documents the full process of a penetration test performed on a target machine in a controlled lab environment. The objective was to gain initial access, escalate privileges to root, and capture the final flag. This exercise demonstrates a realistic attack chain, combining multiple vulnerabilities to achieve a full system compromise.

Disclaimer: All activities were conducted in an isolated, virtual lab environment on a machine designed to be vulnerable for educational purposes.

## Phase 1: Reconnaissance 🕵️
The first step was to identify and map out the target.

### Host Discovery
I began by performing an ARP scan of the local network to find live hosts.

Tool: netdiscover

Result: A target was identified at [TARGET_IP].

! [netdiscover Results](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/1.png)
### Service Enumeration
With the target identified, I ran a comprehensive nmap scan to discover open ports and running services.

Command: nmap -sV -sC -O -A [TARGET_IP]

Key Findings:

SSH (Port 22): OpenSSH 7.9p1 was running.

HTTP (Port 80): An Apache server with a "Bolt - Installation error" page.

NFS (Port 2049): A Network File System service was active.

HTTP (Port 36363): Another Apache server, which was later found to host a phpinfo() page, confirming a significant information disclosure vulnerability.

! [Nmap Scan Results](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/2.png)
 

## Phase 2: Vulnerability Analysis & Exploitation 🔓
This phase involved investigating the services discovered by nmap.

### Exploiting the NFS Misconfiguration
The open NFS service was my first target.

Listing Shares: I used showmount -e [TARGET_IP] and found that the /srv/nfs directory was shared with a wide, insecure range of IPs.

Mounting the Share: I mounted the remote share to my local machine to inspect its contents.

Discovery: Inside the share, I found a password-protected file named save.zip. The unzip command revealed it contained two files: id_rsa and todo.txt.

! [Mounting NFS and Finding save.zip](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/7.png)

### Offline Attack & Credential Discovery
I moved the save.zip file for an offline password attack.

Tool: fcrackzip with the rockyou.txt wordlist.

Result: The password java101 was successfully recovered.

Analysis: After unzipping the file, I found:

id_rsa: A standard SSH private key.

todo.txt: A note signed with the initials "jp", giving me a clue for a potential username.

! [Cracking the Zip and its Contents](https://github.com/IamNotaThreatPenetration_Test-From_Reconnaissance_to_Root/blob/main/images/8.png)

## Phase 3: Pivoting to Web Vulnerabilities 🌐
While the SSH key was a strong lead, it was passphrase-protected. I pivoted back to the web servers to explore other paths.

### Content Discovery
I used ffuf to brute-force directories on both web servers. This revealed several key locations, including /app on port 80 and /dev on port 8080.

! [FFUF Scan Results](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/5.png)
! [FFUF Scan Results](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/6.png)

### Exploiting Information Disclosure
Directory Listing (Port 80): Navigating to [TARGET_IP]/app/ revealed that directory listing was enabled. I explored the directories and found plaintext database credentials in /app/config/config.yml:

Username: bolt

Password: i_love_java

LFI Exploit (Port 8080): I identified the application on port 8080 as BoltWire. Using searchsploit, I found a public Local File Inclusion (LFI) exploit. I used a directory traversal payload (.../../etc/passwd) to read the /etc/passwd file, which confirmed the existence of the user jeanpaul. This correlated perfectly with the "jp" signature found earlier.

! [LFI Exploit Showing /etc/passwd](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/16.png)

## Phase 4: Initial Access & Privilege Escalation 🚀
With all the pieces gathered, it was time to gain access and escalate.

### Gaining a Foothold (Initial Access)
I leveraged the principle of password reuse. I combined the username jeanpaul (confirmed via LFI) with the password i_love_java (found in the config.yml file) and successfully logged in via SSH.

! [Successful SSH Login](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/18.png)

### Privilege Escalation to Root
Immediately after logging in, I checked my sudo privileges.

Enumeration: I ran sudo -l. The output showed that my user, jeanpaul, could run the /usr/bin/zip command as the root user without a password.

Exploitation: I referenced GTFOBins for known methods to escalate privileges with zip. By using a specific command that leverages zip's ability to execute shell commands, I successfully spawned a root shell.

System Compromise: I confirmed my identity as root with the whoami command and captured the final flag from the /root directory.

! [Final Root Shell and Flag](https://github.com/IamNotaThreat/Penetration_Test-From_Reconnaissance_to_Root/blob/main/images/20.png)

## Conclusion & Key Takeaways
This engagement successfully demonstrated how multiple, seemingly separate vulnerabilities can be chained together to achieve full system compromise. The key takeaways include:

Secure Configurations: Insecure services like NFS and misconfigured web servers (directory listing) provide easy entry points.

Credential Management: Plaintext credentials should never be stored in configuration files.

Password Reuse: The reuse of the password "i_love_java" was the critical link that allowed for initial access.

Principle of Least Privilege: The sudo rule allowing a user to run zip as root is a severe violation of this principle and led directly to the final compromise.
