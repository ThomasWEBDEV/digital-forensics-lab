# Digital Forensics Lab - Analyse BlackEnergy

**Investigateur**: Thomas FERET (ThomasWEBDEV)
**Statut**: En cours
**Date de debut**: Fevrier 2026

## Contexte du projet

Ce projet simule une investigation forensics reelle a partir d'un exercice propose par CyberDefenders.
L'objectif est de demonstrer des competences DFIR (Digital Forensics & Incident Response) dans le cadre d'une recherche d'alternance en cybersecurite.

## Scenario

Une entreprise multinationale a ete victime d'une cyberattaque. Les attaquants ont installe le malware **BlackEnergy v2** sur une machine Windows XP. L'equipe de securite a capture la RAM de la machine au moment de la compromission.

Notre role : analyser cette capture memoire pour reconstituer l'attaque.

## Source de l'exercice

**Plateforme**: CyberDefenders (https://cyberdefenders.org)
**Challenge**: BlackEnergy
**Categorie**: Endpoint Forensics / Memory Analysis
**Difficulte**: Moyen
**Note**: 4.5/5

## Outils utilises

| Outil | Version | Role |
|-------|---------|------|
| Volatility 3 | 2.27.0 | Analyse du dump memoire |
| Python 3 | 3.12 | Environnement d'execution |

## Methodologie

On suit la methodologie **SANS PICERL** :

1. **Preparation** - Installation des outils, telechargement du dump memoire
2. **Identification** - Analyse memoire avec Volatility (processus, reseau, malware)
3. **Containment** - Extraction des IOCs (Indicators of Compromise)
4. **Eradication** - Reconstitution de l'attaque, documentation des TTPs
5. **Recovery** - Recommandations de remediation
6. **Lessons Learned** - Rapports techniques professionnels

## Avancement

- [x] Installation de Volatility 3
- [x] Telechargement du dump memoire (2.1 Go)
- [x] Identification du systeme (Windows XP SP3 32bit)
- [ ] Analyse des processus (windows.pslist)
- [ ] Analyse reseau (windows.netscan)
- [ ] Detection code malveillant (windows.malfind)
- [ ] Extraction des IOCs
- [ ] Mapping MITRE ATT&CK
- [ ] Rapports finaux

## Structure du projet
```
digital-forensics-lab/
├── README.md                 # Vue d'ensemble du projet
├── .gitignore                # Exclusion des fichiers lourds
├── data/                     # Donnees brutes (non versionnees)
│   ├── memory/               # Dump memoire (.raw)
│   └── disk/                 # Images disque
├── analysis/                 # Resultats d'analyse
├── reports/                  # Rapports techniques
├── scripts/                  # Scripts d'automatisation
├── screenshots/              # Captures d'ecran
└── config/                   # Configuration des outils
```
