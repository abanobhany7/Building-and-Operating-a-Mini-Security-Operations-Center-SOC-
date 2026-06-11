# Building and Operating a Mini Security Operations Center (SOC)

A hands-on simulation of a fully functional SOC, built and operated over four weeks. The project covers the complete security incident lifecycle — from infrastructure setup and log ingestion, to threat detection, alert triage, incident management, and post-incident root cause analysis.

> **Team:** Threat Shield Alliance
> **Supervisor:** Eng. Wessam Elkhaligy
> **Team Members:** Abanob Hany Roushdy · Abdelrahman Ahmed · Aya Mohamed Mahmoud Zaki · Mohamed Ali Abdelgawad Elnoamani · Omar Samy Samir

---

## Table of Contents

- [Project Overview](#-project-overview)
- [SOC Architecture](#-soc-architecture)
- [Tools & Stack](#-tools--stack)
- [Weekly Deliverables](#-weekly-deliverables)
- [Detected Incidents](#-detected-incidents)
- [KPIs & Results](#-kpis--results)
- [MITRE ATT&CK Coverage](#-mitre-attck-coverage)
- [Key Findings & Recommendations](#-key-findings--recommendations)

---

## Project Overview

This project simulates a real-world SOC environment, executing controlled cyberattacks against a monitored victim machine and detecting them using a SIEM. The three attack vectors successfully detected were:

| Attack Type | Tool Used | Outcome |
|---|---|---|
| Network Reconnaissance | Nmap | Detected via Suricata IDS |
| RDP Brute Force | Hydra | Detected via Winlogbeat + SIEM Rule |
| Malware Delivery | Metasploit | Detected & Quarantined by Defender |

---

## SOC Architecture

The SOC is divided into three zones:

```
┌─────────────────────────────────────────────────────────┐
│                    ATTACKER ZONE                        │
│              Parrot OS (192.168.148.135)                │
│         [ Nmap | Metasploit | Hydra ]                   │
└──────────────────────┬──────────────────────────────────┘
                       │ Attack Traffic
┌──────────────────────▼──────────────────────────────────┐
│                 VICTIM / MONITORING ZONE                │
│  pfSense Firewall ──► Windows 10 Pro (192.168.148.138)  │
│  [Syslog → ELK]       [Winlogbeat → Elasticsearch]     │
└──────────────────────┬──────────────────────────────────┘
                       │ Log Forwarding (HTTPS / Syslog)
┌──────────────────────▼──────────────────────────────────┐
│                    SOC CORE ZONE                        │
│         Ubuntu Server — Elastic Stack (ELK)             │
│    [ Elasticsearch | Logstash | Kibana | Suricata ]     │
└─────────────────────────────────────────────────────────┘
```

**Data Flow:**
- Windows Events → Winlogbeat (HTTPS) → Elasticsearch
- Firewall Events → pfSense Syslog (UDP/514) → Elastic listener
- Network Traffic → Suricata IDS → Filebeat → Elasticsearch

---

## Tools & Stack

| Category | Tool |
|---|---|
| SIEM | Elastic Stack (Elasticsearch, Logstash, Kibana) |
| Endpoint Log Shipper | Winlogbeat |
| Network IDS | Suricata |
| Firewall | pfSense |
| Attacker Machine | Parrot OS |
| Attack Tools | Nmap, Metasploit, Hydra |
| Endpoint Protection | Windows Defender |

---

## Weekly Deliverables

### Week 1 — SOC Setup & Log Ingestion
- Designed the three-zone SOC architecture
- Deployed and verified Elastic Stack (Elasticsearch + Kibana + Logstash) on Ubuntu Server
- Configured Winlogbeat on Windows 10 endpoint (capturing Security, System, Application, and Sysmon event channels)
- Validated live log ingestion via Kibana Discover

### Week 2 — SIEM Configuration & Use Case Development
Developed three detection rules mapped to the **Cyber Kill Chain**:

**Use Case 1 — RDP Brute Force**
- Trigger: 5+ Event ID `4625` (Logon Failure) from the same source IP within 5 minutes
- Logon Types: 3 (Network) or 10 (Remote Interactive)

**Use Case 2 — Malware Infection**
- Trigger: Event ID `1116` (Windows Defender malware detection) — immediate critical alert
- Action: Isolate host, identify file hash and path

**Use Case 3 — Network Reconnaissance**
- Trigger: Suricata signature `ET SCAN Possible Nmap User-Agent Observed`
- Data Source: Suricata IDS logs via Filebeat

All rules were aggregated into a unified **Elastic Security Alert Dashboard**, prioritized by severity.

### Week 3 — Alert Triage & Incident Management
Followed a structured triage process for every alert:
```
Validate (True/False Positive) → Scope (Affected Assets) → Contain → Eradicate
```
Full triage sheets were completed for all three incidents (see below).

### Week 4 — Reporting, Analysis & KPIs
- Computed SOC KPIs (MTTD, TPR, coverage)
- Performed Root Cause Analysis (RCA) on the Malware Infection incident
- Produced corrective action recommendations

---

## Detected Incidents

### INC-SCAN-003 · Network Reconnaissance
| Field | Details |
|---|---|
| Severity | Medium |
| Source IP | 192.168.148.135 (Parrot OS) |
| Target | 192.168.148.138 (Ports 5357, 443, 80, 3389) |
| Detection | Suricata — `ET SCAN RDP Connection Attempt from Nmap` (Rule 2036252) |
| MITRE | TA0007 Discovery → T1046 Network Service Discovery |
| Status | Monitored — precursor to RDP attack |

---

### INC-BRUTE-001 · RDP Brute Force Attack
| Field | Details |
|---|---|
| Severity | High (Risk Score: 73) |
| Source IP | 192.168.148.135 |
| Target | DESKTOP-D743P00 / Administrator account |
| IOCs | Event ID 4625 · Logon Type 3 · IP 192.168.148.135 |
| MITRE | TA0006 Credential Access → T1110.001 Password Guessing |
| Containment | Source IP blocked at pfSense firewall |
| Outcome | No successful logins confirmed |

---

### INC-MAL-002 · Malware Infection
| Field | Details |
|---|---|
| Severity | Critical (Risk Score: 99) |
| Malware | `Trojan:Win64/Metasploit!pz` |
| File Path | `C:\Users\Administrator\Desktop\reverse.exe` |
| Target | DESKTOP-D743P00\Administrator |
| MITRE | TA0002 Execution → T1204.002 Malicious File |
| Status | Auto-remediated — quarantined by Windows Defender |
| Root Cause | No web filtering on pfSense; execution allowed from user-writable temp folder |

---

## KPIs & Results

| Metric | Result |
|---|---|
| Total Alerts Processed | 15 |
| True Positive Rate (TPR) | 100% |
| False Positives | 0 |
| Mean Time To Detect (MTTD) | < 1 minute |
| Asset Visibility Coverage | 100% (Firewall + Endpoint) |

---

## MITRE ATT&CK Coverage

| Tactic | Technique | Sub-Technique | Incident |
|---|---|---|---|
| TA0007 — Discovery | T1046 Network Service Discovery | — | INC-SCAN-003 |
| TA0006 — Credential Access | T1110 Brute Force | T1110.001 Password Guessing | INC-BRUTE-001 |
| TA0002 — Execution | T1204 User Execution | T1204.002 Malicious File | INC-MAL-002 |

---

## Key Findings & Recommendations

**Finding 1:** The Nmap scan preceded the RDP brute force, demonstrating a classic reconnaissance-then-attack pattern. Correlating these alerts across time provided a full attack narrative.

**Finding 2:** The malware was delivered to `AppData\Local\Temp`, a common staging location for payloads that bypasses basic AV on download.

**Recommendations:**
1. **AppLocker / SRP** — Block execution from user-writable directories (`%TEMP%`, `AppData`)
2. **pfSense Web Filtering** — Enable content filtering to block malicious file downloads at the network perimeter
3. **Suricata IPS Mode** — Switch from detection-only (IDS) to inline blocking (IPS) to drop known malicious payloads before they reach endpoints
4. **Account Lockout Policy** — Enforce lockout after 5 failed login attempts to limit brute-force effectiveness

---


## Technologies & Concepts

`ELK Stack` `SIEM` `Winlogbeat` `Suricata IDS` `pfSense` `Incident Response` `Threat Detection` `MITRE ATT&CK` `Brute Force Detection` `Malware Analysis` `Network Forensics` `SOC Operations` `Cyber Kill Chain`

---

*Built as part of the Ruwad Masr Al-Raqmia program — Ministry of Communications and Information Technology, Egypt.*
