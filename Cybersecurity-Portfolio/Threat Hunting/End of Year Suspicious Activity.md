# End of Year Suspicious Activity

## Technologies and Skills


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

Looking further at the results of the previous query, we find evidence of the suspicious user accessing the `BonusMatrix_Draft_v3.xlsx` Excel file. The next thing to look for is if there are traces of data being exfiltrated.

```
DeviceFileEvents
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessAccountDomain == "sys1-dept"
| where InitiatingProcessRemoteSessionIP == "192.168.0.110"
| order by TimeGenerated
| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessRemoteSessionDeviceName, InitiatingProcessRemoteSessionIP, InitiatingProcessUniqueId
```

<img width="2578" height="672" alt="image" src="https://github.com/user-attachments/assets/2543e115-d585-48f4-9440-0fa434288948" />


```
kql
DeviceFileEvents
| where InitiatingProcessAccountName == "5y51-d3p7"
//| where InitiatingProcessCommandLine != "\"powershell.exe\" -ExecutionPolicy Bypass -File C:\\Users\\5y51-D3p7\\Downloads\\PayrollSupportTool.ps1"
| where InitiatingProcessRemoteSessionDeviceName == "YE-HELPDESKTECH"
| order by TimeGenerated
//| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessCommandLine

DeviceNetworkEvents
| where DeviceName == "sys1-dept"
| where InitiatingProcessAccountName == "5y51-d3p7"
| where InitiatingProcessRemoteSessionDeviceName == "YE-HELPDESKTECH"
| order by TimeGenerated desc
//| project TimeGenerated, ActionType, RemoteIP, RemotePort, RemoteUrl

DeviceProcessEvents
| where DeviceName == "sys1-dept"
| where AccountName == "5y51-d3p7"
//| where InitiatingProcessRemoteSessionDeviceName == "YE-HELPDESKTECH"
| project TimeGenerated, FileName, FolderPath, ProcessCommandLine, ProcessRemoteSessionDeviceName
| order by TimeGenerated desc
```
