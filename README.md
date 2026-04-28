# 🛡️ Wazuh SIEM Home Lab — Attack Detection & Monitoring

## Overview

This project documents the design, deployment, and testing of a fully functional Security Information and Event Management (SIEM) environment using **Wazuh** and the **ELK Stack**. The lab simulates real-world attack scenarios and demonstrates how a SOC analyst detects and investigates threats in real time.

This project was built entirely from scratch as part of my cybersecurity portfolio, combining both offensive and defensive security skills.

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Host Machine                       │
│              Windows 11 | 32GB RAM                   │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │ Wazuh Server │  │Windows Victim│  │   Kali    │  │
│  │   Ubuntu     │◄─│   Agent      │  │   Linux   │  │
│  │  20.04 LTS   │  │ Windows 10   │  │ Attacker  │  │
│  │  10.0.0.3    │  │  10.0.0.4    │  │ 10.0.0.6  │  │
│  └──────────────┘  └──────────────┘  └───────────┘  │
│                                                      │
│              NAT Network: 10.0.0.0/24                │
└─────────────────────────────────────────────────────┘
```

### Virtual Machines

| VM | OS | IP | Role |
|---|---|---|---|
| Wazuh Server | Ubuntu 20.04 LTS | 10.0.0.3 | SIEM — log collection, alerting, dashboard |
| Windows Victim | Windows 10 | 10.0.0.4 | Monitored endpoint with Wazuh agent |
| Kali Linux | Kali Rolling | 10.0.0.6 | Attacker machine |

---

## 🛠️ Tools & Technologies

- **Wazuh 4.7.5** — Open source SIEM and XDR platform
- **Elastic Stack** — Log indexing and visualization
- **VirtualBox 7.x** — Virtualization platform
- **Kali Linux** — Offensive security tools (Nmap, Hydra, Metasploit)
- **MITRE ATT&CK Framework** — Attack classification and mapping

---

## 📋 Project Phases

### Phase 1 — Lab Setup
- Deployed 3 VMs on an isolated NAT network using VirtualBox
- Configured network adapters for inter-VM communication
- Set up SSH access from host to Wazuh server for remote management

### Phase 2 — Wazuh Deployment
- Installed Wazuh all-in-one stack (Manager + Indexer + Dashboard) on Ubuntu Server
- Configured Wazuh dashboard accessible via browser on host machine
- Verified all services running: wazuh-manager, wazuh-indexer, wazuh-dashboard

### Phase 3 — Agent Deployment
- Installed Wazuh agent on Windows 10 VM
- Connected agent to Wazuh manager at 10.0.0.3
- Verified agent status: Active — real-time log forwarding confirmed

### Phase 4 — Attack Simulation & Detection
Simulated real-world attack scenarios from Kali Linux while monitoring alerts in Wazuh dashboard.

### Phase 5 — Documentation & Analysis
- Analyzed generated alerts and mapped to MITRE ATT&CK framework
- Documented findings in this report

---

## ⚔️ Attack Simulations & Detections

### Attack 1 — Network Reconnaissance (Nmap)

**Tool:** Nmap 7.95  
**Command:**
```bash
nmap -sV 10.0.0.4
nmap --script vuln 10.0.0.4
```

**What it does:** Scans target for open ports, running services, and known vulnerabilities.

**Results:**
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH
5357/tcp open  http    Microsoft HTTPAPI 2.0
OS: Windows detected
```

**MITRE Technique:** T1595 — Active Scanning  
**Detection:** Network connection logs captured by Wazuh agent

---

### Attack 2 — SSH Brute Force Attack

**Tool:** Metasploit Framework (auxiliary/scanner/ssh/ssh_login)  
**Target:** Administrator account on Windows-Victim (10.0.0.4)  
**Wordlist:** rockyou.txt (14,344,399 passwords)

**Wazuh Alerts Generated:**
| Rule ID | Description | Severity |
|---|---|---|
| 60122 | Logon failure - Unknown user or bad password | Level 5 |
| 60106 | Windows logon success | Level 3 |
| 60137 | Windows User Logoff | Level 3 |

**MITRE Techniques Mapped:**
- T1078 — Valid Accounts
- T1531 — Account Access Removal

**Key Finding:** Wazuh detected 51 authentication failures in rapid succession — a clear brute force pattern that would trigger immediate SOC investigation.

---

### Attack 3 — Malware Detection (Mimikatz)

**Tool:** Mimikatz 2.2.0 (credential dumping tool)  
**Objective:** Attempt to dump credentials from Windows memory (LSASS)

**Result:** Windows Defender detected and blocked Mimikatz before execution, generating security alerts captured by Wazuh.

**MITRE Technique:** T1003 — OS Credential Dumping  
**Detection:** Antivirus/EDR alert logged and forwarded to Wazuh

**Key Finding:** Defense-in-depth working as expected — endpoint protection (Defender) + SIEM monitoring (Wazuh) provide layered detection.

---

## 📊 Dashboard Results

### Security Events Summary (24 hours)

| Metric | Count |
|---|---|
| Total Alerts | 799 |
| Authentication Failures | 51 |
| Authentication Successes | 69 |
| Level 12+ Critical Alerts | 0 |

### Top MITRE ATT&CK Tactics Detected

| Tactic | Count |
|---|---|
| Persistence | 160 |
| Defense Evasion | 158 |
| Privilege Escalation | 158 |
| Initial Access | 156 |
| Impact | 51 |

### Security Configuration Assessment (SCA)

**Benchmark:** CIS Microsoft Windows 10 Enterprise v1.12.0  
**Score: 32%** — 126 passed, 264 failed checks

This low score reflects a default Windows 10 installation with no hardening applied — intentional for lab purposes to maximize attack surface.

---

## 🔍 Key Learnings

1. **Detection Logic** — Understanding what telemetry real attacks generate vs. theory
2. **MITRE ATT&CK Mapping** — Every alert maps to real-world adversary techniques
3. **Defense in Depth** — Multiple layers (Defender + Wazuh) catch what individual tools miss
4. **SOC Analyst Perspective** — Experiencing both sides: launching attacks and triaging alerts
5. **Log Analysis** — Reading raw Windows Event Logs and understanding what they mean

---

## 🚀 Future Improvements

- [ ] Add Active Directory domain controller VM
- [ ] Simulate Kerberoasting and Pass-the-Hash attacks
- [ ] Build custom Wazuh detection rules
- [ ] Add honeypot to capture attacker TTPs
- [ ] Integrate threat intelligence feeds
- [ ] Practice lateral movement detection

---

## 📁 Repository Structure

```
wazuh-siem-lab/
├── README.md
├── screenshots/
│   ├── dashboard-overview.png
│   ├── security-alerts.png
│   ├── agent-overview.png
├── configs/
│   └── wazuh-config.yml
└── notes/
    └── installation-notes.md
```

---

## 👤 Author

**Mohamed Ghazi**  
Cybersecurity Student | Graduating July 2026  
Background: SOC Analyst → Security Engineer → Penetration Tester  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/mohamed-ghazi-323163262)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black)](https://github.com/mohamedghazi-creator)

---

## ⚠️ Disclaimer

This lab was built for educational purposes only. All attacks were performed on virtual machines owned and controlled by me in an isolated network environment. Never perform security testing on systems you do not own or have explicit permission to test.
