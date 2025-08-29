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

11. 

