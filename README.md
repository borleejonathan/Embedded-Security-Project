# Projet sécurité embarquée

![carte arduino](img/arduino.avif)

Le but du projet est de réaliser une attaque pratique contre un Arduino Uno qui contient une phrase secrète (hash et salt associé). 

## Comment faire pour réaliser cette attaque?

Il existe plusieurs techniques qui vont permettre de pouvoir retrouver cette phrase secrète. Mais dans le cas présent, les 2 types d'attaques principales sont:

- Single power analysis (SPA):
  
  Une attaque par canal auxiliaire qui exploite une seule trace de consommation électrique mesurée pendant l’exécution d’un algorithme sur un microcontrôleur.
  En observant la forme temporelle de cette trace, on peut identifier quelles instructions ou quelles opérations ont été exécutées,
  et si ces opérations dépendent d’un secret (par ex. un bit de clé), on peut en déduire ce secret

- Fault injection:
  
  L’ensemble des techniques visant à provoquer intentionnellement un dysfonctionnement dans un matériel en le faisant fonctionner hors des conditions normales
  pour tenter d’obtenir un comportement utile (par ex. un contournement de contrôle, la fuite d’un secret, la corruption d’une vérification).
  Contrairement au side-channel analysis, on modifie l’environnement pour forcer une défaillance, pas seulement on l’observe.

Par la suite on va utiliser le concept de single power analysis
## ChipWhisperer Nano

![chipwhisperer](img/chip.jfif)

L'utilisation du ChipWhisperer Nano est crucial et permet d’observer la consommation du ATmega328P en capturant la courbe de courant pendant la vérification du mot de passe afin d’y repérer des motifs tels que des boucles, des lectures mémoire ou des opérations dépendantes des données. 
Il sert aussi à analyser statistiquement ces traces : on automatise la corrélation entre des hypothèses sur des octets ou des bits du secret et des milliers de traces pour extraire des clés ou valeurs cachées. 

## Attack tree

Le diagramme ci‑dessous présente l’attack tree de notre vault:

![attack tree](img/Diagramme.drawio.png)

## Flash the firmware on the target

Dans un premier temps il va falloir implémenter le firmware dans l'arduino uno

```avrdude -v -patmega328p -carduino -P/dev/ttyACM0 -b115200 -Uflash:w:firmware.elf```

## Schématique du montage

Le schéma représente le montage qui va permettre d'analyser la puissance pour bypasser le password

![schematic](img/schematic.png)

Des résistances de 100 ohm et des capacités entre 100 et 300 µF sont utilisées.

Quelques ajouts sont à effectuer pour que le montage soit complet:
- Une connexion entre la pin 2 du Chip whisperer(CW) et la masse de la breadboard
- Une connexion entre la pin 8 du CW et l'alimentation de 5V de la breadboard
- Une connexion entre la pin 10 du CW et la broche 17 de l'ATMEGA
- Une connexion entre la pin 12 du CW et la broche 16 de l'ATMEGA

## Attack(s) description and implementation

### Capture des traces d’exécution

Objectif : Capturer des traces de consommation pendant l’exécution de la routine de vérification du mot de passe, pour permettre SPA.

Procédure:

1. Envoyer une entrée : Cas « incorrect » : envoyer une chaîne aléatoire (ex. AAAAA…) et capturer la trace.
2. Répéter pour plusieurs entrées distinctes pour voir la variabilité.
3. Si disponible, obtenir une trace « success » : si on connais le mot de passe ou si on a un moyen de le forcer, capturer au moins une trace montrant le parcours de succès.

### Analyse SPA 

Objectif : Utiliser Simple Power Analysis pour repérer la routine de comparaison, déterminer si elle fait des opérations dépendantes des données et extraire le mot de passe octet par octet.

1. Visualisation: Ouvrir les traces capturées sur la fenêtre correspondant à la comparaison (utiliser trigger comme repère).
2. Identifier signatures: Rechercher séquences répétitives : séries d’impulsions ou motifs qui se répètent N fois (N ≈ longueur du mot de passe).
- Rechercher différences de longueur ou d’amplitude entre « fail » et « success » (signature d’early-exit ou lecture du secret).
3. Exploitation par brute force de position: Si la routine s’arrête au premier caractère incorrect (early-exit), procéder position par position :
- Pour la position 1, envoyer différents caractères (0x00 → 0x7F) et mesurer la durée ou la longueur de la séquence dans la trace.
- Le caractère qui provoque la plus longue exécution est très probablement le bon caractère pour la position 1.
- Répéter pour la position 2, etc., jusqu’à reconstituer la chaîne complète.

### Récupération finale (extraction du hash+salt ou du mot de passe)

Objectif : Utiliser le mot de passe reconstitué (via SPA) pour obtenir la sortie finale du vault : Hash + Salt.

Si SPA a permis de reconstituer le mot de passe :
- Saisir le mot de passe reconstitué dans l’interface du vault (via terminal série).

## Vulnerability severity assessment
source: https://nvd.nist.gov/vuln-metrics/cvss/v4-calculator
CVSS v4.0 Score: 1.5 (Low)
### CVSS v4.0 Vector
```CVSS:4.0/AV:P/AC:H/AT:P/PR:N/UI:N/VC:H/VI:L/VA:N/SC:L/SI:N/SA:N/E:P/CR:H/IR:M/AR:L/MAV:P/MAC:H/MAT:P/MPR:N/MUI:N/MVC:H/MVA:L/MSC:L/MSI:L/MSA:N/S:N/AU:N/R:I```

CVSS v4.0 Score: 1.5 (Low)

### Metrics Justifications

### Base Metrics
| Metric | Value | Justification |
|--------|-------|---------------|
| Attack Vector (AV) | Physical (P) | Accès physique requis pour connecter sondes / programmer / glitcher. |
| Attack Complexity (AC) | High (H) | Compétences et calibrage nécessaires (synchronisation, analyse, matériel). |
| Attack Requirements (AT) | Present (P) | Matériel spécialisé nécessaire (ChipWhisperer, sonde, câblage). |
| Privileges Required (PR) | None (N) | Aucun privilège logiciel nécessaire. |
| User Interaction (UI) | None (N) | Pas besoin d’un utilisateur complice. |
| Confidentiality (VC) | High (H) | Divulgation possible de la keyphrase / hash+salt. |
| Integrity (VI) | Low (L) | L’exploitation vise la divulgation ; modification firmware possible mais pas prioritaire. |
| Availability (VA) | None (N) | Attaque ne vise pas à interrompre le service. |
| Confidentiality (SC) | Low (L) | Divulgation du secret pourrait faciliter attaques secondaires, mais scope limité. |
| Integrity (SI) | Low (L) | Possibilité limitée de falsification d’autres systèmes via ce secret. |
| Availability (SA) | None (N) | Pas d’impact attendu sur la disponibilité de systèmes tiers. |

### Supplemental Metrics

| Metric | Value | Justification |
|--------|-------|---------------|
| Safety (S) | None (N) | Pas d’impact physique / risque pour les personnes. |
| Automatable (AU) | No (N) | Besoin d’intervention humaine et d’ajustements, pas automatisable. |
| Recovery (R) | Irrecoverable (I) | Si la keyphrase est divulguée, elle doit être régénérée — impact permanent. |
| Value Density (V) | High (H) | Secret unique et à forte valeur (accès au vault / preuve d’attaque). |
| Vulnerability Response Effort (RE) | Moderate (M) | Corriger nécessite modification firmware + ajout hardware. |

### Environmental (Modified Base Metrics)

| Metric | Value | Justification |
|--------|-------|---------------|
| Attack Vector (MAV) | Physical (P) | Même contraintes locales que pour le cas de base. |
| Attack Complexity (MAC) | High (H) | Attaque complexe et nécessite calibration / compétence. |
| Attack Requirements (MAT) | Present (P) | Matériel spécialisé toujours nécessaire. |
| Privileges Required (MPR) | None (N) | Pas de privilèges logiciels requis. |
| User Interaction (MUI) | None (N) | Pas besoin d’utilisateur pour exploiter. |
| Confidentiality (MVC) | High (H) | Confidentialité critique dans l’environnement. |
| Integrity (MVI) | Not Defined (X) | Intégrité importante (firmware) mais secondaire. |
| Availability (MVA) | Low (L) | Disponibilité souhaitée mais impact limité. |
| Confidentiality (MSC) | Low (L) | Impacts secondaires limités. |
| Integrity (MSI) | Low (L) | Impacts secondaires limités. |
| Availability (MSA) | Negligible (N) | Pas d’impact secondaire. |

### Environmental (Security Requirements)

| Metric | Value | Justification |
|--------|-------|---------------|
| Confidentiality Requirements (CR) | High (H) | L’objectif du vault est de protéger la confidentialité du secret. |
| Integrity Requirements (IR) | Medium (M) | Intégrité importante mais moins critique que confidentialité. |
| Availability Requirements (AR) | Low (L) | Disponibilité non critique par rapport à la confidentialité. |

### Threat Metrics

| Metric | Value | Justification |
|--------|-------|---------------|
| Exploit Maturity (E) | Proof-of-Concept (POC) | Exploitation démontrable en labo avec équipement et scripts, mais pas triviale ni largement diffusée. |


## Contre mesures

- Exécution en temps constant (pas d’early exit)
  
  Principe : éviter que la durée ou la séquence d’instructions dépende des octets testés (ex: exit anticipé dès qu’un caractère ne correspond pas).
  Ainsi aucune différence visible dans la trace de puissance ou le timing ne renseigne l’attaquant.

- Masquage / opérations indépendantes des bits sensibles
  
  Principe : rendre la consommation instantanée indépendante des valeurs traitées en utilisant des transformations aléatoires (masks)
  qui annulent l’effet du secret sur la consommation.

- Ajout de bruit (alimentation filtrée, condensateurs, blindage EM) et random delays

  Principe : augmenter le rapport signal/bruit pour rendre la lecture SPA/DPA plus difficile  (via le matériel et via techniques logicielles).

- Détection d’analyse (watchdog / brown-out / détection alim perturbée)

  Principe : détecter des conditions anormales (chute de tension, glitch, reset fréquent) et réagir (bloquer, effacer secret, entrer en mode sécurisé)
  pour empêcher ou rendre l’attaque destructrice.
