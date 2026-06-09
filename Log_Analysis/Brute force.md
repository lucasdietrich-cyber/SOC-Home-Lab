# SOC001 - SSH Brute Force Detected

## Executive Summary

On June 9, 2026 at 16:35, Wazuh detected a series of failed SSH login attempts on agent **sysvoid** (10.211.55.7). The attack originated from **10.211.55.3** and involved over **491 password attempts** against the user `admin`. The attacker never gained access — the user doesn't exist on the system and the correct password wasn't in the wordlist. **True Positive — Attack failed, no compromise.**

## Incident Details

| | |
| :--- | :--- |
| **Date** | June 9, 2026 @ 16:35 |
| **Attacker IP** | 10.211.55.3 (Kali Linux) |
| **Target** | sysvoid — 10.211.55.7 |
| **Username targeted** | `admin` (non-existent) |
| **Tool used** | Hydra |
| **Passwords tried** | 491 |
| **Result** | Failed — no valid credentials found |
| **Rules triggered** | 5710 (level 5), 2502 (level 10) |

## Attack Simulation

The attack was launched from Kali Linux using Hydra with a custom wordlist:

```bash
hydra -l admin -P password.txt 10.211.55.7 ssh
```

The wordlist was a short custom list. The real password wasn't in it, and the user `admin` doesn't exist on the target machine — so the attack had no chance of succeeding. Still, Wazuh picked up everything.

## Detection — What Wazuh Saw

In **Threat Hunting > Events**, filtering by `rule.id: 2502` returned **61 hits**. Expanding the events confirmed the source IP, the targeted username, and the auth log location.

<img width="1511" height="810" alt="Capture d’écran 2026-06-09 à 16 40 59" src="https://github.com/user-attachments/assets/fdc1c47d-c288-4ec0-a98b-9ff9c22636a8" />
<img width="741" height="736" alt="Capture d’écran 2026-06-09 à 16 38 56" src="https://github.com/user-attachments/assets/a8fd488f-ba2e-4d8f-b205-ded2e1774198" />




Opening one of the alert documents showed:

```
2026-06-05T22:17:12 sysvoid sshd[56584]: Failed password for invalid user admin
from 10.211.55.3 port 59586 ssh2
```

<img width="1512" height="727" alt="Capture d’écran 2026-06-09 à 16 51 41" src="https://github.com/user-attachments/assets/aefda8f3-db6e-476e-b480-4717faaf1760" />
<img width="753" height="733" alt="Capture d’écran 2026-06-09 à 16 50 36" src="https://github.com/user-attachments/assets/b8899e72-6968-422d-b7a7-d4e679253f8c" />
<img width="755" height="730" alt="Capture d’écran 2026-06-09 à 16 50 32" src="https://github.com/user-attachments/assets/896136db-7ca1-4b5d-adcc-972a525c0083" />



Key fields:

| Field | Value |
| :--- | :--- |
| `data.srcip` | 10.211.55.3 |
| `data.srcuser` | admin |
| `rule.firedtimes` | 376 (rule 5710) / 61 (rule 2502) |
| `rule.mitre.id` | T1110.001, T1021.004 |
| `rule.mitre.tactic` | Credential Access, Lateral Movement |

## MITRE ATT&CK

| Tactic | Technique | ID |
| :--- | :--- | :--- |
| Credential Access | Brute Force: Password Guessing | T1110.001 |
| Lateral Movement | Remote Services: SSH | T1021.004 |

## Verdict

- **True Positive** — the attack was real and detected correctly
- **No compromise** — wrong user, password not in list
- In a real environment this would warrant blocking 10.211.55.3 at the firewall and enabling Fail2ban

---
*Lucas Dietrich — SOC Homelab | Wazuh 4.x | Ubuntu ARM64 | OrbStack / MacBook M2*
