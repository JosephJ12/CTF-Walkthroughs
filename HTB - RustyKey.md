# RustyKey

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

![image](https://github.com/user-attachments/assets/483ca663-e7d2-4944-b26e-5564e6d92d2d)

2. Run bloodhound using given credentials `rr.parker:8#t5HE8L!W3A`

![image](https://github.com/user-attachments/assets/3874dc11-7171-4dd9-9f61-b3472efb00f8)

3. We get an error with LDAP saying it can't find dc.rustykey.htb so add that to `/etc/hosts` file and try again.

`sudo bloodhound-python -d rustykey.htb -u rr.parker -p "8#t5HE8L\!W3A" -ns 10.129.63.201 -c all`

4. 
