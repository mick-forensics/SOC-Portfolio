# 🔍 Project 04 — Network Traffic Analysis

**Tools:** Nmap, Wireshark, Nikto, Gobuster  
**Target:** scanme.nmap.org (legal practice server)  
**Date:** May 2025  
**Status:** ✅ Completed

---

## 🎯 Objective
Perform network reconnaissance and traffic analysis to identify 
open ports, vulnerable services and web security issues.

---

## 🔬 Methodology

### 1. Port Scanning — Nmap
```bash
nmap -Pn -sV -p 80,443,22 scanme.nmap.org
```

**Results:**
| Port | State | Service | Version | Risk |
|---|---|---|---|---|
| 22/tcp | open | SSH | OpenSSH 6.6.1p1 | 🔴 High |
| 80/tcp | open | HTTP | Apache 2.4.7 | 🔴 High |
| 443/tcp | closed | HTTPS | — | ⚠️ Medium |

### 2. Vulnerability Research — Searchsploit
```bash
searchsploit Apache 2.4.7
searchsploit OpenSSH 6.6.1
```

**Findings:**
- Apache 2.4.7 → 8 known CVEs including remote code execution
- OpenSSH 6.6.1 → Username enumeration vulnerability (CVE-2018-15473)

### 3. Web Vulnerability Scan — Nikto
**Critical findings:**
- Apache version outdated (2.4.7 vs current 2.4.66)
- Missing security headers: CSP, HSTS, X-Content-Type
- mod_negotiation MultiViews enabled — allows file brute force

### 4. Directory Enumeration — Gobuster
```bash
gobuster dir -u http://scanme.nmap.org -w /usr/share/wordlists/dirb/common.txt
```

**Critical findings:**
- `.svn` repository exposed (301) → possible source code leak
- `.htpasswd` confirmed (403) → password file present

### 5. Traffic Analysis — Wireshark
- Captured HTTP GET request to scanme.nmap.org
- Identified User-Agent: curl/8.19.0
- Confirmed unencrypted HTTP traffic — credentials visible in plain text

---

## 🚨 Key Findings Summary

| Severity | Finding | Impact |
|---|---|---|
| 🔴 Critical | .svn repository exposed | Source code & credential leak |
| 🔴 High | Apache 2.4.7 outdated | Remote code execution risk |
| 🔴 High | OpenSSH 6.6.1 outdated | Username enumeration |
| ⚠️ Medium | 5 missing HTTP headers | XSS, MITM attacks |
| ⚠️ Medium | .htpasswd confirmed | Password file exposed |

---

## 📋 Recommendations
1. Update Apache to latest version (2.4.66+)
2. Update OpenSSH to version 8.0+
3. Block access to .svn directory in Apache config
4. Implement all missing security headers
5. Enable HTTPS and enforce HSTS

---

## 🧠 SOC Relevance
This type of reconnaissance is performed during:
- Threat hunting to identify exposed attack surface
- Incident response to understand what attackers could see
- Vulnerability management to prioritize patching
