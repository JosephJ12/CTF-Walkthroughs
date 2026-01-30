# Virtual Machine Bruteforce Detection

## Skills and Technologies
- Microsoft Sentinel
- Kusto Query Language (KQL)
- Microsoft Azure
- Microsoft Defender for Endpoint (MDE)
- MITRE ATT&CK TTP

## Background

We are tasked with setting up an alert to detect bruteforce logon attempts on our systems. These are the parameters that we will set to determine whether a sequence of login attempts are considered a brute force attack:
- Timeframe of the logins are within the last 3 hours
- Login attempts are from the same remote IP address
- The count of failed logins is greater than 10

## Incident Response Process

#### 1. Write a KQL query to detect bruteforce login attempts

We will query the `DeviceLogonEvents` table on Microsoft Sentinel with the parameters set above for determining bruteforce attacks:

```
DeviceLogonEvents
| where ActionType == "LogonFailed"
| where TimeGenerated < ago(3h)
| summarize LogonAttempts = count() by DeviceName, RemoteIP, bin(TimeGenerated, 1h)
| where LogonAttempts >= 10
| project TimeGenerated, DeviceName, RemoteIP, LogonAttempts
| order by TimeGenerated, LogonAttempts desc
```

<img width="1221" height="1073" alt="image" src="https://github.com/user-attachments/assets/4d1efb0a-1b49-4da5-afca-fda6767fa929" />

#### 2. Create a scheduled alert rule on Sentinel

We create a new alert rule that will trigger every 3 hours and run the KQL query to look for any bruteforce attempts.

<img width="1172" height="1273" alt="image" src="https://github.com/user-attachments/assets/b92e8cd2-b023-47a6-a925-e0ea600476ed" />

#### 3. 
