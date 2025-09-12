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

4. 
