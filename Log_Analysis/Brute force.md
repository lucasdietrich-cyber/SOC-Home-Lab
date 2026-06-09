# SOC001 - SSH Brute Force Detected

## Executive Summary

On June 5, 2026, Wazuh generated multiple alerts on agent **sysvoid** (10.211.55.7) for a sustained SSH brute-force attack originating from IP **10.211.55.3**. Log analysis revealed a high volume of failed SSH login attempts targeting non-existent user accounts, triggering rules 5710 and 2502 a combined total of **437 times**. No successful login was confirmed. This is a **True Positive** for an SSH brute-force attack, consistent with automated credential stuffing or dictionary attack tooling.

## Incident Details

| | |
| :--- | :--- |
| **Date of Incident** | June 5–9, 2026 |
| **Attacker IP** | 10.211.55.3 (Kali Linux — homelab simulation) |
| **Target Host** | sysvoid (10.211.55.7) |
| **Targeted Username** | `admin` (non-existent user) |
| **Attack Type** | SSH Brute-Force — Invalid User Enumeration |
| **Rules Triggered** | 5710 (level 5), 2502 (level 10) |
| **Total Alerts** | 437 (376 + 61 firedtimes) |

## Investigation and Analysis

### 1. Alert Detection — Wazuh

Wazuh detected the attack via two rules triggered on `/var/log/auth.log`:

- **Rule 5710** — `sshd: Attempt to login using a non-existent user` (level 5, firedtimes: 376)
- **Rule 2502** — `syslog: User missed the password more than one time` (level 10, firedtimes: 61)

The high `firedtimes` value confirms automated, high-volume activity inconsistent with manual login attempts.

<img width="1511" height="810" alt="Capture d’écran 2026-06-09 à 16 40 59" src="https://github.com/user-attachments/assets/6a47cf44-01df-444f-8b69-c0cb01d62c9b" />
<img width="741" height="736" alt="Capture d’écran 2026-06-09 à 16 38 56" src="https://github.com/user-attachments/assets/6c2769ca-84ec-4dba-af16-c72e175fc004" />


### 2. Log Analysis — Brute Force Pattern

Investigation in **Threat Hunting > Events** using DQL queries confirmed the attack pattern:

```
data.srcip: "10.211.55.3" AND rule.groups: "sshd"
```

Key fields extracted from the alert documents:

| Field | Value |
| :--- | :--- |
| `data.srcip` | 10.211.55.3 |
| `data.srcuser` | admin |
| `agent.ip` | 10.211.55.7 |
| `decoder.name` | sshd |
| `location` | /var/log/auth.log |
| `rule.mitre.id` | T1110.001, T1021.004 |
| `rule.mitre.tactic` | Credential Access, Lateral Movement |
| `rule.mitre.technique` | Password Guessing, SSH |

Sample log entry:
```
2026-06-05T22:17:12 sysvoid sshd[56584]: Failed password for invalid user admin from 10.211.55.3 port 59586 ssh2
```

<img width="753" height="733" alt="Capture d’écran 2026-06-09 à 16 50 36" src="https://github.com/user-attachments/assets/1ed0d29c-9a9c-45c6-8393-54887639d540" />
<img width="755" height="730" alt="Capture d’écran 2026-06-09 à 16 50 32" src="https://github.com/user-attachments/assets/80fb2bba-3645-4305-a6a2-ba0361d8b491" />


### 3. No Successful Login Confirmed

A review of the alerts showed no `sshd: Successful login` event (rule 5715) from the attacker IP. The brute force did not result in a compromise.

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
| :--- | :--- | :--- |
| Credential Access | Brute Force: Password Guessing | T1110.001 |
| Lateral Movement | Remote Services: SSH | T1021.004 |

## Skills and Tools Utilized

- **Tools:** Wazuh SIEM, OpenSearch Dashboards, Threat Hunting (DQL)
- **Skills:** SSH Log Analysis, Alert Triage, MITRE ATT&CK Mapping, DQL Query Writing

## Playbook — Incident Classification

- **Is Traffic Malicious?** **Yes.** High-volume automated login attempts from a single IP targeting invalid users.
- **Attack Type?** SSH Brute-Force — Invalid User Enumeration.
- **Was the Attack Successful?** **No.** No successful login detected.
- **Should the device be isolated?** **No** — attack was unsuccessful and source is a controlled homelab IP.
- **Incident Classification:** **True Positive — Contained.**

## Conclusion and Recommendations

The alert was a **true positive** for an SSH brute-force attack. The attack was unsuccessful — no credentials were compromised. In a real-world environment, the following actions would be recommended:

1. **BLOCK** the source IP at the firewall level immediately.
2. **HARDEN SSH** — disable password authentication, enforce SSH key-based login only.
3. **IMPLEMENT Fail2ban** — auto-ban IPs after N failed attempts.
4. **RESTRICT SSH access** — limit to known IPs or VPN only.
5. **MONITOR** for any subsequent successful login attempts from the same subnet.
6. **ESCALATE** to Tier 2 if a successful login is detected.

---

*Investigated by: Lucas Dietrich | Homelab environment | Wazuh 4.x on Ubuntu ARM64 (OrbStack/M2)*
