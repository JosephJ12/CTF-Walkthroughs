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

5. 
