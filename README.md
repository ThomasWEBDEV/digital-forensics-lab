# Digital Forensics Lab - Analyses BlackEnergy v2 & HireMe

**Investigateur**: Thomas FERET (ThomasWEBDEV)
**Statut**: Terminé
**Date**: Février 2026
**Profil**: Développeur Web

## Vue d'ensemble du projet

Ce dépôt compile les résultats de deux investigations forensics complètes réalisées sur la plateforme **CyberDefenders**. Ce projet démontre la capacité à mener des analyses d'incidents (IR) et des investigations numériques (DFIR) sur des environnements variés.

| Challenge | Categorie | Difficulte | Score |
|-----------|-----------|------------|-------|
| **BlackEnergy** | Analyse Mémoire | Medium | 8/8 |
| **HireMe** | Analyse Disque | Medium | 17/17 |

---

## 1. Challenge BlackEnergy v2 - Analyse Mémoire

### Résumé
Investigation d'une variante inédite de BlackEnergy v2 sur Windows XP SP3.



### Chaine d'attaque reconstituee
1.  **rootkit.exe** (execution utilisateur) -> Installation service kernel -> Persistence
2.  **Injection DLL** dans svchost.exe (PID 880) -> msxml3r.dll
3.  **Lancement cmd.exe** -> Execution de commandes

### IOCs identifies
| Type | Valeur | Role |
|------|--------|-----------|
| Fichier | rootkit.exe | Dropper initial |
| Fichier | str.sys | Driver rootkit kernel |
| Fichier | msxml3r.dll | DLL injectee dans svchost.exe |
| Reg | SYSTEM\...\Services\hvklkzdz | Service de persistence |

### Commandes cles (Volatility 3)
```bash
vol -f dump.raw windows.info          # Identification systeme
vol -f dump.raw windows.malfind       # Detection injection de code
vol -f dump.raw windows.driverscan    # Analyse des drivers
```

---

## 2. Challenge HireMe - Analyse Disque

### Résumé
Analyse d'une image disque Windows 10 Pro (Horcrux.ad1) pour retracer l'activité de l'utilisateur "Karen" et identifier la persistance.



### Details de l'analyse
Analyse approfondie du registre SAM pour identifier la date de modification du mot de passe administrateur et de la navigation web (AlpacaCare).

### Questions resolues
Le détail des 17 questions résolues est disponible dans le fichier .

---

## Structure du projet

```text
digital-forensics-lab/
├── README.md
├── data/
│   ├── memory/          # Dump memoire (BlackEnergy)
│   └── disk/            # Image disque (HireMe)
├── analysis/
│   └── investigation_notes.md # Notes brutes des 17 questions
└── reports/
    └── incident_response_report.md
```

## Auteur

**Thomas FERET**
Développeur Web
