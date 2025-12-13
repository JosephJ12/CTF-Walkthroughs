# Dante

**Platform: HacktheBox Pro Labs**

**OS: Linux/Windows**


## Key Learnings

- Become exposed to Tensorflow and exploiting h5 files
- Use Docker to run python script
- Doing local port forwarding to gain access to internal ports and services
- Become more familiar with enumerating a Linux machine
- Using Backrest commands to read sensitive files
- The hacker mindset of exploiting a backup feature to read the root flag


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Do a ping sweep of the external network to discover alive hosts

```
$ nmap -sn -T5 --min-rate=1000 -oN external_ping_scan 10.10.110.0/24

Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-13 16:35 CST
Nmap scan report for 10.10.110.2
Host is up (0.066s latency).
Nmap scan report for 10.10.110.100
Host is up (0.067s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.47 seconds
```

Discovered to hosts on the 10.10.110/24 network: .2 and .100

2. Do a full service scan of all ports for the 2 alive hosts

```
$ nmap -sC -sV -T5 --min-rate=1000 -p- -Pn -oN external_service_scan -iL external_hosts 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-12-13 16:38 CST
Nmap scan report for 10.10.110.2
Host is up.
All 65535 scanned ports on 10.10.110.2 are in ignored states.
Not shown: 65535 filtered tcp ports (no-response)

Nmap scan report for 10.10.110.100
Host is up (0.066s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.12
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV IP 172.16.1.100 is not the same as 10.10.110.100
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 8f:a2:ff:cf:4e:3e:aa:2b:c2:6f:f4:5a:2a:d9:e9:da (RSA)
|   256 07:83:8e:b6:f7:e6:72:e9:65:db:42:fd:ed:d6:93:ee (ECDSA)
|_  256 13:45:c5:ca:db:a6:b4:ae:9c:09:7d:21:cd:9d:74:f4 (ED25519)
65000/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/wordpress DANTE{Y0u_Cant_G3t_at_m3_br0!}
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 2 IP addresses (2 hosts up) scanned in 126.36 seconds
```
The nmap output shows 1 host that has open ports, 10.10.110.100, which has 2 open ports of interest, FTP on 21 and HTTP on 65000. 

3. We first test FTP. We try to login anonymously, which is successful. We then proceed to enumerate and exfiltrate any files we can get our hands on. We come across a `todo.txt` file that we download to our local machine.

```
$ ftp 10.10.110.100 
Connected to 10.10.110.100.
220 (vsFTPd 3.0.3)
Name (10.10.110.100:root): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||12604|)
ftp: Can't connect to `10.10.110.100:12604': Connection timed out
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    4 0        0            4096 Apr 14  2021 Transfer
226 Directory send OK.
ftp> cd Transfer
250 Directory successfully changed.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Apr 14  2021 Incoming
drwxr-xr-x    2 0        0            4096 Aug 04  2020 Outgoing
226 Directory send OK.
ftp> cd Incoming
250 Directory successfully changed.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             310 Aug 04  2020 todo.txt
226 Directory send OK.
ftp> get todo.txt
local: todo.txt remote: todo.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for todo.txt (310 bytes).
100% |*************************************************************************************************|   310       31.28 KiB/s    00:00 ETA
226 Transfer complete.
310 bytes received in 00:00 (3.95 KiB/s)
ftp> cd ../Outgoing
250 Directory successfully changed.
ftp> ls
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> exit
221 Goodbye.
```

4. Before moving on to test port 65000, lets read the contents of the `todo.txt`

```
$ cat todo.txt 
- Finalize Wordpress permission changes - PENDING
- Update links to to utilize DNS Name prior to changing to port 80 - PENDING
- Remove LFI vuln from the other site - PENDING
- Reset James' password to something more secure - PENDING
- Harden the system prior to the Junior Pen Tester assessment - IN PROGRESS
```

5. The nmap scan for port 65000 output tells us about a `/wordpress` directory, and the contents of this file indicate that we are likely dealing with a Wordpress site. 

6. We browse to the URL `http://10.10.110.100:65000/wordpress` and confirm our findings.

<img width="1400" height="1188" alt="image" src="https://github.com/user-attachments/assets/d9e30d08-39a8-4b7b-9fc4-4b7efdd41f87" />
