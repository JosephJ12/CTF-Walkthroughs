# TombWatcher

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Medium**


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
![image](https://github.com/user-attachments/assets/38c052d5-4737-4dd6-a5ab-eddd228b28da)

2. With Henry's credentials, enumerate network using Bloodhound. Found high-level path to machine
![image](https://github.com/user-attachments/assets/acff890c-ed2b-45b7-a016-f0f7f93aebef)

3. 


