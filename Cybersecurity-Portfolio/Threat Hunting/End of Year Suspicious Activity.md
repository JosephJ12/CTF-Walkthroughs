# End of Year Suspicious Activity

## Technologies and Skills
- Microsoft Sentinel
- Kusto Query Language (KQL)
- Microsoft Defender for Endpoint (MDE)
- Microsoft Azure

## Background

At the onset of December, routine monitoring detects irregular access patterns during year-end compensation and performance review activities. 

What initially appears as legitimate administrative and departmental behavior reveals a multi-stage sequence involving unauthorized script execution, sensitive file access, data staging, persistence mechanisms, and outbound communication attempts. 

We are tasked with correlating endpoint telemetry across multiple user contexts and systems to reconstruct the full access chain and determine how year-end bonus and performance data was accessed, prepared, and transmitted.

## Threat Hunt Process

#### 1. Initial Enumeration

We start the threat hunt with the following information:
- the host name on which suspicious activity is detected is `sys1-dept`
- the suspicious account is `5y51-d3p7`

We first try to get a general layout of what happened using the following KQL query on Sentinel:
```
DeviceProcessEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
| project TimeGenerated, FileName, InitiatingProcessAccountName, InitiatingProcessCommandLine, ProcessCommandLine, InitiatingProcessRemoteSessionIP, InitiatingProcessRemoteSessionDeviceName
```

<img width="2788" height="1320" alt="image" src="https://github.com/user-attachments/assets/67374813-44bc-427b-9543-a62ce6c6bb54" />

Our initial enumeration reveals the following information:
1. Remote connection from 192.168.0.110
2. Remote session name is YE-HELPDESKTECH
3. Ran a powershell script with the following command: `
"powershell.exe" -ExecutionPolicy Bypass -File C:\Users\5y51-D3p7\Downloads\PayrollSupportTool.ps1`
4. Enumerated basic information about the environment, such as account information, what tasks are running, and network information.

#### 2. Impact of Suspicious Actor

Now that we have a general understanding of what the actor did upon entering, we'll look for the impact. Is there evidence of any sensitive files being accessed or modified? Was there any sensitive information exfiltrated? We'll look into that.

<img width="2432" height="1326" alt="image" src="https://github.com/user-attachments/assets/ec2671a1-c090-4136-a4e8-41efc348ea0e" />

Looking further at the results of the previous query, we find evidence of the suspicious user accessing the `BonusMatrix_Draft_v3.xlsx` Excel file. The next thing to look for is if there are traces of data being exfiltrated, such as a zip file or archive being made.

```
DeviceFileEvents
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessAccountDomain == "sys1-dept"
| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
| order by TimeGenerated
| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessRemoteSessionDeviceName, InitiatingProcessRemoteSessionIP, InitiatingProcessUniqueId
```

<img width="2578" height="672" alt="image" src="https://github.com/user-attachments/assets/2543e115-d585-48f4-9440-0fa434288948" />

We find traces of a zip file being made, along with a HTML file from example.com. This most likely corresponds to the `nslookup.exe example.com` command that the suspicious user ran. To confirm whether a connection to example.com was made, we look at the `DeviceNetworkEvents` logs:

```
DeviceNetworkEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
| order by TimeGenerated
| project TimeGenerated, ActionType, InitiatingProcessCommandLine, InitiatingProcessFileName, RemoteIP, RemotePort, RemoteUrl, InitiatingProcessRemoteSessionDeviceName
```

<img width="2586" height="680" alt="image" src="https://github.com/user-attachments/assets/753a46c8-d728-4de9-bd12-faebcb467383" />

Indeed, there was a connection made to example.com in the logs. So far in our investigation, we've found evidence of the user doing the following sequence in order:
1. Accessing sensitive data
2. Creating a test zip file
3. Testing a remote connection to a dummy URL

Since the test attempt to exfiltrate data was successful, we can suspect that the user attempted to repeat the process with real data. Therefore, we will look deeper into the incident to find out whether the attacker successfully exfiltrated real, sensitive data.

#### 3. Sensitive Data Exfiltration Deep Dive

Since the suspicious user created a test `.zip` file, we can assume that they'll do the same for the actual attempt. We look in the logs for creation of zip files:

```
DeviceFileEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where FileName endswith ".zip"
| order by TimeGenerated desc
| project TimeGenerated, ActionType,FileName, FolderPath, InitiatingProcessCommandLine, InitiatingProcessRemoteSessionDeviceName
```

<img width="2514" height="738" alt="image" src="https://github.com/user-attachments/assets/827e4132-cd04-4260-a99b-afb4716f10c3" />

Indeed, there's another instance of a zip file being created just an hour after the other one. This time, the remote device name is `YE-HRPLANNER`. We'll repeat the threat hunting process as before and look for network events by `YE-HRPLANNER`.

```
DeviceNetworkEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessRemoteSessionDeviceName == "YE-HRPLANNER"
| order by TimeGenerated
| project TimeGenerated, ActionType, InitiatingProcessCommandLine, InitiatingProcessFileName, RemoteIP, RemotePort, RemoteUrl, InitiatingProcessRemoteSessionDeviceName
```

<img width="2438" height="468" alt="image" src="https://github.com/user-attachments/assets/9dbf53e4-e570-4170-8ffa-dc98ace63fa6" />

There indeed is a successful outbound connection to `httpbinorg`. Since the time of the creation of the zip file is 7:26:03 UTC and the time of the connection is 7:26:28 UTC, we can suspect that the zip file was indeed sent out. 

#### 4. Checking for Persistence

Since there are signs of data exfiltration, we first notify our SOC lead and await for futher instructions. In the meantime, we will look for a common procedure attackers do once they gain access to a system: persistence.

We first check for any account creation or modification of passwords. Although this would be very unlikely since it is very noisy and this kind of activity would've been flagged already, we still do our due diligence to check. As expected, we don't find anything in the logs.

Next, another common persistence method is by modifying registry keys. We look at the registry event logs for any suspicious values:

```
DeviceRegistryEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
| project TimeGenerated, ActionType, RegistryKey, RegistryValueData, RegistryValueName
```

<img width="2520" height="934" alt="image" src="https://github.com/user-attachments/assets/36a0bf92-b7a4-4e4a-9e22-0200e032c747" />

We discover a autorun registry key value being set to the custom powershell script run by the suspicious user. We also find a scheduled task creation that runs the `PayrollSupportTool.ps1` powershell script daily.

```
DeviceEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where ActionType == "ScheduledTaskCreated"
| order by TimeGenerated
```

<img width="2524" height="504" alt="image" src="https://github.com/user-attachments/assets/652c46ee-9e9f-4c30-9857-4b65a4b75334" />


## Queries Used

```
DeviceFileEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
| project TimeGenerated, ActionType,FileName, FolderPath, InitiatingProcessCommandLine, RequestSourceIP

DeviceProcessEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
//| project TimeGenerated, FileName, InitiatingProcessAccountName, InitiatingProcessCommandLine, ProcessCommandLine, InitiatingProcessRemoteSessionIP, InitiatingProcessRemoteSessionDeviceName

DeviceFileEvents
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessAccountDomain == "sys1-dept"
| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
//| where InitiatingProcessUniqueId == "2533274790396713"
| order by TimeGenerated
| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessRemoteSessionDeviceName, InitiatingProcessRemoteSessionIP, InitiatingProcessUniqueId

DeviceNetworkEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
| order by TimeGenerated
| project TimeGenerated, ActionType, InitiatingProcessCommandLine, InitiatingProcessFileName, RemoteIP, RemotePort, RemoteUrl, InitiatingProcessRemoteSessionDeviceName
```


## Queries Used 2/4
```
DeviceFileEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where FileName endswith ".zip"
| order by TimeGenerated desc
| project TimeGenerated, ActionType,FileName, FolderPath, InitiatingProcessCommandLine, InitiatingProcessRemoteSessionDeviceName

DeviceFileEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
//| project TimeGenerated, ActionType,FileName, FolderPath, InitiatingProcessCommandLine, RequestSourceIP

DeviceProcessEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
| project TimeGenerated, FileName, InitiatingProcessAccountName, InitiatingProcessCommandLine, ProcessCommandLine, InitiatingProcessRemoteSessionIP, InitiatingProcessRemoteSessionDeviceName

DeviceFileEvents
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessAccountDomain == "sys1-dept"
| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
//| where InitiatingProcessUniqueId == "2533274790396713"
| order by TimeGenerated
//| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessRemoteSessionDeviceName, InitiatingProcessRemoteSessionIP, InitiatingProcessUniqueId

DeviceNetworkEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
//| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
| where InitiatingProcessRemoteSessionDeviceName == "YE-HRPLANNER"
| order by TimeGenerated
//| project TimeGenerated, ActionType, InitiatingProcessCommandLine, InitiatingProcessFileName, RemoteIP, RemotePort, RemoteUrl, InitiatingProcessRemoteSessionDeviceName

DeviceRegistryEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| order by TimeGenerated desc
| project TimeGenerated, ActionType, RegistryKey, RegistryValueData, RegistryValueName

SecurityEvent
| where TenantId == "60c7f53e-249a-4077-b68e-55a4ae877d7c"
//| where DeviceId == "1d0e12b505d61c7eb1f1fd7842d905c99f6ae26a"

DeviceFileEvents
//| where InitiatingProcessAccountName == "5y51-d3p7"
//| where InitiatingProcessAccountDomain == "sys1-dept"
//| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
//| where InitiatingProcessUniqueId == "2533274790396713"
| where FileName contains "scorecard"
| order by TimeGenerated
| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessAccountName, InitiatingProcessRemoteSessionDeviceName, InitiatingProcessRemoteSessionIP, InitiatingProcessUniqueId
```
