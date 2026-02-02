# Virtual Machine Bruteforce Detection

## Skills and Technologies
- Microsoft Sentinel
- Kusto Query Language (KQL)
- Microsoft Azure
- Microsoft Defender for Endpoint (MDE)
- MITRE ATT&CK TTP
- NIST Framework

## Background

We are tasked with setting up an alert to detect bruteforce logon attempts on our systems. These are the parameters that we will set to determine whether a sequence of login attempts are considered a brute force attack:
- Timeframe of the logins are within the last 3 hours
- Login attempts are from the same remote IP address
- The count of failed logins is greater than 10

## Detection and Incident Creation

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

#### 3. Incident created for alert

We discover an incident is created for our rule, so we will follow the NIST 800-61 Incident Response Lifecycle

<img width="2245" height="713" alt="image" src="https://github.com/user-attachments/assets/a0c5eb8b-8ae2-4638-83b0-76cc4df4dafe" />

## NIST 800-61: Incident Response Lifecycle Process

#### 1. Preparation

This part of the process is mostly done by default, since we have Sentinel and alerts in place to dive deeper into this incident. We will assign the incident to ourselves and set the status to Active. Now, we'll go straight into phase 2.

<img width="465" height="277" alt="image" src="https://github.com/user-attachments/assets/84d0cdcb-a74f-4b15-bd7c-6ae2627d7c12" />

#### 2. Detection and Analysis

Clicking on Investigate brings us to this incident mapping

<img width="2559" height="874" alt="image" src="https://github.com/user-attachments/assets/2d3d1d85-805a-4c18-ad7b-25c092d5d36e" />

From here, we notice 2 things:
1. The hosts that were targets of multiple failed logins are `xj11-linux-scan` and `mde-test-cyn`
2. The remote IP addresses that attempted to login are `72.167.225.241` and `134.199.197.179`

With this information, our first task is to figure out the impact of these attacks- whether these bruteforce attempts resulted in a successful login or not. To investigate this, we'll run the following KQL query:

```
let ip1 = "72.167.225.241";
let ip2 = "134.199.197.179";
let device1 = "xj11-linux-scan";
let device2 = "mde-test-cyn";
DeviceLogonEvents
| where ActionType == "LogonSuccess"
| where DeviceName contains device1 or DeviceName == device2
| where RemoteIP in (ip1, ip2)
| where TimeGenerated < ago(3h)
```

<img width="826" height="614" alt="image" src="https://github.com/user-attachments/assets/86b4e271-586e-4461-82e8-ad8daa9e9271" />

Fortunately, there were no successful logins from these sources.

#### 3. Containment, Eradication, and Recovery

If the bruteforce login attempts were successful, we would isolate the compromised machine and trigger an antivirus scan on it. However, since this is not the case, we will look to prevent these attacks as much as possible in the future by updating our Network Security Group, or firewall, rules.

We create 2 new rules: one to allow only our local IP to connect via RDP and another to deny all other connection atttempts to our RDP port, 3389. One thing to make sure is that the rule to allow RDP connections from our local IP address has a higher (lower number) priority than the one to deny all RDP connection attempts since we still want to have access to our machine.

<img width="584" height="1220" alt="image" src="https://github.com/user-attachments/assets/9da6138d-fd80-4dd3-b99f-5dd4788e84ab" />

<img width="580" height="1221" alt="image" src="https://github.com/user-attachments/assets/6d9e11c3-faeb-4b0c-9bb9-56653ed8d0ab" />

#### 4. Post-Incident Activities

