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

4. With bloodhound, we find the path to user is through nn.marcos user

![image](https://github.com/user-attachments/assets/22162d58-fed1-421d-a1e1-36a639eef6ba)

5. However, with rr.parker, there is no way to nn.marcos. After much enumeration and dead ends, we try kerberoasting to gain initial access. But we get no results for it

`nxc ldap dc.rustykey.htb -u rr.parker -p "8#t5HE8L\!W3A" -k --kerberoasting kerberoast.txt`

![image](https://github.com/user-attachments/assets/f06dbd5b-a301-42a9-9034-1a996a147349)

6. Running it again with the debug flag this time, we see that by default, nxc searches only for computers. We change the filter to look for users instead and try again.

![image](https://github.com/user-attachments/assets/1bca6dc5-6fa5-4fc6-aa8b-afbfba610c98)

7. Kerberoasting successful! We got a hash

![image](https://github.com/user-attachments/assets/e4d8528d-03cd-468e-bbde-3ae0a0f6c1df)

8. Crack offline with hashcat

`hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt`

9. Hash doesn't crack, so we try kerberoasting with a different method. First, enumerate all usernames with ldapsearch

``

![image](https://github.com/user-attachments/assets/2f6787e2-1765-4f50-b0d9-c30819116bb2)

10. Then try kerberoasting using impacket's GetUserSPNs.py script using the `-usersfile` flag

``

![image](https://github.com/user-attachments/assets/dd7e5e13-5d06-4055-b210-bd046e3d64da)

11. Kerberoasting is a success and we get back 17 hashes. Try cracking them offline once more using hashcat

`hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt`

![image](https://github.com/user-attachments/assets/f791b6a7-ae86-412b-a5ed-ea7dbfe78616)

12. Got the password for machine account IT-Computer3$.





