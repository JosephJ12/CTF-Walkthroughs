# Crosscheck

### Technologies and Skills


### Scenario

At the onset of December, routine monitoring detects irregular access patterns during year-end compensation and performance review activities. 

What initially appears as legitimate administrative and departmental behavior reveals a multi-stage sequence involving unauthorized script execution, sensitive file access, data staging, persistence mechanisms, and outbound communication attempts. 

We are tasked with correlating endpoint telemetry across multiple user contexts and systems to reconstruct the full access chain and determine how year-end bonus and performance data was accessed, prepared, and transmitted.

### Threat Hunt Process

##### 1. Initial Endpoint Discovery

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
