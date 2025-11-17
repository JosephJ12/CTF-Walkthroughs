# Attacking Enterprise Networks

**Platform: Hack the Box Academy**

**OS: Windows**


## **Disclaimer: Potential spoilers below**

## Scope
External Testing	                          Internal Testing

10.129.x.x ("external" facing target host)	172.16.8.0/23

*.inlanefreight.local (all subdomains)	    172.16.9.0/23
                                            
                                            INLANEFREIGHT.LOCAL (Active Directory domain)

<img width="1504" height="660" alt="image" src="https://github.com/user-attachments/assets/fb7f6597-3abf-4495-8074-4e4f73d8cae8" />

## Walkthrough

1. Do a ping sweep of external IP range with Nmap

```
nmap -sn 10.129.0.0/16 -oN alive_ips.txt --min-rate 3000 -T5

# Nmap 7.94SVN scan initiated Sun Nov 16 22:29:25 2025 as: nmap -sn -oN alive_ips.txt --min-rate 3000 -T5 10.129.0.0/16
Nmap scan report for 10.129.0.1
Nmap scan report for 10.129.2.141
Nmap scan report for 10.129.2.219
Nmap scan report for 10.129.15.95
Nmap scan report for 10.129.35.28
Nmap scan report for 10.129.42.254
Nmap scan report for 10.129.43.4
Nmap scan report for 10.129.48.182
Nmap scan report for 10.129.59.248
Nmap scan report for 10.129.120.171
Nmap scan report for 10.129.124.236
Nmap scan report for 10.129.126.149
Nmap scan report for 10.129.127.86
Nmap scan report for 10.129.173.143
Nmap scan report for 10.129.191.157
Nmap scan report for 10.129.203.22
Nmap scan report for 10.129.204.23
Nmap scan report for 10.129.234.170
Nmap scan report for 10.129.252.88
# Nmap done at Sun Nov 16 22:32:26 2025 -- 65536 IP addresses (19 hosts up) scanned in 181.10 seconds
```

2. Format all alive hosts to only get the IP address from output

```
cat alive_ips.txt | grep Nmap | cut -d ' ' -f5 > alive_ips_formatted.txt

cat alive_ips_formatted.txt
10.129.0.1
10.129.2.141
10.129.2.219
10.129.15.95
10.129.35.28
10.129.42.254
10.129.43.4
10.129.48.182
10.129.59.248
10.129.120.171
10.129.124.236
10.129.126.149
10.129.127.86
10.129.173.143
10.129.191.157
10.129.203.22
10.129.204.23
10.129.234.170
10.129.252.88
```

3. Do a full port scan of all alive hosts

```
nmap -sV -Pn -p- -T5 --min-rate 3000 -oN initial_scan -iL alive_ips_formatted.txt

```
