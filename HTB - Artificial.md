# Artificial

**Platform: Hack the Box**

**OS: Linux**

**Diffculty: Easy**


## Key Learnings

- Familiarize myself with enumerating AD networks using Bloodhound
- Abuse Active Directory DACLs 
- Successfully execute a shadow credential attack via Pywhisker
- Enumerate vulnerable certificate templates and exploit a ESC9 template misconfiguration using Certipy


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan

![image](https://github.com/user-attachments/assets/5d57053c-f6a5-4bf4-bfc6-0a00a89e4c0a)

2. Since SSH doesn't have many vulnerabilities, we focus on the only other port, the web service running on port 80. Checking out the website, we register an account and come across an app with a file upload feature.

![image](https://github.com/user-attachments/assets/2952513a-46b4-48ed-8945-d01ba04d0e8e)

3. There's a dockerfile and file upload functionality that accepts .h5 tensorflow model files. After googling for some exploits, found a this (POC)[https://github.com/Splinter0/tensorflow-rce/tree/main]. Using docker, generate a h5 file that has an RCE payload embedded in it.

`sudo docker build -t artificial .`
`sudo docker run --rm -v "$PWD:/code" -w/code --entrypoint python3 artificial exploit.py`

![image](https://github.com/user-attachments/assets/51ea9576-fbc5-4904-9a98-61deaaa1509e)

4. We got back a exploit.h5 file that we can upload and run on the web app to get RCE. Run netcat and run the RCE h5 file and we get back a shell!

![image](https://github.com/user-attachments/assets/1786e142-af25-424b-8b7d-b8fd735ee259)

5. After enumerating the app folder, discovered a users.db file that contains user hashes. The one in particular we try to crack is the one for user gael, since gael had a folder in the /home directory. Crack hash using hashcat

`hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt`

![image](https://github.com/user-attachments/assets/74aa26db-1f51-467a-b35c-41537d87fbfd)

6. Using gael's creds, SSH into the machine and get user flag!

![image](https://github.com/user-attachments/assets/bd212cde-9674-47af-bbbb-80d52265bab7)

7. 




