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


