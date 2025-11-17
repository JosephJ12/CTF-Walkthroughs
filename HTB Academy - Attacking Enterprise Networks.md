# Attacking Enterprise Networks

**Platform: Hack the Box Academy**

**OS: Windows**


## **Disclaimer: Potential spoilers below**

## Scope

<img width="1504" height="660" alt="image" src="https://github.com/user-attachments/assets/fb7f6597-3abf-4495-8074-4e4f73d8cae8" />

## Walkthrough

1. Do a ping sweep of external IP range with Nmap

```
nmap -sn 10.129.0.0/16 -oN alive_ips.txt --min-rate 3000 -T5

# Nmap 7.94SVN scan initiated Sun Nov 16 22:29:25 2025 as: nmap -sn -oN alive_ips.txt --min-rate 3000 -T5 10.129.0.0/16
Nmap scan report for 10.129.0.1
Nmap scan report for 10.129.2.141
Nmap scan report for 10.129.2.219
Nmap scan report for 10.129.15.95
Nmap scan report for 10.129.35.28
Nmap scan report for 10.129.42.254
Nmap scan report for 10.129.43.4
Nmap scan report for 10.129.48.182
Nmap scan report for 10.129.59.248
Nmap scan report for 10.129.120.171
Nmap scan report for 10.129.124.236
Nmap scan report for 10.129.126.149
Nmap scan report for 10.129.127.86
Nmap scan report for 10.129.173.143
Nmap scan report for 10.129.191.157
Nmap scan report for 10.129.203.22
Nmap scan report for 10.129.204.23
Nmap scan report for 10.129.234.170
Nmap scan report for 10.129.252.88
# Nmap done at Sun Nov 16 22:32:26 2025 -- 65536 IP addresses (19 hosts up) scanned in 181.10 seconds
```

2. Format all alive hosts to only get the IP address from output

```
cat alive_ips.txt | grep Nmap | cut -d ' ' -f5 > alive_ips_formatted.txt

cat alive_ips_formatted.txt
10.129.0.1
10.129.2.141
10.129.2.219
10.129.15.95
10.129.35.28
10.129.42.254
10.129.43.4
10.129.48.182
10.129.59.248
10.129.120.171
10.129.124.236
10.129.126.149
10.129.127.86
10.129.173.143
10.129.191.157
10.129.203.22
10.129.204.23
10.129.234.170
10.129.252.88
```

3. Do a full port scan of all alive hosts

```
nmap -sV -Pn -p- -T5 --min-rate 3000 -oN initial_scan -iL alive_ips_formatted.txt

Nmap scan report for 10.129.0.1
Host is up.
All 65535 scanned ports on 10.129.0.1 are in ignored states.
Not shown: 65535 filtered tcp ports (no-response)

Nmap scan report for 10.129.2.141
Host is up (0.066s latency).
Not shown: 62752 closed tcp ports (reset), 2771 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Nmap scan report for 10.129.2.219
Host is up (0.066s latency).
Not shown: 62896 closed tcp ports (reset), 2630 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.5e
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd (Ubuntu)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd (Ubuntu)
995/tcp open  ssl/pop3    Dovecot pop3d
Service Info: Host: NIXFUND; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.15.95
Host is up (0.067s latency).
Not shown: 62832 closed tcp ports (reset), 2699 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
3000/tcp open  http    Node.js Express framework
3001/tcp open  http    PHP cli server 5.5 or later
3002/tcp open  http    Node.js Express framework
3003/tcp open  http    PHP cli server 5.5 or later (PHP 7.4.3)

Nmap scan report for 10.129.35.28
Host is up (0.066s latency).
Not shown: 62847 closed tcp ports (reset), 2681 filtered tcp ports (no-response)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http        Apache httpd 2.4.29 ((Ubuntu))
110/tcp   open  pop3        Dovecot pop3d
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd (Ubuntu)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
31337/tcp open  Elite?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.94SVN%I=7%D=11/16%Time=691AA7EC%P=x86_64-pc-linux-gnu
SF:%r(GetRequest,1F,"220\x20HTB{pr0F7pDv3r510nb4nn3r}\r\n")%r(SIPOptions,1
SF:F,"220\x20HTB{pr0F7pDv3r510nb4nn3r}\r\n");
Service Info: Host: NIX-NMAP-DEFAULT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.42.254
Host is up (0.066s latency).
Not shown: 62725 closed tcp ports (reset), 2803 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.41 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 4.6.2
445/tcp  open  netbios-ssn Samba smbd 4.6.2
2323/tcp open  telnet      Linux telnetd
8080/tcp open  http        Apache Tomcat
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.43.4
Host is up (0.066s latency).
Not shown: 62766 closed tcp ports (reset), 2767 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
3389/tcp open  ms-wbt-server xrdp
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.48.182
Host is up (0.066s latency).
Not shown: 62866 closed tcp ports (reset), 2656 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 10.129.59.248
Host is up (0.066s latency).
Not shown: 62871 closed tcp ports (reset), 2662 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.120.171
Host is up (0.066s latency).
Not shown: 62864 closed tcp ports (reset), 2651 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
22/tcp    open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5800/tcp  open  http-proxy    sslstrip
5900/tcp  open  vnc           VNC (protocol 3.8)
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap scan report for 10.129.124.236
Host is up (0.066s latency).
Not shown: 62879 closed tcp ports (reset), 2647 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.5e
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd (Ubuntu)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd (Ubuntu)
995/tcp open  ssl/pop3    Dovecot pop3d
Service Info: Host: NIXFUND; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.126.149
Host is up (0.067s latency).
Not shown: 62295 closed tcp ports (reset), 3237 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
22/tcp   open  tcpwrapped
7777/tcp open  tcpwrapped
9000/tcp open  tcpwrapped

Nmap scan report for 10.129.127.86
Host is up (0.066s latency).
Not shown: 62859 closed tcp ports (reset), 2665 filtered tcp ports (no-response)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
25/tcp   open  smtp     Postfix smtpd
53/tcp   open  domain   (unknown banner: 1337_HTB_DNS)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
110/tcp  open  pop3     Dovecot pop3d
111/tcp  open  rpcbind  2-4 (RPC #100000)
143/tcp  open  imap     Dovecot imapd (Ubuntu)
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp  open  ssl/pop3 Dovecot pop3d
8080/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.94SVN%I=7%D=11/16%Time=691AA7FE%P=x86_64-pc-linux-gnu%r(
SF:DNSVersionBindReqTCP,39,"\x007\0\x06\x85\0\0\x01\0\x01\0\0\0\0\x07versi
SF:on\x04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\r\x0c1337_HTB_DN
SF:S");
Service Info: Host:  ubuntu; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.173.143
Host is up (0.066s latency).
Not shown: 62846 closed tcp ports (reset), 2680 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.5e
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd (Ubuntu)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd (Ubuntu)
995/tcp open  ssl/pop3    Dovecot pop3d
Service Info: Host: NIXFUND; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.191.157
Host is up (0.066s latency).
Not shown: 62886 closed tcp ports (reset), 2646 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
7777/tcp open  http        Apache httpd 2.4.41 ((Ubuntu))
9000/tcp open  cslistener?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9000-TCP:V=7.94SVN%I=7%D=11/16%Time=691AA806%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,FD,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nDate:\x20Mon,\x2
SF:017\x20Nov\x202025\x2004:43:50\x20GMT\r\nConnection:\x20close\r\nConten
SF:t-Type:\x20text/plain;\x20charset=UTF-8\r\nContent-Length:\x20107\r\n\r
SF:\nCannot\x20establish\x20effective\x20URI\x20of\x20request\x20to\x20`/`
SF:,\x20request\x20has\x20a\x20relative\x20URI\x20and\x20is\x20missing\x20
SF:a\x20`Host`\x20header")%r(HTTPOptions,FD,"HTTP/1\.1\x20400\x20Bad\x20Re
SF:quest\r\nDate:\x20Mon,\x2017\x20Nov\x202025\x2004:43:50\x20GMT\r\nConne
SF:ction:\x20close\r\nContent-Type:\x20text/plain;\x20charset=UTF-8\r\nCon
SF:tent-Length:\x20107\r\n\r\nCannot\x20establish\x20effective\x20URI\x20o
SF:f\x20request\x20to\x20`/`,\x20request\x20has\x20a\x20relative\x20URI\x2
SF:0and\x20is\x20missing\x20a\x20`Host`\x20header")%r(RTSPRequest,EA,"HTTP
SF:/1\.1\x20505\x20HTTP\x20Version\x20Not\x20Supported\r\nDate:\x20Mon,\x2
SF:017\x20Nov\x202025\x2004:43:50\x20GMT\r\nConnection:\x20close\r\nConten
SF:t-Type:\x20text/plain;\x20charset=UTF-8\r\nContent-Length:\x2074\r\n\r\
SF:nThe\x20server\x20does\x20not\x20support\x20the\x20HTTP\x20protocol\x20
SF:version\x20used\x20in\x20the\x20request\.")%r(RPCCheck,A8,"HTTP/1\.1\x2
SF:0400\x20Bad\x20Request\r\nDate:\x20Mon,\x2017\x20Nov\x202025\x2004:43:5
SF:0\x20GMT\r\nConnection:\x20close\r\nContent-Type:\x20text/plain;\x20cha
SF:rset=UTF-8\r\nContent-Length:\x2023\r\n\r\nUnsupported\x20HTTP\x20metho
SF:d")%r(DNSVersionBindReqTCP,A8,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nDa
SF:te:\x20Mon,\x2017\x20Nov\x202025\x2004:43:51\x20GMT\r\nConnection:\x20c
SF:lose\r\nContent-Type:\x20text/plain;\x20charset=UTF-8\r\nContent-Length
SF::\x2023\r\n\r\nUnsupported\x20HTTP\x20method")%r(SSLSessionReq,A8,"HTTP
SF:/1\.1\x20400\x20Bad\x20Request\r\nDate:\x20Mon,\x2017\x20Nov\x202025\x2
SF:004:44:04\x20GMT\r\nConnection:\x20close\r\nContent-Type:\x20text/plain
SF:;\x20charset=UTF-8\r\nContent-Length:\x2023\r\n\r\nUnsupported\x20HTTP\
SF:x20method")%r(TerminalServerCookie,A8,"HTTP/1\.1\x20400\x20Bad\x20Reque
SF:st\r\nDate:\x20Mon,\x2017\x20Nov\x202025\x2004:44:04\x20GMT\r\nConnecti
SF:on:\x20close\r\nContent-Type:\x20text/plain;\x20charset=UTF-8\r\nConten
SF:t-Length:\x2023\r\n\r\nUnsupported\x20HTTP\x20method");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.203.22
Host is up (0.066s latency).
Not shown: 62773 closed tcp ports (reset), 2760 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.204.23
Host is up (0.067s latency).
Not shown: 62722 closed tcp ports (reset), 2797 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
2222/tcp  open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49672/tcp open  msrpc         Microsoft Windows RPC
Service Info: OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Nmap scan report for 10.129.234.170
Host is up (0.066s latency).
Not shown: 62858 closed tcp ports (reset), 2676 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

Nmap scan report for 10.129.252.88
Host is up (0.066s latency).
Not shown: 62932 closed tcp ports (reset), 2589 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Nov 16 22:46:13 2025 -- 19 IP addresses (19 hosts up) scanned in 640.32 seconds
```

5. Install EyeWitness and run it against alive IP addresses to get screenshots of Website

`python3 Python/EyeWitness.py --only-ports 80,8080 -f ../alive_ips_formatted.txt`

6. IP address to note: 10.129.127.86. Add inlanefreight.local to /etc/hosts file and do subdomain enumeration using ffuf

```
ffuf -u http://10.129.127.86/ -H 'Host: FUZZ.inlanefreight.local' -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt  -fs 15157

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.127.86/
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 :: Header           : Host: FUZZ.inlanefreight.local
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 15157
________________________________________________

careers                 [Status: 200, Size: 51806, Words: 22041, Lines: 732, Duration: 70ms]
blog                    [Status: 200, Size: 8708, Words: 1509, Lines: 232, Duration: 2270ms]
status                  [Status: 200, Size: 878, Words: 105, Lines: 43, Duration: 84ms]
support                 [Status: 200, Size: 26635, Words: 11730, Lines: 523, Duration: 4142ms]
dev                     [Status: 200, Size: 2048, Words: 643, Lines: 74, Duration: 79ms]
tracking                [Status: 200, Size: 35211, Words: 10413, Lines: 791, Duration: 77ms]
vpn                     [Status: 200, Size: 1578, Words: 414, Lines: 35, Duration: 75ms]
ir                      [Status: 200, Size: 28548, Words: 2885, Lines: 210, Duration: 1301ms]
monitoring              [Status: 200, Size: 56, Words: 3, Lines: 4, Duration: 74ms]
:: Progress: [87664/87664] :: Job [1/1] :: 591 req/sec :: Duration: [0:02:49] :: Errors: 0 ::
```

7. Add all the subdomains to /etc/host file and do directory busting on them using ffuf

`inlanefreight.local blog.inlanefreight.local dev.inlanefreight.local careers.inlanefreight.local ir.inlanefreight.local monitoring.inlanefreight.local status.inlanefreight.local support.inlanefreight.local tracking.inlanefreight.local vpn..inlanefreight.local`

```
ffuf -w subdomains.txt:URL -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt:FUZZ -u http://URL/FUZZ
```

8. 
