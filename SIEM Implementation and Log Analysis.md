<div align="center">
<h1> SIEM Implementation and Log Analysis </h1>
</div>

### Objective

Deploy a functional SIEM capable of collecting, centralizing and analyzing 
logs from multiple endpoints in real time.

---

## Architecture

```
┌─────────────────────────────────────────────┐
│              MacBook M2 (Host)              │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │         OrbStack (Ubuntu)           │   │
│  │         Wazuh Server + Manager      │   │
│  │         Dashboard : localhost       │   │
│  └──────────────┬──────────────────────┘   │
│                 │                           │
│    ┌────────────┴────────────┐             │
│    │                         │             │
│  ┌─┴──────────┐   ┌──────────┴──────┐     │
│  │ Windows 11 │   │   Ubuntu VM     │     │
│  │ Parallels  │   │   Parallels     │     │
│  │ Wazuh Agent│   │  Wazuh Agent    │     │
│  └────────────┘   └─────────────────┘     │
│                                             │
│  ┌─────────────────────────┐               │
│  │     Kali Linux          │               │
│  │     Parallels           │               │
│  │  Attack Simulation 🔴   │               │
│  └─────────────────────────┘               │
└─────────────────────────────────────────────┘
```

> Suricata will be integrated soon on the Ubuntu VM for network detection.

---

## Installation

### Wazuh Server (OrbStack — Ubuntu ARM64)

Installation via the official Wazuh script :

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Dashboard accessible at `https://localhost` from the Mac.

---

### Wazuh Agent — Windows 11 (Parallels)

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.5-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.139.171'

NET START Wazuh
```

---

### Wazuh Agent — Ubuntu VM (Parallels)

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.5-1_arm64.deb && sudo WAZUH_MANAGER='192.168.139.171' dpkg -i ./wazuh-agent_4.14.5-1_arm64.deb

sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---


## Screenshots

<img width="1512" height="508" alt="Capture d’écran 2026-06-07 à 13 32 10" src="https://github.com/user-attachments/assets/e86fcf66-32ec-4635-82de-9c805a37b957" />
<img width="1512" height="624" alt="Capture d’écran 2026-06-07 à 13 32 03" src="https://github.com/user-attachments/assets/f1a81d13-2386-4070-9f5e-47cd77024580" />
<img width="1505" height="748" alt="Capture d’écran 2026-06-07 à 13 31 53" src="https://github.com/user-attachments/assets/8f739c32-b3a8-4eaa-9258-f1e79645ada6" />


---

## Skills Demonstrated

- SIEM deployment on ARM64 environment
- Connection and monitoring of heterogeneous endpoints (Windows + Linux)
- Real attack detection via Wazuh rules
- Log analysis and event correlation
