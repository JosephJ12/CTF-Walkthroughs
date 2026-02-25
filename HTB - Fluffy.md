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

6. 

