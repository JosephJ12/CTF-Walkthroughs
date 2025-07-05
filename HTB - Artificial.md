# Artificial

**Platform: Hack the Box**

**OS: Linux**

**Diffculty: Easy**


## Key Learnings

- Become exposed to Tensorflow and exploiting h5 files
- Use Docker to run python script
- Become more familiar with enumerating a Linux machine
- Using Backrest commands to read sensitive files
- The hacker mindset of exploiting a backup feature to read the root flag


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

7. Doing basic enumeration on the machine shows an open internal port 9898. Using SSH, do local port forwarding

`ssh -L 1234:localhost:9898 gael@artificial.htb`

![image](https://github.com/user-attachments/assets/2ba63829-f10b-481e-8332-00100ee62f98)

8. Checking out the internal port 9898 reveals a Backrest web app

![image](https://github.com/user-attachments/assets/97e1e201-49cc-4645-8f79-dc577437b673)

9. After some enumeration, find a backrest backup zip file in /var folder. Use scp to download to local machine

`scp -r gael@artificial.htb:/var/backups/backrest_backup.tar.gz ./`

10. After unzipping the file, find a config.json file containing a Bcrypt password for backrest_root user

![image](https://github.com/user-attachments/assets/9761c14e-a2a9-436f-9350-ebee97f73d13)

11. The password hash is encoded with Base64 format. After decoding, crack the bcrypt hash using hashcat and we get backrest_root's password to log into the internal service.

![image](https://github.com/user-attachments/assets/9a49d679-00ab-4fe6-b0f7-904e5427a565)

12. Creating a repo allows us to run commands on the machine. Type help to get a list of commands

![image](https://github.com/user-attachments/assets/8e78c458-b992-4390-af17-959b8700be56)

13. My focus is on the backup command. Will attempt to run this command to create a backup of the /root directory and read its contents

`backup /root/root.txt`

![image](https://github.com/user-attachments/assets/3dc4b52a-6a02-4cb6-9892-671e0e9bdc0e)

14. Now use the dump command to get contents of root.txt file

`dump d623c61c /root/root.txt`

![image](https://github.com/user-attachments/assets/a16b8778-e2f7-484c-9511-16e350edcb67)

15. Got the root flag! Pwned machine

## Remediation Summary
- 




