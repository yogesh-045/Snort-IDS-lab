# ============================================================
# Snort IDS Lab - Custom Local Rules
# Author: Yogesh Mali
# Lab Network: 192.168.56.0/24
# Kali Linux: 192.168.56.102
# Metasploitable 2: 192.168.56.101
# ============================================================


# ------------------------------------------------------------
# Rule 1: ICMP Ping Detection
# Detect ICMP traffic from Kali Linux to Metasploitable 2
# ------------------------------------------------------------

alert icmp 192.168.56.102 any -> 192.168.56.101 any \
(msg:"ICMP Ping Detected"; sid:1000001; rev:2;)


# ------------------------------------------------------------
# Rule 2: Possible TCP SYN Port Scan
# Detect TCP SYN packets directed toward the home network
# ------------------------------------------------------------

alert tcp any any -> $HOME_NET any \
(flags:S; msg:"Possible TCP SYN Port Scan"; sid:1000002; rev:1;)


# ============================================================
# Future Rule
# SSH Brute-Force Detection
#
# A dedicated threshold/correlation rule will be added after
# completing and validating the SSH brute-force test.
# ============================================================
