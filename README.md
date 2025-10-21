# Projet sécurité embarquée

Le but du projet est de réaliser une attaque pratique contre un Arduino Uno qui contient une phrase secrète (hash et salt associé). 

## Comment faire pour réaliser cette attaque?

Il existe plusieurs techniques qui vont permettre de pouvoir retouver cette phrase secrète. Mais dans le cas présent, les 2 types d'attaques principales sont:

- Single power analysis:
  
  Une attaque par canal auxiliaire qui exploite une seule trace de consommation électrique mesurée pendant l’exécution d’un algorithme sur un microcontrôleur.
  En observant la forme temporelle de cette trace, on peut identifier quelles instructions ou quelles opérations ont été exécutées,
  et si ces opérations dépendent d’un secret (par ex. un bit de clé), on peut en déduire ce secret

- Fault injection:
  
  L’ensemble des techniques visant à provoquer intentionnellement un dysfonctionnement dans un matériel en le faisant fonctionner hors des conditions normales
  pour tenter d’obtenir un comportement utile (par ex. un contournement de contrôle, la fuite d’un secret, la corruption d’une vérification).
  Contrairement au side-channel analysis, on modifie l’environnement pour forcer une défaillance, pas seulement on l’observe.

Par la suite on va utiliser le concept de single power analysis

## 🚀 Fonctionnalités
- Décrire les principales fonctionnalités
- Exemple : collecte de données GPS et humidité du sol
- Exemple : affichage sur écran OLED

## ⚙️ Matériel / Technologies
- Arduino Mega
- Capteurs ultrason / GPS NEO 6M / capteurs d’humidité
- Librairies utilisées : `VarSpeedServo`, `Adafruit_SSD1306`, etc.

## 📦 Installation
1. Clone le dépôt :
   ```bash
   git clone https://github.com/ton-nom-utilisateur/ton-repo.git
