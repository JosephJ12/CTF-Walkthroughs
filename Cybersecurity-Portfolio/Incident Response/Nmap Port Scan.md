# External Nmap Port Scan

## Skills and Technologies
- Nmap
- Microsoft Sentinel
- Kusto Query Language (KQL)
- Kali Linux
- Microsoft Azure VM

## Scenario
We detect suspicious activity from an external source, seeming to be a port scan of our Microsoft Azure system. We are tasked with investigating the incident, gathering evidence of the activities of the threat actor, and determining whether the scan resulted in further compromise of the system. 

## Part 1: Running the Nmap Scan from Kali Linux

We will run `nmap` from our local Kali Linux machine against the public IP of our Microsoft Azure VM. 

`nmap -sC -sV -p- -Pn --min-rate=1000 -T5 20.246.64.95`

<img width="550" height="207" alt="image" src="https://github.com/user-attachments/assets/2f5d685e-8315-4f1f-ab82-e15f256de49b" />

The scan results show that there is 1 port open, 3389 for RDP.

Now, let's head over to Sentinel and take a look at the logs.

## Part 2: Incident Response to Suspicious Port Scan Activity

