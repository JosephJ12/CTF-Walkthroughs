# Authority

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
```
PORT      STATE SERVICE       VERSION
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-07 02:12:41Z)
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2025-11-07T02:13:46+00:00; +4h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2025-11-07T02:13:45+00:00; +4h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2025-11-07T02:13:46+00:00; +4h00m01s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2025-11-07T02:13:45+00:00; +4h00m01s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp  open  ssl/https-alt
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=172.16.2.118
| Not valid before: 2025-11-05T02:09:23
|_Not valid after:  2027-11-07T13:47:47
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 82
|     Date: Fri, 07 Nov 2025 02:12:48 GMT
|     Connection: close
|     <html><head><meta http-equiv="refresh" content="0;URL='/pwm'"/></head></html>
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET, HEAD, POST, OPTIONS
|     Content-Length: 0
|     Date: Fri, 07 Nov 2025 02:12:48 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 1936
|     Date: Fri, 07 Nov 2025 02:12:54 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1><hr class="line" /><p><b>Type</b> Exception Report</p><p><b>Message</b> Invalid character found in the HTTP protocol [RTSP&#47;1.00x0d0x0a0x0d0x0a...]</p><p><b>Description</b> The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49628/tcp open  msrpc         Microsoft Windows RPC
49641/tcp open  msrpc         Microsoft Windows RPC
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49693/tcp open  msrpc         Microsoft Windows RPC
49694/tcp open  msrpc         Microsoft Windows RPC
49702/tcp open  msrpc         Microsoft Windows RPC
49707/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8443-TCP:V=7.94SVN%T=SSL%I=7%D=11/6%Time=690D1D5F%P=x86_64-pc-linux
SF:-gnu%r(GetRequest,DB,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20text/htm
SF:l;charset=ISO-8859-1\r\nContent-Length:\x2082\r\nDate:\x20Fri,\x2007\x2
SF:0Nov\x202025\x2002:12:48\x20GMT\r\nConnection:\x20close\r\n\r\n\n\n\n\n
SF:\n<html><head><meta\x20http-equiv=\"refresh\"\x20content=\"0;URL='/pwm'
SF:\"/></head></html>")%r(HTTPOptions,7D,"HTTP/1\.1\x20200\x20\r\nAllow:\x
SF:20GET,\x20HEAD,\x20POST,\x20OPTIONS\r\nContent-Length:\x200\r\nDate:\x2
SF:0Fri,\x2007\x20Nov\x202025\x2002:12:48\x20GMT\r\nConnection:\x20close\r
SF:\n\r\n")%r(FourOhFourRequest,DB,"HTTP/1\.1\x20200\x20\r\nContent-Type:\
SF:x20text/html;charset=ISO-8859-1\r\nContent-Length:\x2082\r\nDate:\x20Fr
SF:i,\x2007\x20Nov\x202025\x2002:12:48\x20GMT\r\nConnection:\x20close\r\n\
SF:r\n\n\n\n\n\n<html><head><meta\x20http-equiv=\"refresh\"\x20content=\"0
SF:;URL='/pwm'\"/></head></html>")%r(RTSPRequest,82C,"HTTP/1\.1\x20400\x20
SF:\r\nContent-Type:\x20text/html;charset=utf-8\r\nContent-Language:\x20en
SF:\r\nContent-Length:\x201936\r\nDate:\x20Fri,\x2007\x20Nov\x202025\x2002
SF::12:54\x20GMT\r\nConnection:\x20close\r\n\r\n<!doctype\x20html><html\x2
SF:0lang=\"en\"><head><title>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\
SF:x20Request</title><style\x20type=\"text/css\">body\x20{font-family:Taho
SF:ma,Arial,sans-serif;}\x20h1,\x20h2,\x20h3,\x20b\x20{color:white;backgro
SF:und-color:#525D76;}\x20h1\x20{font-size:22px;}\x20h2\x20{font-size:16px
SF:;}\x20h3\x20{font-size:14px;}\x20p\x20{font-size:12px;}\x20a\x20{color:
SF:black;}\x20\.line\x20{height:1px;background-color:#525D76;border:none;}
SF:</style></head><body><h1>HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x
SF:20Request</h1><hr\x20class=\"line\"\x20/><p><b>Type</b>\x20Exception\x2
SF:0Report</p><p><b>Message</b>\x20Invalid\x20character\x20found\x20in\x20
SF:the\x20HTTP\x20protocol\x20\[RTSP&#47;1\.00x0d0x0a0x0d0x0a\.\.\.\]</p><
SF:p><b>Description</b>\x20The\x20server\x20cannot\x20or\x20will\x20not\x2
SF:0process\x20the\x20request\x20due\x20to\x20something\x20that\x20is\x20p
SF:erceived\x20to\x20be\x20a\x20client\x20error\x20\(e\.g\.,\x20malformed\
SF:x20request\x20syntax,\x20invalid\x20");
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_smb2-time: ERROR: Script execution failed (use -d to debug)
|_clock-skew: mean: 4h00m00s, deviation: 0s, median: 4h00m00s
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!
```

2. The open ports signals a classic Domain Controller. We first try anonymous authentication to SMB and LDAP without any luck. We notice there is an open HTTPS port on 8443 and we find a PWM login page. We try logging in with common passwords but all fails. So we shift our focus to the open SMB port.

<img width="853" height="585" alt="image" src="https://github.com/user-attachments/assets/423108da-20ff-4b39-b1fe-2596c29437de" />

3. We find that we're able to log into SMB anonymously.

`smbclient -L //authority.htb/ -N`

<img width="726" height="364" alt="image" src="https://github.com/user-attachments/assets/82ca7e99-5cd9-49fb-b41e-676056ff2f9e" />

4. Enumerating the share, we find a file encrypted by Ansible Vault. We download the file to our local machine for offline cracking.

<img width="748" height="568" alt="image" src="https://github.com/user-attachments/assets/f6d5bc2f-282a-422d-979c-1fc1713f4bf5" />

5. Before cracking these, we turn these hashes into a crackeable friendly format using the `ansible2john` script

`ansible2john ldap_admin_password.hash > ldap_admin_password.ansible2john`

<img width="1264" height="220" alt="image" src="https://github.com/user-attachments/assets/55cade06-ec75-4b30-9bee-d441e8c7797c" />

6. Crack using hashcat module 16900

`hashcat -m 16900 ldap_admin_password.ansible2john /usr/share/wordlists/rockyou.txt`

<img width="1266" height="356" alt="image" src="https://github.com/user-attachments/assets/aa758bad-7443-4650-96fb-9178fc126828" />

7. Turns out, all 3 ansible hashes are !@#$%^&*. We use this password to decrypt all 3  ansible vault hashes

`cat pwm_admin_login.hash | ansible-vault decrypt`

<img width="472" height="268" alt="image" src="https://github.com/user-attachments/assets/7628984f-84fc-456d-9613-3491fcba6715" />

8. This gives us the pwm admin creds: `svc_pwm:pWm_@dm!N_!23`. We use this password to log into the Configuration Editor of PWM.

<img width="889" height="241" alt="image" src="https://github.com/user-attachments/assets/0328cd65-073a-4d1e-853b-c081dc1b55a9" />

9. 
