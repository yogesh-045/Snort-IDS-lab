# ============================================================
# Snort IDS Lab - Custom Local Rules
# Author: Yogesh Mali
# Lab Network: 192.168.56.0/24
# Kali Linux: 192.168.56.102
# Metasploitable 2: 192.168.56.101
# ============================================================


# ------------------------------------------------------------
# SID 1000001 - ICMP Ping Detection
# Detect ICMP traffic from Kali to Metasploitable 2
# ------------------------------------------------------------

alert icmp 192.168.56.102 any -> 192.168.56.101 any (msg:"ICMP Ping Detected"; sid:1000001; rev:2;)


# ------------------------------------------------------------
# SID 1000002 - Possible TCP SYN Port Scan
# Detect TCP SYN packets directed toward the lab network
# ------------------------------------------------------------

alert tcp any any -> $HOME_NET any (flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)


# ------------------------------------------------------------
# SID 1000003 - Possible SSH Brute Force Activity
# Detect repeated SSH connection attempts to TCP port 22
# from the same source within 60 seconds
# ------------------------------------------------------------

alert tcp any any -> 192.168.56.101 22 (flags:S; msg:"Possible SSH Brute Force Activity"; sid:1000003; rev:1; detection_filter:track by_src,count 5,seconds 60;)


# ============================================================
# End of Custom Rules
# ============================================================
