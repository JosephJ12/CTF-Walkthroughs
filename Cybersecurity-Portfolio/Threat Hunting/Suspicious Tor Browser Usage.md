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

#### 4. Outgoing Network Connections via Tor
Our next step is to look for traces of outbound connections to known Tor ports. 
After googling for common Tor ports, we get the following: `9150, 9050, 9051, 9030, 9001, 443`

So we will look for all network events with the following criteria:
- host = edr-icedamerica
- time generated after Tor installer timestamp
- remote port in list of common Tor ports
- tor.exe or firefox.exe in the process command

Our KQL query comes out like:

```
DeviceNetworkEvents
| where DeviceName == "edr-icedamerica"
| where TimeGenerated >= todatetime('2026-02-02T22:36:56.0709422Z')
| where RemotePort in ('9050', '9150', '9051', '9001', '9030', '443')
| where InitiatingProcessCommandLine contains "tor.exe" or InitiatingProcessCommandLine contains "firefox.exe"
| order by TimeGenerated asc
| project TimeGenerated, ActionType, InitiatingProcessCommandLine, InitiatingProcessFileName, RemoteIP, RemoteUrl, RemotePort
```

<img width="2526" height="866" alt="image" src="https://github.com/user-attachments/assets/f4f0db94-face-4783-ae69-33278cda1437" />

