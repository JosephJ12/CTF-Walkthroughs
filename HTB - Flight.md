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

<img width="821" height="653" alt="image" src="https://github.com/user-attachments/assets/1d31dac1-44e3-4fd5-a040-55f5b93b6bdd" />

2. The nmap scan reveals a bunch of open ports, including DNS (53), LDAP (389,3268) and Kerberos (88) meaning that this is a Domain Controller. We also see an open web server on port 80 that looks interesting. We add the domain to our `/etc/hosts` file and check out the website, which seems to just be static.

<img width="1008" height="842" alt="image" src="https://github.com/user-attachments/assets/b06483e9-7a32-4ccc-af01-d0e54784401d" />

3. As with all websites, we do subdomain enumeration to try to increase our attack surface using `ffuf`

`ffuf -u http://flight.htb -H "Host: FUZZ.flight.htb" -w /usr/share/wordlist/seclists`



