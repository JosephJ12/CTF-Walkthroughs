# Suspicious Tor Browser Usage

## Skills and Technologies
- Microsoft Sentinel
- Microsoft Azure VM
- Kusto Query Language (KQL)
- Microsoft Defender for Endpoint (MDE)

## Background

We come across an unauthorized Tor browser usage from the internal network. We are tasked with investigating the incident. To begin the hunt, we are given the device name and general date of the activity logged:
- `edr-icedamerica`
- Feb 2nd, 2026

## Threat Hunting Process

#### 1. Investigating File Events
To begin the hunt, we want to find evidence that confirms Tor usage and gives us a generic idea of what happened. We look for evidence of the Tor browser being installed in the file events table:

```
DeviceFileEvents
| where DeviceName == "edr-icedamerica"
| where FileName contains "tor" and FileName endswith ".exe"
| where TimeGenerated >= datetime(2026-02-02)
| order by TimeGenerated desc
| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessCommandLine, InitiatingProcessFileName
```

<img width="2526" height="952" alt="image" src="https://github.com/user-attachments/assets/d8f26ec7-0d1b-435f-8b05-11b549742edd" />

Indeed, we find traces of a Tor installer.

#### 2. Deeper Dive into File Events
Now that we know the time stamp for when Tor was installed, we look for all instances of `tor` in the filename after the specified time.

```
DeviceFileEvents
| where DeviceName == "edr-icedamerica"
| where FileName contains "tor"
| where TimeGenerated >= todatetime('2026-02-02T22:36:56.0709422Z')
| order by TimeGenerated asc
| project TimeGenerated, ActionType, FileName, FolderPath, InitiatingProcessCommandLine, InitiatingProcessFileName
```

<img width="2516" height="794" alt="image" src="https://github.com/user-attachments/assets/e5f6281e-e5f7-4154-93cb-14157c73369d" />

#### 3. Investigating Tor Usage
Though from the previous query results, we can assume Tor usage, we'll confirm this by looking through the process logs

```
DeviceProcessEvents
| where DeviceName == "edr-icedamerica"
| where TimeGenerated >= todatetime('2026-02-02T22:36:56.0709422Z')
| where ProcessCommandLine contains "tor.exe" or ProcessCommandLine contains "firefox.exe"
| order by TimeGenerated asc
| project TimeGenerated, InitiatingProcessVersionInfoFileDescription, ProcessCommandLine, InitiatingProcessCommandLine, FileName
```

<img width="2526" height="964" alt="image" src="https://github.com/user-attachments/assets/bd5d0ef1-1811-4100-a68f-4744cd453aae" />

We confirm usage of Tor

#### 4. 
