# Snort IDS Lab

![Snort](https://img.shields.io/badge/Snort-3.12.2.0-red)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-Lab-blue)
![Metasploitable 2](https://img.shields.io/badge/Metasploitable%202-Lab-orange)
![Nmap](https://img.shields.io/badge/Nmap-Network%20Scanning-green)
![Status](https://img.shields.io/badge/Project-Completed-success)

A practical Network Intrusion Detection System (NIDS) lab using **Snort 3**, **Kali Linux**, and **Metasploitable 2**.

This project demonstrates how Snort can monitor network traffic, use custom detection rules, and generate security alerts for suspicious network activity such as ICMP traffic and TCP SYN scanning.

---

## 📌 Project Overview

The objective of this project is to build and test a lightweight Network Intrusion Detection System using Snort in an isolated virtual lab environment.

The lab consists of:

- Kali Linux as the security testing machine
- Metasploitable 2 as the intentionally vulnerable target
- Snort 3 as the Network Intrusion Detection System
- Nmap for controlled network scanning
- Custom Snort rules for traffic detection and alert generation

The project focuses on understanding how network traffic is captured, inspected, matched against detection rules, and converted into security alerts.

---

## 🎯 Objectives

The main objectives of this project are:

- Install Snort 3 on Kali Linux
- Verify the Snort installation
- Configure Snort for network traffic monitoring
- Create custom Snort detection rules
- Detect ICMP traffic
- Detect TCP SYN scanning activity
- Monitor SSH connection attempts
- Generate and analyze Snort alerts
- Understand basic IDS workflow
- Document detection results in a practical lab environment

---

## 🏗️ Lab Architecture

```text
                         Isolated Host-Only Network
                              192.168.56.0/24
                                     |
             +-----------------------+-----------------------+
             |                                               |
             |                                               |
     Kali Linux                                      Metasploitable 2
   192.168.56.102                                    192.168.56.101
             |                                               |
             |                                               |
             +------------------- Traffic -------------------+
                                     |
                                     v
                                Snort IDS
                                     |
                                     v
                               Security Alerts
```

## Network Configuration

| Machine | IP Address | Role |
|---|---|---|
| Kali Linux | `192.168.56.102` | Security testing / traffic generation |
| Metasploitable 2 | `192.168.56.101` | Vulnerable target |
| Network | `192.168.56.0/24` | Isolated lab network |
| Snort Interface | `eth0` | Traffic monitoring |

> **Note:** The environment was designed as an isolated virtual lab using the host-only network to prevent testing traffic from reaching external systems.

# 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Snort 3 | Network Intrusion Detection |
| Kali Linux | Security testing platform |
| Metasploitable 2 | Vulnerable lab target |
| Nmap | Network and port scanning |
| Linux CLI | Configuration and administration |
| Custom Snort Rules | Traffic detection |

## 🔧 Snort Installation

Snort was installed using the Kali Linux package manager.
```text
sudo apt update
sudo apt install snort -y
```
The system already contained Snort 3, so the package manager reported that the latest version was installed.

### Verify Snort Version
```text
snort -V
```
Installed version:
```text
Snort++ Version 3.12.2.0
```
Additional components reported by Snort include:

- DAQ 3.0.24
- libpcap 1.10.6
- LuaJIT 2.1
- OpenSSL 3.6.3
- PCRE2 10.46
- ZLIB 1.3.2

## ⚙️ Snort Configuration

The primary Snort configuration file used in this lab is:
```text
/etc/snort/snort.lua
```
Configuration validation was performed using:
```text
sudo snort -T -c /etc/snort/snort.lua
```
Successful validation produced:
```text
Snort successfully validated the configuration (with 0 warnings).
```
This confirmed that the Snort configuration was syntactically valid and could be loaded successfully.

## 📜 Custom Rules

Custom detection rules were stored in:
```text
/etc/snort/rules/local.rules
```
The local rules file was used to define project-specific detection signatures.

## 1. ICMP Detection Rule

The first custom rule was created to detect ICMP traffic from the Kali test machine to the Metasploitable 2 machine.
```text
alert icmp 192.168.56.102 any -> 192.168.56.101 any (msg:"ICMP Ping Detected"; sid:1000001; rev:2;)
```
### Rule Explanation

| Component | Meaning |
|---|---|
| `alert` | Generate an alert |
| `icmp` | Match ICMP traffic |
| `192.168.56.102` | Kali Linux source |
| `192.168.56.101` | Metasploitable 2 destination |
| `msg` | Alert message |
| `sid:1000001` | Unique custom rule ID |
| `rev:2` | Rule revision |

Testing

The rule was tested using:
```text
ping -c 4 192.168.56.101
```
Snort successfully generated alerts such as:
```text
[**] [1:1000001:2] "ICMP Ping Detected" [**]
{ICMP} 192.168.56.102 -> 192.168.56.101
```
Result
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
Snort
      |
      v
ICMP Ping Detected
```

## 2. TCP SYN Port Scan Detection

A second custom rule was used to identify TCP SYN packets directed toward the target network.
```text
alert tcp any any -> $HOME_NET any (flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)
```

### Rule Explanation

| Component | Meaning |
|---|---|
| `alert` | Generate an alert |
| `tcp` | Match TCP traffic |
| `flags:S` | Match TCP SYN packets |
| `$HOME_NET` | Configured home network |
| `msg` | Alert message |
| `sid:1000002` | Unique custom rule ID |

> **Important:** This rule identifies SYN packets and therefore represents a possible TCP SYN scan indicator. A single SYN packet by itself does not prove that a port scan is occurring.

## 🔎 Nmap Port Scan

A controlled Nmap SYN scan was performed against the Metasploitable 2 lab machine.
```text
nmap -sS 192.168.56.101
```
The target was successfully identified as active.

The scan discovered multiple open TCP services, including:
```text
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3306/tcp  open  mysql
5432/tcp  open  postgresql
5900/tcp  open  vnc
8009/tcp  open  ajp13
8180/tcp  open  unknown
```
The complete scan identified 23 open TCP ports on the Metasploitable 2 system.

## 🚨 Port Scan Alert

During the controlled Nmap scan, Snort generated alerts matching the custom TCP SYN rule.

Example:
```text
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
[Priority: 0]
{TCP} 192.168.56.102:45709 -> 192.168.56.101:50636
```
Additional SYN packets targeting different ports were also detected.

Example:
```text
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
{TCP} 192.168.56.102:45709 -> 192.168.56.101:3005
```
### Detection Flow
```text
Nmap SYN Scan
      |
      v
Multiple TCP SYN packets
      |
      v
Snort Rule SID 1000002
      |
      v
Possible TCP SYN Port Scan
      |
      v
Security Alert
```

## 3. SSH Monitoring

Metasploitable 2 exposes an SSH service on:
```text
22/tcp
```
This was confirmed using:
```text
nmap -p 22 192.168.56.101
```
Expected result:
```text
22/tcp open ssh
```
SSH connection attempts can be monitored by Snort because the traffic is carried over TCP port 22.

### Important Detection Note

The current generic SYN detection rule can also generate an alert when a legitimate SSH connection begins because SSH uses TCP SYN packets to establish a connection.

Therefore:
```text
Possible TCP SYN Port Scan
```
should not be interpreted as a confirmed SSH brute-force attack.

For a production-quality implementation, SSH brute-force detection should use additional logic such as repeated connection/authentication attempts, thresholds, event correlation, or SSH authentication logs.

# 📊 Detection Results

| Detection Scenario | Test Method | Snort Result | Status |
|---|---|---|---|
| ICMP Traffic | `ping` | ICMP Ping Detected | ✅ Detected |
| TCP SYN Scan | `nmap -sS` | Possible TCP SYN Port Scan | ✅ Detected |
| SSH Service | `nmap -p 22` | SSH service identified | ✅ Verified |
| SSH Brute Force | Repeated SSH attempts | Requires dedicated threshold/correlation rule | 🔧 Enhancement |

## 📸 Evidence

Screenshots captured during the lab are stored in:

[screenshots](screenshots/)

Recommended screenshot structure:
```text
screenshots/
│
├── 01_snort_version.png
├── 02_snort_configuration_validation.png
├── 03_kali_ms2_connectivity.png
├── 04_icmp_detection_alert.png
├── 05_nmap_port_scan.png
├── 06_snort_port_scan_alert.png
└── 07_snort_ssh_bruteforce_alert.png
```
Evidence Description

### 01 — Snort Version

Shows the installed Snort 3.12.2.0 version.

### 02 — Configuration Validation

Shows successful Snort configuration validation with zero warnings.

### 03 — Network Connectivity

Demonstrates communication between Kali Linux and Metasploitable 2.

### 04 — ICMP Detection

Shows Snort generating the custom ICMP alert.

### 05 — Nmap Port Scan

Shows the controlled SYN scan against Metasploitable 2.

### 06 — Port Scan Detection

Shows Snort detecting TCP SYN traffic generated during the Nmap scan.

### 07 — SSH Monitoring

Reserved for the dedicated SSH brute-force detection implementation.

## 🧪 Testing Methodology

The following workflow was used:
```text
1. Configure isolated virtual machines
              ↓
2. Verify network connectivity
              ↓
3. Install and verify Snort
              ↓
4. Validate snort.lua configuration
              ↓
5. Create custom detection rules
              ↓
6. Start Snort in packet monitoring mode
              ↓
7. Generate controlled test traffic
              ↓
8. Observe Snort alerts
              ↓
9. Analyze alert information
              ↓
10. Document results
```

## 📈 Alert Analysis

A Snort alert contains useful information for security monitoring.

Example:
```text
[1:1000002:1] "Possible TCP SYN Port Scan"
{TCP} 192.168.56.102:45709 -> 192.168.56.101:50636
```

### Alert Components

| Field | Value | Meaning |
|---|---|---|
| Rule SID | `1000002` | Custom detection rule |
| Message | `Possible TCP SYN Port Scan` | Detection description |
| Protocol | TCP | Network protocol |
| Source IP | `192.168.56.102` | Kali Linux |
| Source Port | `45709` | Temporary source port |
| Destination IP | `192.168.56.101` | Metasploitable 2 |
| Destination Port | `50636` | Target port |

This information can help a security analyst identify:

- Source system
- Destination system
- Network protocol
- Destination port
- Detection signature
- Nature of the observed traffic

## 🔐 Security Considerations

This project was performed in an isolated virtual lab environment.

The target system used in the project is Metasploitable 2, which is intentionally vulnerable and designed for security testing and education.

All scanning and testing activities were limited to the lab environment:
```text
192.168.56.0/24
```
The project should not be used to scan or attack systems without authorization.

## 🧠 Key Learning Outcomes

Through this project, the following concepts were practiced:

- Network Intrusion Detection Systems
- Snort 3 installation and configuration
- Packet capture
- Network interface monitoring
- Snort custom rules
- Rule SID management
- ICMP traffic detection
- TCP SYN analysis
- Port scanning detection
- Nmap scanning
- SSH service monitoring
- Security alert interpretation
- Basic network security analysis
- Linux command-line administration

## 🚀 Future Improvements

The project can be extended with more advanced detection capabilities.

### Planned Enhancements
- Threshold-based SSH brute-force detection
- SSH authentication failure correlation
- UDP scan detection
- ICMP flood detection
- DNS anomaly detection
- HTTP attack detection
- Better port-scan correlation
- Alert logging
- Automated alert reporting
- Integration with a SIEM
- Grafana/ELK-based visualization
- Automated security dashboard

## 📁 Repository Structure
```text
snort-ids-lab/
│
├── README.md
│   
│
├── docs/
│   ├── setup.md
│   ├── configuration.md
│   ├── detection.md
│   └── results.md
│
├── lab/
│   └── network-topology.png
│
├── rules/
│   └── local.rules
│
├── screenshots/
│   ├── 01_snort_version.png
│   ├── 02_snort_configuration_validation.png
│   ├── 03_kali_ms2_connectivity.png
│   ├── 04_icmp_detection_alert.png
│   ├── 05_nmap_port_scan.png
│   ├── 06_snort_port_scan_alert.png
│   └── 07_snort_ssh_bruteforce_alert.png
│
└── LICENSE
```
## 💼 Skills Demonstrated

This project demonstrates practical experience with:

![Network Security](https://img.shields.io/badge/Network%20Security-Cybersecurity-blue)
![Snort 3](https://img.shields.io/badge/Snort%203-IDS-red)
![Custom IDS Rules](https://img.shields.io/badge/Custom%20IDS%20Rules-Snort-orange)
![Nmap](https://img.shields.io/badge/Nmap-Network%20Scanning-green)
![TCP/IP](https://img.shields.io/badge/TCP%2FIP-Networking-blueviolet)
![Port Scan Detection](https://img.shields.io/badge/Port%20Scan%20Detection-Detection-critical)
![SSH Monitoring](https://img.shields.io/badge/SSH%20Monitoring-Security-blue)
![Linux](https://img.shields.io/badge/Linux-Administration-black)
## 🏁 Conclusion

This project demonstrates the practical implementation of a Network Intrusion Detection System using Snort 3 in an isolated cybersecurity laboratory.

Snort was successfully installed and configured on Kali Linux, custom detection rules were created, and controlled network traffic was generated against Metasploitable 2.

The IDS successfully detected ICMP traffic and TCP SYN scanning activity and generated corresponding security alerts.

The project provides a foundation for implementing more advanced security monitoring techniques such as threshold-based brute-force detection, event correlation, centralized logging, and SIEM integration.

## 📚 References
- Snort Documentation
- Nmap Documentation
- Kali Linux Documentation
- Metasploitable 2 Documentation

### 👨‍💻 Author

Yogesh Mali

Cybersecurity Learner | SOC / Blue Team Enthusiast

⭐ If you found this project useful, consider giving the repository a star.
