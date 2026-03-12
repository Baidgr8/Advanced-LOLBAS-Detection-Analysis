# Advanced LOLBAS Detection & Analysis

**Incident Type:** Living-off-the-Land (LOLBAS) Malware Delivery & Persistence  
**Analyst:** Babafemi A. Ikujebi  
**Date:** February 26, 2026  
**Environment:** Windows Sandbox (Isolated Forensic Lab)  
**Tools Used:** PowerShell, Certutil, Regedit, Auditpol, Get-WinEvent

---

# Project Overview (Recruiter Quick View)

This project simulates a **Living-off-the-Land (LOLBAS) attack** in which legitimate Windows utilities are abused to download a malicious payload and establish persistence.

The objective of the investigation was to detect attacker activity using **native Windows logs and forensic artifacts**, even in the absence of Sysmon.

### Key Investigation Activities

- Detection of malicious tool transfer using `certutil.exe`
- Registry persistence analysis
- Manual audit policy configuration using `auditpol`
- PowerShell artifact recovery
- Event log analysis and telemetry troubleshooting

### Key Findings

- Remote payload downloaded using **Certutil**
- Persistence established using **Registry Run Keys**
- Default Windows logging provided **limited visibility**
- **PowerShell history artifacts** confirmed attacker activity

### Incident Classification

**Confirmed Malicious Activity – High Risk Technique**

---

# SOC Case Snapshot

| Field | Details |
|------|------|
| Attack Type | Living-off-the-Land (LOLBAS) |
| Initial Access | Command execution via PowerShell |
| Tool Transfer | certutil.exe |
| Persistence Method | Registry Run Key |
| Environment | Windows Sandbox |
| Logging Challenge | No Sysmon / Limited telemetry |
| Key Artifact | PowerShell history |
| Detection Method | Artifact-based investigation |
| Risk Level | High |

---

# Executive Summary

This project simulates a **multi-stage Living-off-the-Land attack** using native Windows binaries.

The attack involved the use of **certutil.exe** to download a payload from an external source, followed by the creation of a **registry persistence mechanism**.

Because **Sysmon was not installed** and default Windows logging was limited, I manually configured **Security Auditing policies using `auditpol`** to capture process creation events.

Even after enabling auditing, **Event ID 4688 logs did not reliably capture the activity**, which required pivoting to alternative evidence sources.

The investigation ultimately confirmed:

- Successful tool transfer using **certutil**
- Execution of the downloaded payload
- Persistence via **registry autorun key**

This simulation demonstrates how attackers can abuse **trusted system binaries** to evade detection and why proper logging configuration is critical for SOC visibility.

---

# Investigation Methodology

The investigation followed a structured SOC workflow:

1. Simulate attacker tool transfer using certutil
2. Establish persistence using registry modification
3. Review Windows event logs
4. Configure advanced auditing with auditpol
5. Search for alternative forensic artifacts
6. Correlate evidence across system telemetry
7. Map techniques to MITRE ATT&CK
8. Document findings and remediation steps

---

# Investigation Report

## Phase 1 – Ingress Tool Transfer  
**MITRE Technique:** T1105 – Ingress Tool Transfer

In this phase, I simulated attacker behavior using **certutil.exe**, a legitimate Windows utility commonly abused for file transfers.

The command downloaded a remote payload from GitHub using:

- `-urlcache`
- `-split`

This demonstrates how attackers abuse **signed Microsoft binaries** to bypass traditional security controls.

### Screenshot – Certutil Command Execution

INSERT SCREENSHOT HERE

*(PowerShell terminal showing the certutil.exe command execution)*

---

## Phase 2 – Persistence via Registry Run Keys  
**MITRE Technique:** T1547.001 – Registry Run Keys

After downloading the payload, I simulated persistence by modifying:

path: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run

A new registry value named:

SecurityUpdate


was created to ensure the payload executes whenever the user logs in.

This technique is commonly used by malware to maintain persistence.

### Screenshot – Registry Persistence

INSERT SCREENSHOT HERE

*(Regedit window showing SecurityUpdate value in Run key)*

---

## Phase 3 – Telemetry & Log Analysis

### Challenge

The Windows Sandbox environment did **not contain Sysmon**, and default Windows logging did not initially capture the expected **Event ID 4688 process creation logs**.

To improve logging visibility, I enabled advanced auditing using:

auditpol


Despite enabling auditing, the execution of `certutil.exe` was not consistently logged.

This represents a **real-world SOC challenge where logging configurations are incomplete or misconfigured.**

### Investigation Pivot

Due to missing telemetry, I pivoted to alternative evidence sources:

- PowerShell command history
- Registry artifacts
- File system artifacts
- Manual command verification

Using PowerShell:

(Get-PSReadlineOption).HistorySavePath


I located the **PowerShell history file**, which contained the executed certutil command.

### Screenshot – Event Log / Artifact Evidence

INSERT SCREENSHOT HERE

*(Output highlighting certutil execution evidence)*

---

## Phase 4 – Forensic Artifact Recovery

Because event logging was unreliable, I examined **PowerShell history artifacts** stored in:

ConsoleHost_history.txt


Using:

(Get-PSReadlineOption).HistorySavePath


I retrieved the history file and confirmed the execution timeline of attacker commands.

This demonstrates how investigators can recover evidence **even when primary telemetry is incomplete.**

### Screenshot – PowerShell History Artifact

INSERT SCREENSHOT HERE

*(Terminal output displaying command history location)*

---

# Evidence

### Process Tree

PowerShell → certutil.exe → downloaded payload → registry modification

Explanation:

- PowerShell executed the certutil command
- certutil downloaded the remote payload
- payload saved locally
- registry run key created for persistence

### Screenshot – Process Tree

INSERT SCREENSHOT HERE

---

# Indicators of Compromise (IOCs)

| IOC Type | Indicator |
|------|------|
| LOLBAS Binary | certutil.exe |
| Registry Key | HKCU\Software\Microsoft\Windows\CurrentVersion\Run\SecurityUpdate |
| Artifact | PowerShell command history |
| Tool Transfer | Remote file retrieved via certutil |

---

# MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|------|------|------|
| Command & Control | Ingress Tool Transfer | T1105 |
| Persistence | Registry Run Keys | T1547.001 |
| Execution | Command and Scripting Interpreter (PowerShell) | T1059.001 |
| Defense Evasion | Living-off-the-Land Binary (Certutil) | T1218 |

---

# Attack Flow Analysis

The simulated attack followed this chain:

1. Attacker executes PowerShell command
2. PowerShell launches `certutil.exe`
3. certutil downloads remote payload
4. Payload saved locally
5. Registry Run key created for persistence
6. Payload executes at user logon

Attack Flow:

PowerShell → Certutil Download → Payload Execution → Registry Persistence → System Compromise

---

# Impact Assessment

| Category | Assessment |
|------|------|
| Tool Transfer | Successful |
| Persistence | Successfully established |
| Logging Visibility | Limited |
| Artifact Recovery | Successful |
| EDR Detection | Blocked on monitored system |

**Risk Level:** High

If executed on a production system without proper monitoring, this technique could enable:

- Persistent unauthorized access
- Remote payload execution
- Evasion of signature-based detection
- Continued attacker control after reboot

No lateral movement or data exfiltration occurred in this simulation.

---

# Detection Opportunities

Security teams could detect similar attacks through:

- Monitoring `certutil.exe` network activity
- Alerting on registry Run key modifications
- PowerShell command logging
- Detection of LOLBAS binaries executing suspicious downloads
- Endpoint detection monitoring

---

# Containment & Remediation Recommendations

### Immediate Actions

- Remove malicious registry entry
- Delete downloaded payload
- Terminate suspicious processes
- Isolate affected host

### Short-Term Actions

- Enable process creation auditing
- Deploy Sysmon
- Enable PowerShell logging

### Long-Term Improvements

- Implement behavioral EDR
- Monitor LOLBAS binaries
- Enable PowerShell script block logging
- Validate logging configurations regularly

---

# Lessons Learned

- Default Windows logging may provide insufficient visibility
- LOLBAS techniques can evade signature detection
- Proper auditing configuration improves investigation capability
- PowerShell artifacts provide valuable forensic evidence
- Behavioral monitoring significantly improves detection

---

# Skills Demonstrated

- LOLBAS attack simulation
- Registry persistence analysis
- Windows audit policy configuration
- Event log investigation
- PowerShell forensic artifact recovery
- MITRE ATT&CK mapping
- Endpoint security evaluation
- SOC investigation documentation
