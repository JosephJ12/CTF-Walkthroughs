# Internal Data Exfiltration

### Skills and Technologies
- Microsoft Azure
- Microsoft Defender for Endpoint (MDE)
- Powershell
- Kusto Query Language (KQL)

### Scenario
An employee that has local administrator privileges on a Windows machine is suspected of compressing company data and exfiltrating it to an external computer. We are tasked with investigating this issue and gathering evidence via logs if this suspicious activity is discovered to be taking place. 

### Threat Hunting Steps

#### 1. Check logs for zip file creation

We first check the MDE logs for any file activity that contains the word .zip. Our main goal right now is to gain general information, things such as a relative time frame and which device (if any) has suspicious activity.

```
DeviceFileEvents
| where FileName endswith "zip" and FileName != "VMAgentLogs.zip"
| order by Timestamp desc
```

<img width="2503" height="761" alt="image" src="https://github.com/user-attachments/assets/c46e4a70-e7f5-4840-80e1-6362f5db938c" />

#### 2. Deeper investigation of process events during this time frame

Now that we have a general time frame and device name to go off of, we will further look into any processes and events that happened using the following KQL query:

```
let IncidentTime = datetime(2026-01-29T16:49:20.4813094Z);
DeviceProcessEvents
| where DeviceName == "stefano-test"
| where Timestamp between ((IncidentTime - 2m) .. (IncidentTime + 2m))
| order by Timestamp desc
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, ProcessVersionInfoOriginalFileName, ProcessCommandLine
```

<img width="2501" height="1060" alt="image" src="https://github.com/user-attachments/assets/dd027fd6-b007-4e67-a23e-4e6a39238418" />

#### 3. Look for evidence of outbound connection sending data

So far, we have evidence of a Powershell script that installs 7zip and compresses employee data into a zip file. Now, we will check the logs to see if this file was sent outside our network.

```
let IncidentTime = datetime(2026-01-29T16:49:20.4813094Z);
DeviceNetworkEvents
| where DeviceName == "stefano-test"
| where Timestamp between ((IncidentTime - 2m) .. (IncidentTime + 2m))
| where AdditionalFields contains "Out"
| order by Timestamp desc
```
<img width="2504" height="1203" alt="image" src="https://github.com/user-attachments/assets/26b8c172-d52e-4cf8-a1af-0e5dbb10adb7" />

Fortunately, doesn't seem like we have any logs showing that a file was sent outside

#### 4. 
