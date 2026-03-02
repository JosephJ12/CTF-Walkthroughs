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

