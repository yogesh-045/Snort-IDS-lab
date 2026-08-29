# Snort IDS - Configuration

This document describes the Snort configuration used in the Network Intrusion Detection System lab.

---

## 1. Configuration Overview

Snort 3 uses a Lua-based configuration file to define network settings, inspection modules, detection policies, and other IDS components.

The main configuration file used in this project is:

```text
/etc/snort/snort.lua
```
Custom detection rules are maintained separately in:
```text
/etc/snort/rules/local.rules
```

## 2. Lab Network

The project uses an isolated **Host-Only virtual network**.

| Component | Value |
|---|---|
| Network | `192.168.56.0/24` |
| Kali Linux | `192.168.56.102` |
| Metasploitable 2 | `192.168.56.101` |
| Snort Interface | `eth0` |

The isolated network allows security testing without targeting external systems.

## 3. Snort Configuration File

The primary configuration file is:
```text
/etc/snort/snort.lua
```
It contains configuration for multiple Snort modules, including:

- Network configuration
- Packet decoding
- Stream processing
- TCP inspection
- ICMP inspection
- Application identification
- Port scanning
- Detection engine
- Alert output
- IPS rules

## 4. HOME_NET Configuration

`HOME_NET` defines the network that Snort considers its protected or monitored network.

For this lab, the network is:
```text
192.168.56.0/24
```
The configuration should contain a network definition equivalent to:
```text
HOME_NET = '192.168.56.0/24'
```
This allows Snort rules using `$HOME_NET` to identify traffic directed toward the lab network.

For example, the TCP SYN detection rule uses:
```text
alert tcp any any -> $HOME_NET any (flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)
```

## 5. Network Interface

Snort monitors network traffic through the selected network interface.

The interface used in this lab is:
```text
eth0
```
The interface was verified using:
```text
ip addr
```
The relevant interface configuration was:
```text
eth0
inet 192.168.56.102/24
```
## 6. Custom Rules

Project-specific detection rules are stored in:
```text
/etc/snort/rules/local.rules
```
The rules used in this project include:

## ICMP Detection
```text
alert icmp 192.168.56.102 any -> 192.168.56.101 any (msg:"ICMP Ping Detected"; sid:1000001; rev:2;)
```
## TCP SYN Detection
alert tcp any any -> $HOME_NET any (flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)

## SSH Brute-Force Activity
```text
alert tcp any any -> 192.168.56.101 22 (flags:S; msg:"Possible SSH Brute Force Activity"; sid:1000003; rev:1; detection_filter:track by_src,count 5,seconds 60;)
```

## 7. Rule SID Management

Each custom rule uses a unique **Snort Signature ID (SID)**.

| SID | Rule | Purpose |
|---:|---|---|
| `1000001` | ICMP Ping Detected | Detect ICMP traffic |
| `1000002` | Possible TCP SYN Port Scan | Detect TCP SYN activity |
| `1000003` | Possible SSH Brute Force Activity | Detect repeated SSH connection attempts |

The `100000x` range is used for the custom rules created specifically for this lab.

## 8. Configuration Validation

Before running Snort, the configuration is validated using:
```text
sudo snort -T -c /etc/snort/snort.lua
```
A successful validation produces:
```text
Snort successfully validated the configuration (with 0 warnings).
```
This confirms that the configuration can be loaded without configuration errors.

## 9. Running Snort with Custom Rules

Snort can be started in live packet-monitoring mode using:

```bash
sudo snort -q -c /etc/snort/snort.lua -R /etc/snort/rules/local.rules -i eth0 -A alert_fast
```
### Command Breakdown

| Option | Description |
|---|---|
| `-q` | Quiet mode |
| `-c` | Specifies the Snort configuration file |
| `-R` | Loads the custom rules file |
| `-i eth0` | Monitors the `eth0` interface |
| `-A alert_fast` | Displays alerts in fast format |

## 10. Alert Output

During testing, Snort displays matching alerts in the terminal.

Example ICMP alert:
```text
[**] [1:1000001:2] "ICMP Ping Detected" [**]
{ICMP} 192.168.56.102 -> 192.168.56.101
```
Example TCP SYN alert:
```text
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
{TCP} 192.168.56.102:45709 -> 192.168.56.101:50636
```
Example SSH activity alert:
```text
[**] [1:1000003:1] "Possible SSH Brute Force Activity" [**]
{TCP} 192.168.56.102:xxxxx -> 192.168.56.101:22
```

## 11. SSH Detection Configuration

The SSH rule monitors TCP port 22 on the Metasploitable 2 target.
```text
192.168.56.101:22
```
The rule uses a detection filter:
```text
detection_filter:track by_src,count 5,seconds 60
```
This means Snort tracks the source IP and generates an alert when at least five matching SYN packets are observed from the same source within 60 seconds.

### Important Limitation

This rule detects repeated SSH connection attempts.

It does not directly inspect SSH passwords or determine whether an individual password was incorrect because SSH authentication traffic is encrypted.

For stronger confirmation of brute-force activity, Snort alerts should be correlated with SSH authentication logs on the target system.

## 12. Configuration Testing Workflow

The configuration workflow used in the project is:
```text
Edit Configuration
       |
       v
Configure HOME_NET
       |
       v
Add Custom Rules
       |
       v
Validate Configuration
       |
       v
Start Snort
       |
       v
Generate Test Traffic
       |
       v
Observe Alerts
       |
       v
Analyze Results
```
## 13. Configuration Validation Result

The Snort configuration was successfully validated with zero warnings.

This confirmed that:

- Snort configuration loaded successfully
- Required modules were loaded
- Custom detection rules could be used
- The eth0 interface could be monitored
- Snort was ready for traffic analysis

## 14. Security Considerations

This configuration was designed for an isolated cybersecurity laboratory.

The testing environment consists of:
```text
Kali Linux
192.168.56.102
       |
       |
192.168.56.101
Metasploitable 2
```
All scanning and detection activities should be performed only against systems that are owned by the tester or for which explicit authorization has been provided.

## 15. Summary

The Snort configuration provides the foundation for the IDS lab.

The configuration defines the monitored network, network interface, detection rules, and alert behavior.

Combined with the custom rules in `local.rules`, Snort can detect and report selected network activities including ICMP traffic, TCP SYN scanning, and repeated SSH connection attempts.
