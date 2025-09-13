# Soccer

**Platform: Hack the Box**

**OS: Linux**

**Diffculty: Easy**


## Key Learnings

- Become exposed to Tensorflow and exploiting h5 files
- Use Docker to run python script
- Doing local port forwarding to gain access to internal ports and services
- Become more familiar with enumerating a Linux machine
- Using Backrest commands to read sensitive files
- The hacker mindset of exploiting a backup feature to read the root flag


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan

`nmap -sC -sV -p- -Pn -T5 -oN soccer.htb`

<img width="913" height="1144" alt="image" src="https://github.com/user-attachments/assets/afdfcf9f-7c9b-488f-a5d2-affbc4dec15a" />

2. There are only 3 ports open: 22, 80 and 9091. Since SSH port 22 is a very uncommon route to exploit for initial foothold, we focus our attention on the other 2 ports. Particularly, a web server is one of the best places to start. We do subdomain enumeration and directory busting using ffuf.

`ffuf -u http://soccer.htb/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`

<img width="914" height="734" alt="image" src="https://github.com/user-attachments/assets/04b78668-82d5-4b60-80ed-da1f3f25b357" />

3. Found directory /tiny! We land on a Tiny File Manager login page. We try the default credentials of admin:admin@123 and we successfully login!

<img width="1982" height="547" alt="image" src="https://github.com/user-attachments/assets/f0a6be78-93f9-4af4-ab37-b2928d47083a" />

4. We check out the site and find that we can upload files to the /tiny/uploads/ directory! We upload a PHP reverse shell one liner and get a reverse shell back!

<img width="632" height="157" alt="image" src="https://github.com/user-attachments/assets/ee9cd616-cc80-4d2b-b492-1e2d89ac9a77" />

5. Looking at the /home directory, we find only 1 folder for /player. So assuming the next user we need is player, we look for all files that have "player" in it and we come across the soc-player subdomain

`grep -rnw "player" / 2>/dev/null`

<img width="926" height="204" alt="image" src="https://github.com/user-attachments/assets/49be6c01-e9ab-4106-987e-b31c4072ba57" />

6. Add the subdomain to our /etc/hosts file and go to the site. We sign up for the site and notice there's a call to port 9091 that we enumerated before with Nmap

<img width="1105" height="840" alt="image" src="https://github.com/user-attachments/assets/d9b6b8fd-9529-4412-af9c-3d63e9988731" />

<img width="1058" height="386" alt="image" src="https://github.com/user-attachments/assets/f69d7833-0e07-4410-bf43-a0d16deed17e" />

7. 
