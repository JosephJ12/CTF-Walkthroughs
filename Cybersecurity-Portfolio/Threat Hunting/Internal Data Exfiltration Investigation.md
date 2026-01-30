# Internal Data Exfiltration

## Skills and Technologies
- Microsoft Azure
- Microsoft Defender for Endpoint (MDE)
- Powershell
- Kusto Query Language (KQL)

## Scenario
An employee that has local administrator privileges on a Windows machine is suspected of compressing company data and exfiltrating it to an external computer. We are tasked with investigating this issue and gathering evidence via logs if this suspicious activity is discovered to be taking place. 

## Threat Hunting Steps

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

## MITRE ATT&CK TTP

| ATT&CK Tactic               | Technique ID | Technique Name                                | Description (Aligned to Findings)                                                                                                 | Reference Link                                                                                   |
| --------------------------- | ------------ | --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Execution                   | T1059.001    | Command and Scripting Interpreter: PowerShell | PowerShell was used to execute a script (`exfiltratedata.ps1`) that downloaded tooling and performed file compression activities. | [https://attack.mitre.org/techniques/T1059/001/](https://attack.mitre.org/techniques/T1059/001/) |
| Command and Control         | T1105        | Ingress Tool Transfer                         | The PowerShell script downloaded and installed the external archiving utility 7zip, which is not native to the Windows OS.        | [https://attack.mitre.org/techniques/T1105/](https://attack.mitre.org/techniques/T1105/)         |
| Collection                  | T1560.001    | Archive Collected Data: Archive via Utility   | Sensitive employee data was compressed into a ZIP archive using 7zip, indicating preparation for potential exfiltration.          | [https://attack.mitre.org/techniques/T1560/001/](https://attack.mitre.org/techniques/T1560/001/) |
| Collection                  | T1074.001    | Data Staged: Local Data Staging               | The compressed archive was staged locally on the endpoint without immediate outbound transfer.                                    | [https://attack.mitre.org/techniques/T1074/001/](https://attack.mitre.org/techniques/T1074/001/) |


## Summary of Findings

After investigating MDE logs, we found traces of a suspicious poweshell script, `exfiltratedata.ps1`, that downloads the popular archiving tool, `7zip`, and uses it to archive the employee data spreadsheet on the `stefano-test` machine. However, we were not able to gather any evidence for this file being exfiltrated on the network. 
