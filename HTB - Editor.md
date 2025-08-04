# Editor

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

<img width="718" height="606" alt="image" src="https://github.com/user-attachments/assets/6cec0550-f165-4630-a6f1-0c49f8a3cf8f" />

2. Enumerating port 80 only shows a page to download simplicode application. Enumerate the site on port 8080 and check robots.txt file

<img width="503" height="974" alt="image" src="https://github.com/user-attachments/assets/0ba4ea39-3bd0-4cf7-967b-203f33681b42" />

3. 
