# Digital Forensics Lab - Analyse BlackEnergy v2

**Investigateur**: Thomas FERET (ThomasWEBDEV)
**Statut**: Termine
**Date**: Fevrier 2026

---

## Contexte du projet

Ce projet presente une investigation forensics complete d'une compromission par le malware BlackEnergy v2.
Il s'appuie sur le challenge "BlackEnergy" de la plateforme CyberDefenders (note 4.5/5, difficulte Medium).

Developpe dans le cadre d'une recherche d'alternance en cybersecurite (SOC Analyst / Incident Responder).

---

## Resultats de l'investigation

### Systeme compromis

| Propriete | Valeur |
|-----------|--------|
| OS | Windows XP SP3 (build 2600) |
| Architecture | 32 bits (x86) |
| Date de compromission | 2023-02-13 18:25:26 UTC |

### Malware identifie

**BlackEnergy v2** - malware utilise dans des attaques contre des infrastructures critiques (notamment Ukraine 2015).

### Chaine d'attaque reconstituee
```
rootkit.exe (execution utilisateur)
    |
    +--> Installation service kernel (hvklkzdz / str.sys)
    |        Persistence au demarrage Windows
    |
    +--> Injection DLL dans svchost.exe (PID 880)
    |        msxml3r.dll a l'adresse 0x9a0000
    |
    +--> Lancement cmd.exe
             Execution de commandes
```

### IOCs identifies

| Type | Valeur | Role |
|------|--------|------|
| Fichier | rootkit.exe | Dropper initial |
| Fichier | str.sys | Driver rootkit kernel |
| Fichier | suzmkbtuoyeki.sys | Alias driver (nom aleatoire) |
| Fichier | msxml3r.dll | DLL injectee dans svchost.exe |
| Registre | SYSTEM\ControlSet001\Services\hvklkzdz | Service de persistence |
| Memoire | 0x980000 (svchost PID 880) | Adresse injection code |

### MITRE ATT&CK

| Tactique | Technique | Evidence |
|----------|-----------|----------|
| Execution | T1204.002 - Malicious File | rootkit.exe execute par l'utilisateur |
| Persistence | T1543.003 - Windows Service | Service hvklkzdz, demarrage automatique |
| Defense Evasion | T1055.001 - DLL Injection | msxml3r.dll dans svchost.exe |
| Defense Evasion | T1014 - Rootkit | Driver kernel hvklkzdz |
| Defense Evasion | T1036.005 - Masquerading | msxml3r.dll imite msxml3.dll |
| Execution | T1059.003 - cmd.exe | cmd.exe lance par rootkit.exe |

---

## Outils utilises

| Outil | Version | Usage |
|-------|---------|-------|
| Volatility 3 | 2.27.0 | Analyse memoire (10 plugins) |
| Python | 3.12 | Environnement d'execution |

---

## Commandes cles
```bash
# Activation environnement
source venv/bin/activate

# Identification du systeme
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.info

# Analyse des processus
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.pslist

# Detection injection de code
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.malfind

# Analyse DLLs du processus suspect
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.ldrmodules --pid 880

# Fichiers ouverts par le processus suspect
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.handles --pid 880

# Analyse des drivers
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.driverscan

# Analyse persistence registre
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.registry.printkey --offset 0xe1035b60 --key "ControlSet001\Services\hvklkzdz"
```

---

## Methodologie

Investigation menee selon la methodologie **SANS PICERL** :

| Phase | Actions |
|-------|---------|
| Preparation | Installation Volatility 3, acquisition dump memoire |
| Identification | Analyse processus, detection injection, identification malware |
| Containment | Extraction IOCs, analyse drivers, analyse registre |
| Eradication | Reconstitution chaine d'attaque, documentation TTPs |
| Recovery | Recommandations remediation |
| Lessons Learned | Rapport IR complet, documentation portfolio |

---

## Structure du projet
```
digital-forensics-lab/
├── README.md
├── .gitignore
├── data/
│   ├── memory/          # Dump memoire (non versionne - 2.1 Go)
│   └── disk/
├── analysis/            # Resultats des plugins Volatility
│   ├── windows_info.txt
│   ├── pslist.txt
│   ├── malfind.txt
│   ├── dlllist_880.txt
│   ├── ldrmodules_880.txt
│   ├── handles_880.txt
│   ├── cmdline.txt
│   ├── driverscan.txt
│   ├── hivelist.txt
│   └── registry_hvklkzdz.txt
├── reports/
│   └── incident_response_report.md
└── config/
    ├── tools.md
    └── commands.md
```

---

## Source

**Plateforme**: CyberDefenders (https://cyberdefenders.org)
**Challenge**: BlackEnergy - Endpoint Forensics
**Score**: 8/8 questions (100%)
**Difficulte**: Medium

---

## Auteur

**Thomas FERET**
Etudiant en cybersecurite - Specialisation Blue Team / DFIR
Recherche alternance : SOC Analyst / Incident Responder

Autres projets :
- [IDS Detection Lab](https://github.com/ThomasWEBDEV/ids-detection-lab) - Suricata + ELK Stack
