# Home-SOC-Lab---Simulated-Intrusion-Privilege-Escalation-Investigation

## Overview
This project documents a simulated intrusion conducted in a home cybersecurity lab using **Kali Linux as the attacker and Ubuntu as the target system.**

The goal of this lab was to simulate a realistic attack lifecycle and analyze it from a SOC analyst perspective, focusing on:
- Detecting brute-force authentication attempts
- Investigating suspicious login activity
- Analyzing Linux authentication logs
- Identifying privilege escalation behavior
- Extracting Indicators of Compromise (IoCs)

All attack activity was intentionally generated in a controlled environment and analyzed using system logs.

## Lab Environment
**Target System**
- Ubuntu Virtual Machine
**Attacker System**
- Kali Linux Virtual Machine 
**Network**
- Isolated virtual lab environment
**Key Tools Used**
- Nmap
- Hydra
- SSH
- Linux log analysis tools (grep, last, who)
**Logs analyzed:**
- /var/log/auth.log
- /var/log/syslog

## Attack Simulation Workflow
The simulated intrusion followed a typical attack lifecycle.
**1. Reconnaissance**
The attacker performed network reconnaissance to identify exposed services.

Example scans: </br>
   nmap -sS 192.168.x.x </br>
   nmap -sV 192.168.x.x </br>
   nmap -A 192.168.x.x <br>
   nmap -p- 192.168.x.x
   
**Findings:**
- Only port 22 (SSH) was open
- SSH service version identified
- Target OS fingerprinted as Linux
- Minimal attack surface observed
This confirmed the environment was a clean lab system with limited exposed services.

## 2. SSH Brute-Force Simulation
Brute-force login attempts were generated to simulate unauthorized access attempts.

Example commands: </br>
ssh testuser@192.168.x.x

Automated attempts: </br>
hydra -l testuser -P passlist.txt ssh://192.168.x.x

Observed behavior:
- Repeated failed login attempts
- Authentication failures recorded in system logs
- Pattern consistent with automated login attempts

## Log Analysis
Authentication logs were analyzed to detect suspicious activity.

Examples: </br>
sudo grep "Failed password" /var/log/auth.log </br>
sudo grep "Invalid user" /var/log/auth.log </br>
sudo grep "Accepted password" /var/log/auth.log </br>
sudo grep sshd /var/log/auth.log

Indicators observed: </br>
- Multiple failed authentication attempts
- Invalid user login attempts
- Successful login events
- Repeated connections from the same IP address

These patterns clearly demonstrate brute-force attack activity.

## 4. Session Investigation
User activity and sessions were reviewed using: </br>
       last </br>
       who
These commands helped verify:
- Active login sessions
- Historical login attempts
- Potential unauthorized access
## 5. Privilege Escalation Simulation
A controlled misconfiguration was introduced to simulate a privilege escalation vulnerability.

The user testuser was granted passwordless sudo access to the find binary.

Sudo configuration: </br>
   testuser ALL=(ALL) NOPASSWD: /usr/bin/find 

This allowed execution of root commands via the -exec option.

Exploitation: </br>
   sudo -l </br>
   sudo find . -exec /bin/bash \; -quit </br>
   whoami

Result: </br>
   root </br>
This confirmed **successful privilege escalation.**

## Log Evidence of Escalation
Escalation activity was visible in authentication logs.

Example detection: </br>
sudo grep sudo /var/log/auth.log

Example log entry: </br>
sudo: </br>
testuser : COMMAND=/usr/bin/find </br>

This provides a clear **Indicator of Compromise** for escalation activity.

## Indicators of Compromise (IoCs)
| IoC Type                 | Example                                  | Description                     |
|--------------------------|------------------------------------------|---------------------------------|
| Suspicious IP            | 192.168.x.x                              | Source of repeated SSH attempts |
| Failed Logins            | Failed password for testuser             | Brute-force attempt             |
| Authentication Failures  | pam_unix(sshd:auth)                      | Repeated login failures         |
| Privilege Escalation     | sudo: testuser : COMMAND=/usr/bin/find   | Misconfigured sudo execution    |
| Automated Login Behavior | Multiple attempts per second             | Scripted brute-force activity   |

## Mitigation Strategies
Security improvements to prevent this attack scenario include:
- Enforce strong password policies
- Implement multi-factor authentication for SSH
- Follow the least privilege principle
- Monitor authentication logs for abnormal patterns
- Restrict SSH access to trusted IP addresses
- Regularly audit sudoers configurations

## Remediation Actions
**Containment**
- Block suspicious IP addresses
- Lock affected user accounts
**Eradication**
Remove insecure sudo configuration: </br>
sudo visudo

Remove:</br>
testuser ALL=(ALL) NOPASSWD: /usr/bin/find

Verify: </br>
sudo -l
**Recovery**
-Reset user credentials
-Review running processes
- Audit system configurations

## Key Skills Demonstrated
This project demonstrates practical experience in:
- SOC investigation workflows
- Linux log analysis
- Brute-force attack detection
- Privilege escalation analysis
- Indicator of Compromise identification
- Incident response documentation

## Key Takeaway
This lab highlights how simple configuration mistakes combined with weak monitoring can allow attackers to move from initial access to full system compromise.

Effective log monitoring, proper privilege management, and strong authentication controls are essential to detect and prevent these attacks.

## Note
All analysis was performed in a controlled home lab environment. Sensitive information such as real IP addresses has been sanitized.

## Author

Durga Sai Sri Ramireddy  
Master's in Cybersecurity
