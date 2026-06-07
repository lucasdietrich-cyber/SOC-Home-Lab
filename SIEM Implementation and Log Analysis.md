# 🛡️ SIEM Implementation and Log Analysis

## Objectif

Déployer un SIEM fonctionnel capable de collecter, centraliser et analyser 
les logs de plusieurs endpoints en temps réel.

---

## 🏗️ Architecture

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

> 🔄 Suricata sera intégré prochainement sur l'Ubuntu VM pour la détection réseau.

---

## ⚙️ Installation

### Wazuh Server (OrbStack — Ubuntu ARM64)

Installation via le script officiel Wazuh :

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Accès au dashboard via `https://localhost` depuis le Mac.

---

### Wazuh Agent — Windows 11 (Parallels)

Installation et enregistrement de l'agent via PowerShell :

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi `
  -OutFile wazuh-agent.msi
msiexec /i wazuh-agent.msi WAZUH_MANAGER='<IP_SERVEUR>' /q
NET START WazuhSvc
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


## 📸 Screenshots

> À ajouter : dashboard Wazuh, alertes SSH brute force, agents connectés

---

## 🎯 Compétences démontrées

- Déploiement d'un SIEM sur environnement ARM64
- Connexion et supervision de endpoints hétérogènes (Windows + Linux)
- Détection d'attaques réelles via règles Wazuh
- Analyse de logs et corrélation d'événements
