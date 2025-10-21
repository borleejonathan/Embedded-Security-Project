# Projet sécurité embarquée

![carte arduino](img/arduino.avif)

Le but du projet est de réaliser une attaque pratique contre un Arduino Uno qui contient une phrase secrète (hash et salt associé). 

## Comment faire pour réaliser cette attaque?

Il existe plusieurs techniques qui vont permettre de pouvoir retrouver cette phrase secrète. Mais dans le cas présent, les 2 types d'attaques principales sont:

- Single power analysis:
  
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

L'utilisation du ChipWhisperer Nano est crucial et permet de permet d’observer la consommation du ATmega328P en capturant la courbe de courant pendant la vérification du mot de passe afin d’y repérer des motifs tels que des boucles, des lectures mémoire ou des opérations dépendantes des données. 
Il sert aussi à analyser statistiquement ces traces : on automatise la corrélation entre des hypothèses sur des octets ou des bits du secret et des milliers de traces pour extraire des clés ou valeurs cachées. 

## Attack tree

Le diagramme ci‑dessous présente l’attack tree de notre coffre:

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


## Vulnerability severity assessment

Selon l’évaluation CVSS 4.0, la vulnérabilité obtient un score de 3.7 (Low).
Bien que l’attaque permette la divulgation complète du secret stocké, elle requiert un accès physique au dispositif, un équipement spécialisé (ChipWhisperer, oscilloscope, sonde de courant) et des compétences avancées en analyse de canaux auxiliaires.
Ces conditions limitent considérablement la probabilité d’exploitation réelle, d’où la sévérité faible attribuée par le score CVSS

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
