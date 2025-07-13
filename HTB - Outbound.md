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


2. Nmap shows only SSH and HTTP ports open. Enumerate subdomains and do dirbusting on machine

