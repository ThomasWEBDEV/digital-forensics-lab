# Notes d'investigation

**Identifiant du cas**: DFIR-2026-001
**Investigateur**: Thomas FERET
**Date**: 2026-02-13
**Classification**: Interne

## Challenge

**Source**: CyberDefenders - BlackEnergy
**Categorie**: Endpoint Forensics / Analyse memoire
**Difficulte**: Moyen
**Score**: 8/8 questions resolues (100%)

## Scenario

Une entreprise multinationale a subi une cyberattaque resultant en un vol de donnees sensibles.
L'attaque a utilise une variante inedite du malware BlackEnergy v2.
Un dump memoire a ete obtenu depuis la machine infectee pour analyse.

## Systeme compromis

| Propriete | Valeur |
|-----------|--------|
| OS | Windows XP SP3 |
| Architecture | 32 bits (x86) |
| Profil Volatility | WinXPSP2x86 |
| Date du dump | 2023-02-13 18:29:11 UTC |

## Findings

### Processus suspects

| PID | Nom | PPID | Statut | Suspicion |
|-----|-----|------|--------|-----------|
| 964 | rootkit.exe | 1484 (explorer.exe) | Termine | Nom explicitement malveillant |
| 1960 | cmd.exe | 964 (rootkit.exe) | Termine | Lance par rootkit.exe |
| 880 | svchost.exe | 660 (services.exe) | Actif | Code injecte detecte |

### Injection de code

- **Processus cible**: svchost.exe (PID 880)
- **Adresse d'injection**: 0x980000
- **Indicateur**: MZ header detecte par malfind, PAGE_EXECUTE_READWRITE
- **DLL injectee**: msxml3r.dll (adresse base: 0x9a0000)

### Fichiers malveillants

| Fichier | Chemin | Type | Role |
|---------|--------|------|------|
| str.sys | C:\WINDOWS\system32\drivers\str.sys | Driver kernel | Rootkit BlackEnergy |
| msxml3r.dll | C:\WINDOWS\system32\msxml3r.dll | DLL injectee | Module malveillant |
| rootkit.exe | Bureau utilisateur | Executable | Dropper initial |

## IOCs

| Type | Valeur | Confiance |
|------|--------|-----------|
| Fichier | str.sys | Haute |
| Fichier | msxml3r.dll | Haute |
| Processus | rootkit.exe | Haute |
| Adresse memoire | 0x980000 | Haute |

## MITRE ATT&CK

| Tactique | Technique | Evidence |
|----------|-----------|----------|
| Defense Evasion | T1055 - Process Injection | Code injecte dans svchost.exe |
| Defense Evasion | T1014 - Rootkit | str.sys charge comme driver kernel |
| Execution | T1059.003 - cmd.exe | cmd.exe lance par rootkit.exe |
| Persistence | T1543.003 - Windows Service | Driver installe comme service |

## Statut

- [x] Challenge selectionne sur CyberDefenders
- [x] Fichiers telecharges
- [x] Analyse memoire (Volatility 3) - 8/8 questions
- [x] IOCs extraits
- [x] MITRE ATT&CK mappe
- [ ] Rapport final redige

## Phase 2 - Analyse disque (HireMe - CyberDefenders)

### Systeme analyse

| Propriete | Valeur |
|-----------|--------|
| Hostname | TOTALLYNOTAHACK |
| OS | Windows 10 Pro |
| Build number | 16299 |
| Utilisateur principal | Karen |
| Image | Horcrux.ad1 |
| Outil | Autopsy 4.22 + FTK Imager 4.5 |

### Questions resolues

| Question | Reponse | Methode |
|----------|---------|---------|
| Q1 - Administrateur | Karen | FTK Imager - dossier Users |
| Q2 - Build number | 16299 | Autopsy - Keyword Search CurrentBuildNumber |
| Q3 - Hostname | TOTALLYNOTAHACK | Autopsy - Operating System Information |

| Q4 - Application messagerie | Skype | Autopsy - Installed Programs |
