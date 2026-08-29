# Snort IDS - Results & Analysis

This document summarizes the results obtained during the Snort IDS laboratory.

---

# 1. Test Summary

The following detection scenarios were tested:

| Test | Tool / Method | Result |
|---|---|---|
| Snort Installation | APT | Successful |
| Configuration Validation | Snort `-T` | Successful |
| Network Connectivity | Ping | Successful |
| ICMP Detection | Ping | Detected |
| TCP SYN Scan | Nmap | Detected |
| SSH Service | Nmap | Verified |
| SSH Brute Force | Dedicated correlation rule | Future Enhancement |

---

# 2. Snort Installation Result

Snort was successfully installed on Kali Linux.

Version:

```text
Snort++ 3.12.2.0
```

# 3. Configuration Result

Snort configuration validation was successful.

Command:
```text
sudo snort -T -c /etc/snort/snort.lua
```
Result:
```text
Snort successfully validated the configuration (with 0 warnings).
```
This confirmed that the Snort configuration could be loaded successfully.

# 4. Packet Capture Result

Snort successfully captured and analyzed traffic on the eth0 interface.

Example packet statistics:
```text
received: 61
analyzed: 61
```
Protocol statistics included:
```text
icmp4: 32
ipv4: 44
udp: 12
```
This demonstrated that Snort was actively processing network traffic.

# 5. ICMP Detection Result

A custom ICMP detection rule was created:
```text
alert icmp 192.168.56.102 any -> 192.168.56.101 any (msg:"ICMP Ping Detected"; sid:1000001; rev:2;)
```
The rule was tested using:
```text
ping -c 4 192.168.56.101
```
Snort generated:
```text
[**] [1:1000001:2] "ICMP Ping Detected" [**]
{ICMP} 192.168.56.102 -> 192.168.56.101
```
### Result

**ICMP traffic was successfully detected.**

# 6. Port Scan Detection Result

A controlled TCP SYN scan was performed using:
```text
nmap -sS 192.168.56.101
```
The scan identified multiple open TCP services on Metasploitable 2.

The custom Snort rule:
```text
alert tcp any any -> $HOME_NET any (flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)
```
generated alerts.

Example:
```
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
[Priority: 0]
{TCP} 192.168.56.102:45709 -> 192.168.56.101:50636
```
### Result

***TCP SYN scanning activity was successfully detected.***

# 7. SSH Monitoring Result

The Metasploitable 2 SSH service was verified:
```text
22/tcp open ssh
```
using:
```text
nmap -p 22 192.168.56.101
```
TCP SYN traffic targeting port 22 was also observed by Snort.

Example:
```text
[**] [1:1000002:1] "Possible TCP SYN Port Scan" [**]
{TCP} 192.168.56.102:38972 -> 192.168.56.101:22
```
This confirms that Snort can observe traffic targeting the SSH service.

However, a generic SYN alert is not sufficient to prove an SSH brute-force attack.

8. Overall Results
 ```text
                    Snort IDS
                       |
          +------------+------------+
          |            |            |
          v            v            v
        ICMP       TCP SYN        SSH
       Detection     Scan        Monitoring
          |            |            |
          v            v            v
       ALERT        ALERT       TRAFFIC
       Generated    Generated   Observed
```
The installation was verified using:
```text
snort -V
```

# 9. Detection Summary

| Detection | Rule SID | Status |
|---|---:|---|
| ICMP Ping Detection | `1000001` | ✅ Successful |
| TCP SYN Scan Detection | `1000002` | ✅ Successful |
| SSH Service Monitoring | `1000002` | ✅ Observed |
| Dedicated SSH Brute Force Detection | `1000003` | ✅ Successful |

# 10. Security Analysis

The lab demonstrates how an IDS can provide visibility into network activity.

The alerts provide:

- Source IP address
- Destination IP address
- Source port
- Destination port
- Protocol
- Detection rule
- Alert message

A security analyst can use this information to investigate suspicious activity and determine whether additional investigation or response is required.

# 11. Limitations

The current implementation uses simple custom signatures.

The TCP SYN rule can generate alerts for individual SYN packets, which means it can produce false positives during legitimate TCP connection establishment.

Similarly, detecting SSH brute force reliably requires more than simply detecting TCP connections to port 22.

A production-ready IDS should use:

- Thresholds
- Event correlation
- Authentication logs
- Multiple indicators
- Baseline traffic analysis
- SIEM integration

# 12. Conclusion

The Snort IDS lab successfully demonstrated the basic workflow of a Network Intrusion Detection System.

Snort 3.12.2.0 was installed and configured on Kali Linux and successfully captured network traffic from the isolated lab environment.

Custom rules were created to detect ICMP traffic and TCP SYN scanning activity.

Controlled testing using Ping and Nmap generated corresponding Snort alerts.

The project also demonstrated the importance of understanding alert context. A TCP SYN alert targeting SSH port 22 does not automatically represent brute-force activity, highlighting the importance of correlation and threshold-based detection in real-world SOC environments.

Overall, the project provides a practical foundation for learning:

- Network monitoring
- IDS configuration
- Custom detection rules
- Alert analysis
- Network reconnaissance detection
- Security event investigation
