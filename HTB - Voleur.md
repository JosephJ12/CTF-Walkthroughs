# Voleur

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Medium**


## Table of Contents
- [Key Learnings](#key-learnings)
- [Walkthrough](#walkthrough)
- [Remediation Summary](#remediation-summary)


## Key Learnings

- Familiarize myself with enumerating AD networks using Bloodhound
- Learn to abuse WriteOwner privilege using Impacket's owneredit.py script
- Give an account FullControl using Impacket's dacledit.py tool
- Enumerate and restore deleted user accounts on PowerShell
- Exploit ESC15 vulnerable template
- Add user to Domain Admin group with an LDAP shell


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan along with bloodhound

![image](https://github.com/user-attachments/assets/9374b63e-36de-4aa2-96d2-ef3957b8f57b)

2. Bloodhound doesn't give us a path from credentials given ryan.naylor. After enumerating using netexec smb, found an interesting IT share so download files with the spider_plus module

`nxc smb dc.voleur.htb -u ryan.naylor -p 'HollowOct31Nyt' -k -M spider_plus -o DOWNLOAD_FLAG=True`

![image](https://github.com/user-attachments/assets/ac3f931f-437f-47de-a391-d6399fff89d4)

3. Trying to open the .xlsx file shows that it's password protected, so use the office2john script and crack password offline.


