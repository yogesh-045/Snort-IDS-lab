
# Snort IDS - Detection Scenarios

This document describes the detection scenarios performed using Snort custom rules.

The project focuses on:

1. ICMP traffic detection
2. TCP SYN port-scan detection
3. SSH service monitoring

---

# 1. ICMP Detection

## Objective

Detect ICMP traffic generated between Kali Linux and Metasploitable 2.

---

## Custom Rule

```text
alert icmp 192.168.56.102 any -> 192.168.56.101 any (msg:"ICMP Ping Detected"; sid:1000001; rev:2;)
```

### Rule Breakdown

| Component | Description |
|---|---|
| `alert` | Generate an alert |
| `icmp` | Match ICMP traffic |
| `192.168.56.102` | Kali source IP |
| `192.168.56.101` | Metasploitable 2 destination |
| `msg` | Alert message |
| `sid:1000001` | Unique rule identifier |
| `rev:2` | Rule revision |

### Test

The rule was tested using:
```text
ping -c 4 192.168.56.101
```
### Alert

Snort generated:
```text
[**] [1:1000001:2] "ICMP Ping Detected" [**]
{ICMP} 192.168.56.102 -> 192.168.56.101
```
### Detection Flow
```text
Kali Linux
192.168.56.102
      |
      | ICMP Echo Request
      v
Metasploitable 2
192.168.56.101
      |
      v
Snort IDS
      |
      v
ICMP Ping Detected
```

# 2. TCP SYN Port Scan Detection
### Objective

Detect TCP SYN packets generated during a controlled port scan.

###Custom Rule
```text
alert tcp any any -> $HOME_NET any (flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)
```
## Rule Breakdown

| Component | Description |
|---|---|
| `alert` | Generate an alert |
| `tcp` | Match TCP traffic |
| `flags:S` | Match TCP SYN packets |
| `$HOME_NET` | Configured home network |
| `msg` | Alert message |
| `sid:1000002` | Unique rule identifier |

## Test

A TCP SYN scan was performed using Nmap:
```text
nmap -sS 192.168.56.101
```
The scan identified multiple open services on Metasploitable 2.

Examples included:
```text
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3306/tcp  open  mysql
5432/tcp  open  postgresql
5900/tcp  open  vnc
```
## Snort Alert

Snort successfully detected TCP SYN traffic.

Example:
```text
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
[Priority: 0]
{TCP} 192.168.56.102:45709 -> 192.168.56.101:50636
```
Another detected connection:
```text
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
{TCP} 192.168.56.102:45709 -> 192.168.56.101:3005
```

## Detection Flow
```text
Nmap SYN Scan
      |
      v
TCP SYN Packets
      |
      v
Snort Custom Rule
SID: 1000002
      |
      v
Possible TCP SYN Port Scan
      |
      v
Security Alert
```
###Important Note

The custom rule detects TCP SYN packets.

A single TCP SYN packet does not prove that a port scan is occurring.

Therefore, the alert message uses:
```text
Possible TCP SYN Port Scan
```
rather than claiming that every detected SYN packet is a confirmed port scan.

For a production IDS implementation, thresholding and event correlation can be added to detect repeated scanning behavior more accurately.

# 3. SSH Brute-Force Detection

## Objective

Detect repeated SSH connection attempts against the Metasploitable 2 SSH service.

SSH operates on TCP port 22.

Target:

```text
192.168.56.101:22
```
Source:
```text
192.168.56.102
```
### Custom Rule
```text
alert tcp any any -> 192.168.56.101 22 (flags:S; msg:"Possible SSH Brute Force Activity"; sid:1000003; rev:1; detection_filter:track by_src,count 5,seconds 60;)
```
### Rule Logic

The rule tracks TCP SYN packets directed toward SSH port 22.

An alert is generated when the same source produces at least five connection attempts within 60 seconds.

| Parameter | Meaning |
|---|---|
| `tcp` | TCP traffic |
| `192.168.56.101` | Metasploitable 2 |
| `22` | SSH port |
| `flags:S` | TCP SYN |
| `track by_src` | Track attempts by source IP |
| `count 5` | Five attempts |
| `seconds 60` | Within 60 seconds |
| `sid:1000003` | SSH detection rule |

## Testing

Repeated SSH authentication attempts were generated from Kali Linux against the isolated Metasploitable 2 lab target.

The testing was performed only inside the authorized virtual laboratory.

Expected Alert
```text
[**] [1:1000003:1] "Possible SSH Brute Force Activity" [**]
{TCP} 192.168.56.102:xxxxx -> 192.168.56.101:22
```
## Analysis

The Snort alert indicates repeated SSH connection attempts from the same source.

Because SSH authentication is encrypted, the Snort packet rule does not directly identify whether an individual password was incorrect.

Therefore, SSH authentication logs should be correlated with the Snort alert to confirm repeated failed login attempts.

## Detection Flow
```text
Repeated SSH Attempts
        |
        v
TCP SYN packets to port 22
        |
        v
Snort SID 1000003
        |
        v
Possible SSH Brute Force Activity
        |
        v
Correlate with SSH authentication logs
        |
        v
Security Investigation
```
# 4. Alert Analysis

Snort alerts provide useful information to a security analyst.

Example:
```text
[1:1000002:1] "Possible TCP SYN Port Scan"
{TCP} 192.168.56.102:45709 -> 192.168.56.101:50636
```
Important fields include:

| Field | Example | Meaning |
|---|---|---|
| SID | `1000002` | Detection rule |
| Message | `Possible TCP SYN Port Scan` | Alert description |
| Protocol | `TCP` | Network protocol |
| Source IP | `192.168.56.102` | Kali Linux |
| Source Port | `45709` | Temporary source port |
| Destination IP | `192.168.56.101` | Metasploitable 2 |
| Destination Port | `50636` | Target port |

This information helps identify the source, destination, protocol, and nature of suspicious network activity.
