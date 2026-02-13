# Rapport d'Intervention - Incident Response Report

**Case ID**: DFIR-2026-001
**Investigateur**: Thomas FERET
**Date**: 2026-02-13
**Classification**: Interne

---

## Résumé Exécutif

Une machine Windows XP SP3 a été compromise par le malware BlackEnergy v2. L'attaquant a déployé un rootkit kernel via un driver malveillant (`str.sys` / `suzmkbtuoyeki.sys`) et injecté du code dans le processus système `svchost.exe` via une DLL malveillante (`msxml3r.dll`). Le mécanisme de persistence repose sur un service Windows enregistré sous un nom aléatoire (`hvklkzdz`) se lançant automatiquement au démarrage.

---

## 1. Vue d'ensemble de l'incident

### 1.1 Chronologie

| Heure (UTC) | Evenement |
|-------------|-----------|
| 2023-02-13 17:54:15 | Demarrage normal de la machine |
| 2023-02-13 18:25:15 | Lancement de taskmgr.exe (investigation utilisateur ?) |
| 2023-02-13 18:25:26 | Execution de rootkit.exe - debut de la compromission |
| 2023-02-13 18:25:26 | Installation du service hvklkzdz dans le registre |
| 2023-02-13 18:25:26 | Lancement de cmd.exe par rootkit.exe |
| 2023-02-13 18:25:26 | Injection de msxml3r.dll dans svchost.exe (PID 880) |
| 2023-02-13 18:29:08 | Lancement de DumpIt.exe - capture memoire par l'equipe IR |
| 2023-02-13 18:29:11 | Acquisition du dump memoire |

### 1.2 Systemes affectes

| Propriete | Valeur |
|-----------|--------|
| OS | Windows XP SP3 (build 2600) |
| Architecture | 32 bits (x86) |
| Utilisateur | CyberDefenders |
| Profil Volatility | WinXPSP2x86 |

### 1.3 Vecteur d'attaque

Execution directe de `rootkit.exe` depuis le bureau de l'utilisateur. Le vecteur initial (phishing, download malveillant) n'est pas determinable depuis le dump memoire seul.

---

## 2. Methodologie d'investigation

### 2.1 Acquisition des preuves

| Preuve | Taille | Outil | Hash |
|--------|--------|-------|------|
| CYBERDEF-567078-20230213-171333.raw | 2.1 Go | DumpIt | A verifier |

### 2.2 Outils utilises

| Outil | Version | Usage |
|-------|---------|-------|
| Volatility 3 | 2.27.0 | Analyse memoire |
| Python | 3.12 | Environnement d'execution |

### 2.3 Plugins Volatility executes

| Plugin | Objectif | Fichier de sortie |
|--------|----------|-------------------|
| windows.info | Identification du systeme | windows_info.txt |
| windows.pslist | Liste des processus | pslist.txt |
| windows.malfind | Detection injection code | malfind.txt |
| windows.dlllist | DLLs chargees par svchost | dlllist_880.txt |
| windows.ldrmodules | Modules non declares | ldrmodules_880.txt |
| windows.handles | Fichiers ouverts par svchost | handles_880.txt |
| windows.cmdline | Arguments des processus | cmdline.txt |
| windows.driverscan | Drivers charges en memoire | driverscan.txt |
| windows.registry.hivelist | Liste des ruches registre | hivelist.txt |
| windows.registry.printkey | Cles de persistence | registry_hvklkzdz.txt |

---

## 3. Findings

### 3.1 Processus suspects

**rootkit.exe (PID 964)**
- Parent : explorer.exe (PID 1484) - lance depuis l'interface utilisateur
- A lance cmd.exe (PID 1960) immediatement apres son execution
- Termine a 18:25:26 UTC apres avoir accompli sa mission

**svchost.exe (PID 880)**
- Processus legitime Windows compromis par injection de code
- Contient un MZ header a l'adresse 0x980000 (detecte par malfind)
- DLL msxml3r.dll injectee a l'adresse 0x9a0000
- Handle ouvert sur \WINDOWS\system32\drivers\str.sys

### 3.2 Mecanisme de persistence

BlackEnergy installe un service Windows au demarrage :

| Propriete | Valeur |
|-----------|--------|
| Nom du service | hvklkzdz (nom aleatoire) |
| ImagePath | \??\C:\WINDOWS\system32\drivers\suzmkbtuoyeki.sys |
| Type | 1 (Kernel Driver) |
| Start | 2 (Automatique au demarrage) |
| Date installation | 2023-02-13 18:25:26 UTC |

### 3.3 Fichiers malveillants

| Fichier | Chemin | Role |
|---------|--------|------|
| rootkit.exe | Bureau utilisateur | Dropper - installe le rootkit |
| str.sys | C:\WINDOWS\system32\drivers\ | Driver rootkit BlackEnergy |
| suzmkbtuoyeki.sys | C:\WINDOWS\system32\drivers\ | Alias du driver (nom aleatoire) |
| msxml3r.dll | C:\WINDOWS\system32\ | DLL malveillante injectee dans svchost |
| hvklkzdz | Service Windows | Persistence au demarrage |

### 3.4 Techniques d'evasion

- **Nom aleatoire** : driver renomme en `hvklkzdz` et `suzmkbtuoyeki.sys`
- **Process injection** : code injecte dans svchost.exe (processus legitime)
- **DLL masquee** : msxml3r.dll imite le nom legitime msxml3.dll
- **Kernel rootkit** : driver charge au niveau kernel pour masquer sa presence

---

## 4. Indicateurs de Compromission (IOCs)

### 4.1 Fichiers

| Fichier | Chemin | Confiance |
|---------|--------|-----------|
| rootkit.exe | Bureau CyberDefenders | Haute |
| str.sys | C:\WINDOWS\system32\drivers\ | Haute |
| suzmkbtuoyeki.sys | C:\WINDOWS\system32\drivers\ | Haute |
| msxml3r.dll | C:\WINDOWS\system32\ | Haute |

### 4.2 Registre

| Cle | Valeur | Confiance |
|-----|--------|-----------|
| SYSTEM\ControlSet001\Services\hvklkzdz | ImagePath = suzmkbtuoyeki.sys | Haute |
| SYSTEM\ControlSet001\Services\hvklkzdz | Start = 2 (auto) | Haute |

### 4.3 Memoire

| Indicateur | Valeur | Confiance |
|------------|--------|-----------|
| Adresse injection | 0x980000 dans svchost.exe (PID 880) | Haute |
| Base DLL injectee | 0x9a0000 | Haute |
| Driver malveillant | hvklkzdz a 0xb912c000 | Haute |

---

## 5. Mapping MITRE ATT&CK

| Tactique | Technique | Sous-technique | Evidence | Confiance |
|----------|-----------|----------------|----------|-----------|
| Execution | T1204 - User Execution | T1204.002 - Malicious File | rootkit.exe execute depuis le bureau | Haute |
| Persistence | T1543 - Create or Modify System Process | T1543.003 - Windows Service | Service hvklkzdz, Start=2 | Haute |
| Privilege Escalation | T1543.003 | Windows Service | Driver kernel installe | Haute |
| Defense Evasion | T1055 - Process Injection | T1055.001 - DLL Injection | msxml3r.dll injectee dans svchost.exe | Haute |
| Defense Evasion | T1014 - Rootkit | - | Driver hvklkzdz / str.sys en kernel | Haute |
| Defense Evasion | T1036 - Masquerading | T1036.005 - Match Legitimate Name | msxml3r.dll imite msxml3.dll | Haute |
| Execution | T1059 - Command Interpreter | T1059.003 - cmd.exe | cmd.exe lance par rootkit.exe | Haute |

---

## 6. Analyse de la cause racine

L'utilisateur a execute `rootkit.exe` directement depuis son bureau. Ce fichier est le dropper de BlackEnergy v2 qui a :

1. Installe un driver kernel (`str.sys`) renomme aleatoirement
2. Enregistre ce driver comme service Windows avec demarrage automatique
3. Injecte une DLL malveillante (`msxml3r.dll`) dans `svchost.exe`
4. Lance `cmd.exe` pour executer des commandes supplementaires

La persistence est assurée par le service kernel qui se recharge a chaque demarrage.

---

## 7. Recommandations

### 7.1 Actions immediates

- Isoler la machine du reseau
- Ne pas redemarrer (risque de perte de preuves volatiles)
- Capturer une image disque complete avant remediation
- Bloquer les hashes des fichiers malveillants sur les endpoints

### 7.2 Court terme

- Reinstaller le systeme d'exploitation (compromission kernel)
- Analyser les autres machines du reseau pour propagation
- Rechercher rootkit.exe dans les logs de securite
- Verifier les comptes utilisateurs pour exfiltration de donnees

### 7.3 Long terme

- Mettre a jour Windows XP vers un OS supporte
- Deployer un EDR (Endpoint Detection and Response)
- Former les utilisateurs a ne pas executer de fichiers inconnus
- Mettre en place un principe de moindre privilege
- Activer la journalisation des evenements Windows (Sysmon)

---

## 8. Conclusion

La machine analysee a ete compromise par BlackEnergy v2, un malware sophistique utilise dans des attaques contre des infrastructures critiques. L'analyse memoire a permis d'identifier l'ensemble de la chaine d'attaque : execution initiale, persistence via service kernel, injection de code dans un processus legitime et techniques d'evasion avancees.

La reinstallation complete du systeme est obligatoire. Une compromission au niveau kernel ne peut pas etre remedied par un simple scan antivirus.

---

## Annexes

### A. Journal des preuves

| ID | Fichier | Taille | Acquisition | Analyste |
|----|---------|--------|-------------|---------|
| E01 | CYBERDEF-567078-20230213-171333.raw | 2.1 Go | DumpIt | Thomas FERET |

### B. Versions des outils

| Outil | Version |
|-------|---------|
| Volatility 3 | 2.27.0 |
| Python | 3.12 |

### C. Commandes executees
```bash
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.info
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.pslist
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.malfind
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.dlllist --pid 880
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.ldrmodules --pid 880
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.handles --pid 880
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.cmdline
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.driverscan
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.registry.hivelist
vol -f data/memory/CYBERDEF-567078-20230213-171333.raw windows.registry.printkey --offset 0xe1035b60 --key "ControlSet001\Services\hvklkzdz"
```
