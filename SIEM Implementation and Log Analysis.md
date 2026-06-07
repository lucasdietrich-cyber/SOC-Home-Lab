# 🛡️ SIEM Implementation and Log Analysis

## Objectif

Déployer un SIEM fonctionnel capable de collecter, centraliser et analyser 
les logs de plusieurs endpoints en temps réel.

---

## 🏗️ Architecture

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
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update && sudo apt install wazuh-agent
sudo WAZUH_MANAGER='<IP_SERVEUR>' systemctl start wazuh-agent
```

---

## 🔍 Détections réalisées

| Attaque | Source | Règle déclenchée | MITRE ATT&CK |
|---------|--------|------------------|--------------|
| SSH Brute Force | Kali Linux → Ubuntu VM | Multiple authentication failures | T1110 — Brute Force |

---

## 📸 Screenshots

> À ajouter : dashboard Wazuh, alertes SSH brute force, agents connectés

---

## 🎯 Compétences démontrées

- Déploiement d'un SIEM sur environnement ARM64
- Connexion et supervision de endpoints hétérogènes (Windows + Linux)
- Détection d'attaques réelles via règles Wazuh
- Analyse de logs et corrélation d'événements
```

Les commandes sont approximatives — corrige-les si elles correspondent pas exactement à ce que t'as utilisé. Et remplace `<IP_SERVEUR>` par ce que t'as mis réellement.
