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

We first check the MDE logs for any file activity that contains the word .zip

```
DeviceFileEvents
| where FileName endswith "zip" and FileName != "VMAgentLogs.zip"
| order by Timestamp desc
```

<img width="2503" height="761" alt="image" src="https://github.com/user-attachments/assets/19127193-3f70-4cba-8071-507057708064" />

#### 2. 
