# 🪟 Project 06 — Windows Forensics Investigation

**Tools:** PowerShell, Windows Event Viewer  
**System:** Windows 10/11  
**Date:** May 2026  
**Status:** ✅ Completed

---

## 🎯 Objective
Perform Windows forensic analysis using PowerShell to investigate
Event Logs, identify suspicious login activity, audit Registry
autorun keys and analyze system shutdown history.

---

## 🔬 Methodology

### 1. Event Log Enumeration
```powershell
Get-EventLog -List
```

**Logs found:**
| Log | Entries |
|---|---|
| Security | 31,530 |
| System | 55,025 |
| Application | 55,847 |
| Windows PowerShell | 4,721 |

---

### 2. Security Event Analysis
```powershell
Get-EventLog -LogName Security -Newest 100 | Where-Object {
  $_.EventID -in @(4624,4625,4634,4648,4672,4720,4728)
} | Select-Object TimeGenerated, EventID, Message | Format-Table -AutoSize
```

**Critical EventIDs monitored:**
| EventID | Name | Found |
|---|---|---|
| 4624 | Successful Logon | ✅ Yes — all SYSTEM services |
| 4625 | Failed Logon | ⚠️ Yes — 7 attempts |
| 4672 | Special Privileges | ✅ Yes — normal SYSTEM |
| 4720 | Account Created | ❌ None |
| 4728 | Added to Admin Group | ❌ None |

---

### 3. Failed Login Investigation
```powershell
Get-EventLog -LogName Security | Where-Object {$_.EventID -eq 4625} | Select-Object -First 5 TimeGenerated, Message | Format-List
```

**Findings — 7 Failed Logins:**

| Source | Count | Type | IP | Risk |
|---|---|---|---|---|
| AsusSoftwareManager.exe | 3 | Logon Type 2 (Local) | N/A | 🟡 Low |
| User: guest | 4 | Logon Type 3 (Network) | 192.168.1.163 | 🟢 Info |

**Analysis:**
- AsusSoftwareManager failures = outdated stored credentials in ASUS software
- Guest login attempts from 192.168.1.163 = internal Kali Linux VM (authorized lab activity)
- No external unauthorized access detected ✅

---

### 4. Shutdown History Analysis
```powershell
Get-EventLog -LogName System | Where-Object {
  $_.EventID -in @(1074,6006,6008)
} | Select-Object -First 10 TimeGenerated, EventID, Message | Format-Table -AutoSize
```

**Results:**
| Date | EventID | Type |
|---|---|---|
| 5/19/2026 3:23 AM | 6006 | Clean shutdown ✅ |
| 5/18/2026 8:44 PM | 1074 | User initiated restart ✅ |
| 5/17/2026 4:09 PM | 6008 | ⚠️ Unexpected shutdown |
| 5/15/2026 12:11 PM | 6008 | ⚠️ Unexpected shutdown |

**Analysis:** 2 unexpected shutdowns detected — likely caused by lab activity or OS updates

---

### 5. Registry Autorun Analysis
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
```

**HKLM Run — System Level (4 entries):**
| Program | File | Status |
|---|---|---|
| SecurityHealth | SecurityHealthSystray.exe | ✅ Legitimate |
| RtkAudUService | RtkAudUService64.exe | ✅ Legitimate |
| AvastUI | AvLaunch.exe | ✅ Legitimate |
| Bdagent | bdagent.exe | ✅ Legitimate |

**HKCU Run — User Level (11 entries):**
| Program | Status |
|---|---|
| OneDrive, Discord, Steam | ✅ Legitimate |
| Claude, Docker Desktop | ✅ Legitimate |
| Grammarly, Canva | ✅ Legitimate |
| Bitdefender Hub, Avast Browser | ✅ Legitimate |

**No malicious autorun entries detected ✅**

---

## 🚨 Findings Summary

| # | Severity | Finding | Action |
|---|---|---|---|
| 1 | 🟡 Low | AsusSoftwareManager login failures | Reconfigure ASUS credentials |
| 2 | 🟢 Info | Guest login attempts from Kali VM | Authorized lab activity |
| 3 | 🟡 Low | 2 unexpected system shutdowns | Monitor for recurrence |
| 4 | 🟢 Info | 15 autorun entries | All legitimate programs |

---

## 📋 Incident Report

```
CASE: Windows Forensics Investigation
SYSTEM: MICHAELGLAPTOP
ANALYST: Michael Galindez (GhostForensic)
DATE: 2026-05-20

EXECUTIVE SUMMARY:
PowerShell-based forensic analysis of Windows Event Logs and 
Registry completed. No malicious indicators found.

FINDINGS:
1. LOGIN FAILURES (LOW) — 7 total, all explained and authorized
2. UNEXPECTED SHUTDOWNS (LOW) — 2 events, lab-related
3. REGISTRY AUTORUN (INFO) — 15 entries, all legitimate

CONCLUSION: System clean. No indicators of compromise detected.
```

---

## 🧠 SOC Relevance

Windows Event Log analysis is performed in SOC for:
- **Threat Hunting** — detecting unauthorized access attempts
- **Incident Response** — reconstructing attack timelines
- **Compliance** — auditing user activity and privilege use
- **Forensics** — evidence collection for legal proceedings

Registry analysis detects:
- **Malware persistence** — unauthorized autorun entries
- **Lateral movement** — unexpected program additions
- **Living off the Land** — abuse of legitimate Windows tools

---

## 🔑 Key EventIDs Reference

| EventID | Description | SOC Priority |
|---|---|---|
| 4624 | Successful logon | Monitor |
| 4625 | Failed logon | 🔴 Alert |
| 4648 | Logon with explicit credentials | 🔴 Alert |
| 4672 | Special privileges assigned | Monitor |
| 4720 | User account created | 🔴 Critical |
| 4728 | Member added to admin group | 🔴 Critical |
| 6008 | Unexpected system shutdown | Investigate |
