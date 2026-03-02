# RDP Password Spray Investigation

## Technologies and Skills
- Microsoft Sentinel
- Kusto Query Language (KQL)
- Microsoft Defender for Endpoint (MDE)
- Microsoft Azure

## Background

Suspicious RDP login activity has been detected on a cloud-hosted Windows server. Multiple failed attempts were followed by a successful login, suggesting brute-force or password spraying behaviour. We are given 2 pieces of information to begin:
1. The compromised device name is "slflarewinsysmo"
2. The incident date is September 12, 2025

## Incident Summary

## Incident Timeline
| Timestamp (UTC) | Event |
| --- | --- |
| 9/16/2025, 7:46:23.147 PM | Password spray attack begins |
| 9/16/2025, 7:46:52.872 PM | Successful login |
| 9/16/2025, 7:38:40.063 PM | Attacker runs suspicious msupdate.exe command | 
| 9/16/2025, 7:39:45.516 PM | Attacker creates scheduled task MSUpdateService for persistence |
| 9/16/2025, 7:40:28.765 PM | Attacker begins enumerating environment |
| 9/16/2025, 7:43:20.873 PM | Attempt to exfiltrate backup_sync.zip file to remote IP 185.92.220.87. Attempt unsuccessful |


## Threat Hunt Process

#### 1. Attacker IP Address

MITRE Technique:
🔸 T1110.001 – Brute Force: Password Guessing

First, we want to find out the WHO in the incident. We look for the attacker's remote IP using this KQL query:

```
DeviceLogonEvents
| where DeviceName contains "flare"
| where isnotempty(RemoteIP)
| where TimeGenerated between (datetime('2025-09-13') .. datetime('2025-09-17'))
| order by TimeGenerated
| project TimeGenerated, DeviceName, ActionType, AccountDomain, AccountName, RemoteIP, RemoteDeviceName, LogonType
```

<img width="1666" height="605" alt="image" src="https://github.com/user-attachments/assets/b9cdd34f-7e29-4e94-a119-da38ea0cbe54" />

We see a rapid succession of LoginFailed followed by a successful login from the remote IP `159.26.106.84`. 

#### 2. Compromised Account

MITRE Technique:
🔸 T1078 – Valid Accounts

Looking at the logs, we discover that the same remote IP has a successful login via RDP. We take note of the compromised user account: `slflare`

<img width="510" height="284" alt="image" src="https://github.com/user-attachments/assets/6ce11591-3057-401e-bb6b-615814adf934" />

#### 3. Executed Binary Name

MITRE Techniques:
🔸 T1059.003 – Command and Scripting Interpreter: Windows Command Shell
🔸 T1204.002 – User Execution: Malicious File

A common procedure that attackers will do once logged into a host is:
1. Run malicious files from the User folder, Temp, or Public folders
2. Enumerate the machine and user (username, privileges, network connections, running processes, etc.)

We'll look for process events that looks for any processes triggered from commands that contain .exe, .ps1, or .bat (which are common Windows executable extensions) and from the Public, Downloads, Temp, or user folders (which are common locations for attackers to store malicious files on).

```
DeviceProcessEvents
| where DeviceName == "slflarewinsysmo"
| where InitiatingProcessAccountName == "slflare"
| where FolderPath has_any ("Public", "slflare", "Temp", "Downloads")
| where TimeGenerated between (todatetime('2025-09-16T18:46:23.1474699Z') .. datetime('2025-09-17'))
| where ProcessCommandLine has_any (".exe", ".ps1", ".bat")
| order by TimeGenerated
| project TimeGenerated, InitiatingProcessAccountName, FileName, FolderPath, InitiatingProcessCommandLine, ProcessCommandLine
```

<img width="1465" height="390" alt="image" src="https://github.com/user-attachments/assets/005136d7-8f57-484f-b7e6-37ceef756c0e" />

We come across an execution of the `msupdate.exe` script that looks unusual.

#### 4. Command Line Used to Execute Binary

MITRE Technique:
🔸 T1059 – Command and Scripting Interpreter

Suspicious command ran by attacker: `"msupdate.exe" -ExecutionPolicy Bypass -File C:\Users\Public\update_check.ps1`

#### 5. Persistence Mechanism Created

MITRE Technique:
🔸 T1053.005 – Scheduled Task/Job: Scheduled Task

Another common methodology of attackers is persistence. One way that attackers achieve this is through scheduled tasks:

```
DeviceProcessEvents
| where DeviceName == "slflarewinsysmo"
| where InitiatingProcessAccountName == "slflare"
| where TimeGenerated between (todatetime('2025-09-16T18:46:23.1474699Z') .. datetime('2025-09-17'))
| where ProcessCommandLine has_any (".exe", ".ps1", ".bat")
| order by TimeGenerated
| project TimeGenerated, InitiatingProcessAccountName, FileName, FolderPath, InitiatingProcessCommandLine, ProcessCommandLine, ProcessCreationTime
```

<img width="1680" height="785" alt="image" src="https://github.com/user-attachments/assets/f2a63e4c-c9e4-4216-bb53-e7c483241e10" />

#### 6. Defender Setting Modified

MITRE Technique:
🔸 T1562.001 – Impair Defenses: Disable or Modify Windows Defender

#### 7. Discovery Command Ran

MITRE Techniques:
🔸 T1082 – System Information Discovery

After achieving persistence, attackers would generally enumerate the environment. We find evidence of the attacker doing post-exploitation enumeration with this KQL query:

```
DeviceProcessEvents
| where DeviceName == "slflarewinsysmo"
| where InitiatingProcessAccountName == "slflare"
| where TimeGenerated between (todatetime('2025-09-16T18:46:23.1474699Z') .. datetime('2025-09-17'))
| where ProcessCommandLine has_any (".exe", ".ps1", ".bat")
| order by TimeGenerated
| project TimeGenerated, InitiatingProcessAccountName, FileName, FolderPath, InitiatingProcessCommandLine, ProcessCommandLine, ProcessCreationTime
```

<img width="1599" height="518" alt="image" src="https://github.com/user-attachments/assets/98f3564c-095b-480b-bf8a-5f90d5e44793" />

#### 8. Archive File Created By Attacker

MITRE Technique:
🔸 T1560.001 – Archive Collected Data: Local Archiving

Most attackers don't just access a machine without a purpose. They often try to steal and exfiltrate sensitive data. We'll check the SIEM logs for evidence of this by looking for any commands with common archiving extensions: .zip, .tar, .gz, and .7z.

```
DeviceProcessEvents
| where DeviceName == "slflarewinsysmo"
| where InitiatingProcessAccountName == "slflare"
| where TimeGenerated between (todatetime('2025-09-16T18:46:23.1474699Z') .. datetime('2025-09-17'))
| where ProcessCommandLine has_any ("zip", "tar", "gz", "7z")
| order by TimeGenerated
| project TimeGenerated, InitiatingProcessAccountName, FileName, FolderPath, InitiatingProcessCommandLine, ProcessCommandLine, ProcessCreationTime
```

<img width="1848" height="150" alt="image" src="https://github.com/user-attachments/assets/672ebc7f-d31d-4f16-a115-4d2333620c4d" />

We find evidence of the attacker creating the `backup_sync.zip` file.

#### 9. C2 Connection Destination

MITRE Techniques:
🔸 T1071.001 – Application Layer Protocol: Web Protocols (HTTP/S)
🔸 T1105 – Ingress Tool Transfer

Since the attacker created a zip file, it's natural to assume they would try to send it to a remote URL or server. We'll look at the network event logs for evidence of data being sent out:

```
DeviceNetworkEvents
| where DeviceName == "slflarewinsysmo"
| where InitiatingProcessCommandLine contains "backup_sync.zip"
| where TimeGenerated between (todatetime('2025-09-16T18:46:23.1474699Z') .. datetime('2025-09-17'))
```

<img width="1522" height="239" alt="image" src="https://github.com/user-attachments/assets/e003d15a-d835-4bdb-8f8c-7a78d68b87ed" />

The attacker did indeed try to send the zip file to the remote IP `185.92.220.87` on port 8081. Fortunately, the connection failed.
