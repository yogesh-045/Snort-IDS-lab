# Snort IDS - Setup & Configuration

This document describes the installation, configuration, and network setup of the Snort IDS lab.

---

## 1. Lab Environment

The project was implemented in an isolated virtual cybersecurity lab.

| Component | Configuration |
|---|---|
| IDS | Snort 3.12.2.0 |
| Security Testing OS | Kali Linux |
| Target OS | Metasploitable 2 |
| Kali IP | 192.168.56.102 |
| Metasploitable 2 IP | 192.168.56.101 |
| Network | 192.168.56.0/24 |
| Network Type | Host-Only |
| Monitoring Interface | eth0 |

---

## 2. Network Architecture

```text
              Host-Only Network
                192.168.56.0/24
                       |
          +------------+------------+
          |                         |
          v                         v
   Kali Linux               Metasploitable 2
 192.168.56.102              192.168.56.101
          |                         |
          |   Test Traffic          |
          +------------+------------+
                       |
                       v
                  Snort IDS
                       |
                       v
                     Alerts
```

## 3. Snort Installation

Snort was installed on Kali Linux using the APT package manager.
```text
sudo apt update
sudo apt install snort -y
```
The installed Snort version was verified using:
```text
snort -V
```
The lab used:
```text
Snort++ Version 3.12.2.0
```
## 4. Network Interface Verification

The network interface was checked using:
```text
ip addr
```
The monitoring interface was:
```text
eth0
```
Kali Linux:
```text
192.168.56.102/24
```
Metasploitable 2:
```text
192.168.56.101/24
```
## 5. Connectivity Testing

Connectivity between Kali Linux and Metasploitable 2 was verified using:
```text
ping -c 4 192.168.56.101
```
Successful ICMP communication confirmed that both virtual machines could communicate over the isolated network.

## 6. Snort Configuration

The main Snort configuration file used in the project was:
```text
/etc/snort/snort.lua
```
Snort configuration was validated using:
```text
sudo snort -T -c /etc/snort/snort.lua
```
Successful validation produced:
```text
Snort successfully validated the configuration (with 0 warnings).
```
## 7. Custom Rules

Custom detection rules were stored in:
```text
/etc/snort/rules/local.rules
```
The local rules file was used for project-specific detection signatures.

Rules were loaded during testing using:
```text
-R /etc/snort/rules/local.rules
```

## 8. Running Snort

Snort was started in live packet-monitoring mode using:
```text
sudo snort -q -c /etc/snort/snort.lua \
-R /etc/snort/rules/local.rules \
-i eth0 \
-A alert_fast
```

### Command Explanation

| Option | Purpose |
|---|---|
| `-q` | Quiet mode |
| `-c` | Specify Snort configuration |
| `-R` | Load additional rules |
| `-i` | Select network interface |
| `-A alert_fast` | Display alerts in fast format |

## 9. Configuration Result

The Snort configuration was successfully validated and the IDS was able to capture traffic from the eth0 interface.

The packet statistics confirmed that Snort was receiving and analyzing network traffic.

Example:
```text
received: 61
analyzed: 61
```
The captured traffic included:
```text
icmp4: 32
ipv4: 44
udp: 12
```
This confirmed that the IDS was actively processing network packets.

## 10. Security Note

All testing was performed inside an isolated virtual lab using Kali Linux and Metasploitable 2.

No unauthorized external systems were targeted.

The techniques documented in this project should only be performed against systems for which the tester has explicit authorization.
