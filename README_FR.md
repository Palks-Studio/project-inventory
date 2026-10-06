<p align="center">
  <img src="docs/images/inventory.png"
       alt="Project Inventory — rapport HTML d'analyse technique généré"
       width="1200">
</p>

> 🇫🇷 Français | [🇬🇧 English](./README.md)

![License](https://img.shields.io/badge/License-Commercial-lightgreen.svg)
![Type](https://img.shields.io/badge/Type-Technical%20Analysis-151b1c?style=flat)
![Python](https://img.shields.io/badge/Python-3.11%2B-0095b1?style=flat)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20macOS-0a5645?style=flat)
![Language](https://img.shields.io/badge/Lang-FR%20%2F%20EN-0a5645?style=flat)
[![YouTube](https://img.shields.io/badge/YouTube-@Palks__Studio-FF0000?style=flat&logo=youtube&logoColor=white)](https://www.youtube.com/watch?v=Bw-s8SEV7rw)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-@Palks__Studio-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/palks-studio/)
[![Outil](https://img.shields.io/badge/Outil-Project%20Inventory-0095b1?style=flat)](https://palks-studio.com/fr/project-inventory/)

<p align="center">
  <a href="https://palks-studio.com">
    <img src="https://img.shields.io/badge/Palks%20Studio-Website-0095b1?style=for-the-badge" />
  </a>
</p>


# Project Inventory

Project Inventory est un outil d'analyse statique et de cartographie technique d'un projet logiciel, utilisable en environnement local comme sur un serveur.

Il parcourt récursivement un dossier projet afin d'identifier sa structure technique : dépendances, fichiers de configuration, infrastructure Docker, variables d'environnement, services externes et imports Python.

L'objectif est simple : pouvoir ouvrir un projet existant, y compris un projet inconnu ou hérité, et obtenir rapidement une vue exploitable de ce qu'il contient et de ce dont il dépend.

> Analyse directe depuis le système de fichiers. Aucun code source ni secret n'a besoin d'être envoyé vers un service externe.

> Ce dépôt constitue une présentation technique et une documentation du projet.  
> Il ne contient pas de code source téléchargeable ni de fichiers de production.

---

## Fonctionnalités actuelles

### Analyse récursive du projet

Project Inventory parcourt le dossier sélectionné et ses sous-dossiers afin de détecter les fichiers techniques connus.

Certains répertoires générés ou non pertinents sont ignorés, notamment :

- `.git`  
- `.idea`  
- `.vscode`  
- `__pycache__`  
- `node_modules`  
- `vendor`  
- `.venv`  
- `venv`  
- `dist`  
- `build`  
- `coverage`

La détection fonctionne à la fois par nom de fichier connu et, lorsque nécessaire, par extension, notamment pour les fichiers `.csproj`.

---

## Dépendances

Project Inventory détecte actuellement les dépendances de 10 écosystèmes.

### Python

Formats pris en charge :

- `requirements.txt`  
- `pyproject.toml`  
- `poetry.lock`  
- `Pipfile`  
- `Pipfile.lock`

### Node.js

Formats pris en charge :

- `package.json`  
- `package-lock.json`  
- `yarn.lock`  
- `pnpm-lock.yaml`

### PHP

Formats pris en charge :

- `composer.json`  
- `composer.lock`

### Go

Formats pris en charge :

- `go.mod`  
- `go.sum`

### Rust

Formats pris en charge :

- `Cargo.toml`  
- `Cargo.lock`

### Ruby

Formats pris en charge :

- `Gemfile`  
- `Gemfile.lock`

### Java / JVM

Formats pris en charge :

- `pom.xml`  
- `build.gradle`  
- `build.gradle.kts`

### .NET

Formats pris en charge :

- `*.csproj`

### Dart / Flutter

Formats pris en charge :

- `pubspec.yaml`  
- `pubspec.lock`

### Elixir

Formats pris en charge :

- `mix.exs`  
- `mix.lock`

---

## Consolidation des dépendances

Les dépendances provenant de plusieurs fichiers sont regroupées par écosystème et par package.

Par exemple, une dépendance présente à la fois dans un manifeste et dans son lock file n'est affichée qu'une seule fois, avec ses différentes sources :

```text
dio
    pubspec.lock         ==5.7.0 [verrouillée / locked]
    pubspec.yaml         ^5.7.0
```

Une dépendance découverte uniquement dans un lock file est également conservée.

Cela permet notamment de faire apparaître des dépendances transitives qui ne sont pas directement déclarées dans le manifeste principal.

Les packages portant le même nom dans deux écosystèmes différents restent distincts.

---

## Sous-projets

Project Inventory identifie les différents sous-projets techniques présents dans une même arborescence à partir des manifestes et fichiers de configuration détectés.

Chaque sous-projet peut être associé à un ou plusieurs écosystèmes ainsi qu'aux fichiers techniques ayant permis son identification.

Cela permet notamment de distinguer, dans un même dépôt, un backend Python, un frontend Node.js ou d'autres composants disposant de leurs propres manifestes.

---

## Infrastructure Docker

### Docker Compose

Project Inventory détecte actuellement les services déclarés dans :

- `docker-compose.yml`  
- `docker-compose.yaml`  
- `compose.yml`  
- `compose.yaml`

Pour chaque service détecté, Project Inventory peut notamment extraire :

- l'image utilisée  
- la configuration de build  
- les ports  
- les fichiers d'environnement  
- les volumes  
- les dépendances entre services  
- les variables d'environnement déclarées, sans exposer leurs valeurs  
- le nom du conteneur  
- la politique de redémarrage

### Composants d'infrastructure

Certaines images Docker détectées peuvent être rapprochées de composants d'infrastructure connus, notamment :

- Redis  
- PostgreSQL  
- MySQL  
- MariaDB  
- MongoDB  
- RabbitMQ

Ces signatures enrichissent les services déjà découverts. Elles ne remplacent pas l'analyse dynamique des fichiers Docker Compose.

### Relations entre composants

Project Inventory construit une cartographie des relations observables entre les composants détectés dans le projet.

Les relations actuellement représentées comprennent notamment :

- les dépendances déclarées par les sous-projets (`declares`)  
- les dépendances entre services Docker Compose (`depends_on`)  
- les relations de build entre services Docker Compose et sous-projets (`builds`)  
- l'utilisation de fichiers d'environnement par les services (`uses_env_file`)  
- les montages de volumes Docker Compose (`mounts`)  
- l'utilisation de services externes par les sous-projets (`uses_external_service`)  
- les relations avec des composants d'infrastructure lorsqu'elles peuvent être établies à partir des données détectées

Chaque relation conserve son type, les types des composants source et cible ainsi que les preuves ayant permis de l'établir.

Project Inventory ne crée pas de relation lorsqu'elle ne peut pas être établie à partir des fichiers, déclarations ou éléments effectivement observés.

### Dockerfile

L'analyse d'un `Dockerfile` permet actuellement d'extraire :

- les images de base `FROM`  
- les répertoires `WORKDIR`  
- les ports `EXPOSE`  
- les commandes `CMD`  
- les points d'entrée `ENTRYPOINT`

Exemple :

```text
Image de base / Base image : python:3.12-slim
Répertoire de travail / Working directory : /app
Port exposé / Exposed port : 5000
Commande / Command : ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "app:app"]
```

---

## Variables d'environnement

Les fichiers suivants peuvent être analysés :

- `.env`  
- `.env.example`  
- `.env.sample`

Project Inventory extrait uniquement les noms des variables.

Les valeurs ne sont pas affichées afin d'éviter d'exposer des clés API, mots de passe ou autres secrets.

Exemple :

```text
OPENAI_API_KEY
ENABLE_PERSISTENCE
MEMORY_MAX_TURNS
STRICT_MODE
```

---

## Services externes

Project Inventory peut rapprocher les dépendances et les variables d'environnement de signatures de services connus.

Les signatures actuellement intégrées comprennent notamment :

- OpenAI  
- Stripe  
- AWS  
- SendGrid  
- Twilio  
- Sentry

Une détection peut être accompagnée de plusieurs preuves :

```text
OpenAI
    package    openai
    variable   OPENAI_API_KEY
```

Les signatures servent à enrichir les éléments découverts par le scanner. Elles ne remplacent pas la découverte dynamique des packages et variables.

Un package inconnu du catalogue de signatures reste donc visible dans l'inventaire.

---

## Analyse des imports Python

Les fichiers source Python sont analysés avec l'AST Python.

Les imports sont classés en trois catégories :

- bibliothèque standard  
- dépendances externes  
- modules internes au projet

Exemple :

```text
STANDARD PYTHON
  os
  json
  datetime

EXTERNES / EXTERNAL
  flask
  openai
  dotenv

INTERNES / INTERNAL
  main
  storage
```

---

## Référencement des dépendances Python

Project Inventory rapproche actuellement les dépendances Python déclarées des références trouvées dans le code source et, lorsque pertinent, dans Docker.

Exemple :

```text
flask
    RÉFÉRENCÉE / REFERENCED
    import flask

gunicorn
    RÉFÉRENCÉE / REFERENCED
    Docker ["gunicorn", "-w", "2", ...]
```

L'outil parle volontairement de dépendance **référencée** ou de **référence non détectée**.

L'absence de référence statique ne signifie pas nécessairement qu'une dépendance est inutilisée à l'exécution.

---

## Principe de détection

Project Inventory distingue deux mécanismes.

### Découverte

Les fichiers techniques indiquent au scanner où chercher et comment interpréter leur contenu.

Le contenu lui-même est découvert dynamiquement.

Par exemple, Project Inventory n'a pas besoin de connaître à l'avance tous les packages Python, Node.js, PHP, Go ou Rust existants pour les inventorier.

### Enrichissement

Certaines signatures permettent ensuite d'interpréter les éléments découverts.

Par exemple :

```text
OPENAI_API_KEY -> OpenAI
STRIPE_SECRET_KEY -> Stripe
SENTRY_DSN -> Sentry
```

Le principe est donc :

> Indiquer au scanner où et comment chercher, sans lui imposer à l'avance ce qu'il doit trouver.

---

## Installation

Project Inventory nécessite Python 3.11 ou une version ultérieure.

Pour les instructions complètes d'installation et les commandes spécifiques à chaque plateforme, consultez `INSTALL_FR.md`.

---

## Utilisation actuelle

Depuis la racine de Project Inventory :

```bash
python app.py
```

L'application demande ensuite le chemin du projet à analyser :

```text
Chemin du projet à analyser :
```

Les résultats de l'analyse sont affichés dans le terminal et exportés automatiquement dans le dossier `output`.

Les formats actuellement générés sont :

- `project-inventory.json`  
- `project-inventory.html`  
- un ensemble de rapports CSV dans le dossier `output/csv`

Les exports CSV comprennent :

- `dependencies.csv`  
- `dependency_usage.csv`  
- `dockerfiles.csv`  
- `environment_variables.csv`  
- `external_services.csv`  
- `imports.csv`  
- `infrastructure_components.csv`  
- `service_relationships.csv`  
- `services.csv`  
- `source_files.csv`  
- `subprojects.csv`  
- `technical_files.csv`

Le rapport HTML fournit une vue structurée et lisible de l'inventaire.

Le fichier JSON conserve les données détaillées dans un format exploitable par d'autres outils.

Les fichiers CSV fournissent des exports spécialisés pouvant être ouverts dans un tableur ou utilisés pour d'autres traitements.

---

## Structure du projet

```text
project-inventory/
├── analyzers/
│   ├── __init__.py
│   │   → (FR) Initialise le module d’analyse.
│   │   → (EN) Initializes the analysis module.
│   │
│   ├── dependencies.py
│   │   → (FR) Consolide les dépendances détectées et analyse leurs références dans le projet.
│   │   → (EN) Consolidates detected dependencies and analyzes their references within the project.
│   │
│   └── relationships.py
│       → (FR) Analyse les relations entre les services et composants détectés dans le projet.
│       → (EN) Analyzes relationships between services and components detected within the project.
│
├── detectors/
│   ├── __init__.py
│   │   → (FR) Initialise le module des détecteurs.
│   │   → (EN) Initializes the detector module.
│   │
│   ├── environment.py
│   │   → (FR) Détecte les noms des variables d’environnement sans exposer leurs valeurs.
│   │   → (EN) Detects environment variable names without exposing their values.
│   │
│   ├── infrastructure.py
│   │   → (FR) Analyse les fichiers d’infrastructure, notamment Docker Compose et Dockerfile.
│   │   → (EN) Analyzes infrastructure files, including Docker Compose and Dockerfile.
│   │
│   ├── languages.py
│   │   → (FR) Analyse les fichiers source et classe les imports Python.
│   │   → (EN) Analyzes source files and classifies Python imports.
│   │
│   ├── packages.py
│   │   → (FR) Extrait les dépendances depuis les manifestes et lock files des écosystèmes pris en charge.
│   │   → (EN) Extracts dependencies from manifests and lock files across supported ecosystems.
│   │
│   ├── projects.py
│   │   → (FR) Identifie les sous-projets techniques à partir des manifestes et fichiers de configuration détectés.
│   │   → (EN) Identifies technical subprojects from detected manifests and configuration files.
│   │
│   └── services.py
│       → (FR) Identifie les services externes à partir des packages et variables d’environnement détectés.
│       → (EN) Identifies external services from detected packages and environment variables.
│
├── docs/
│   ├── CHANGELOG.md
│   │   → (FR) Historique des modifications et évolutions du projet en anglais.
│   │   → (EN) Tracks project changes and development history in English.
│   │
│   ├── CHANGELOG_FR.md
│   │   → (FR) Historique des modifications et évolutions du projet en français.
│   │   → (EN) Tracks project changes and development history in French.
│   │
│   ├── INSTALL.md
│   │   → (FR) Instructions d’installation et d’utilisation en anglais.
│   │   → (EN) Installation and usage instructions in English.
│   │
│   ├── INSTALL_FR.md
│   │   → (FR) Instructions d’installation et d’utilisation en français.
│   │   → (EN) Installation and usage instructions in French.
│   │
│   ├── README.md
│   │   → (FR) Documentation générale et présentation du projet en anglais.
│   │   → (EN) General project documentation and overview in English.
│   │
│   └── README_FR.md
│       → (FR) Documentation générale et présentation du projet en français.
│       → (EN) General project documentation and overview in French.
│
├── exporters/
│   ├── __init__.py
│   │   → (FR) Initialise le module d’export des rapports.
│   │   → (EN) Initializes the report export module.
│   │
│   ├── csv_export.py
│   │   → (FR) Génère les rapports d’inventaire au format CSV.
│   │   → (EN) Generates inventory reports in CSV format.
│   │
│   ├── html_export.py
│   │   → (FR) Génère les rapports d’inventaire au format HTML.
│   │   → (EN) Generates inventory reports in HTML format.
│   │
│   └── json_export.py
│       → (FR) Génère les rapports d’inventaire au format JSON.
│       → (EN) Generates inventory reports in JSON format.
│
├── output/
│   └── .gitkeep
│       → (FR) Conserve le dossier de sortie dans Git lorsqu’il est vide.
│       → (EN) Keeps the output directory tracked by Git when empty.
│
├── scanner/
│   ├── __init__.py
│   │   → (FR) Initialise le module principal de scan du projet.
│   │   → (EN) Initializes the main project scanning module.
│   │
│   ├── filesystem.py
│   │   → (FR) Parcourt récursivement le système de fichiers et détecte les fichiers techniques à analyser.
│   │   → (EN) Recursively scans the filesystem and detects technical files to analyze.
│   │
│   └── project_scanner.py
│       → (FR) Orchestre l’analyse complète du projet et centralise les résultats des différents détecteurs.
│       → (EN) Orchestrates the complete project analysis and centralizes results from the different detectors.
│
├── LICENCE.md
│   → (FR) Définit les conditions de licence et les droits d’utilisation de Project Inventory en français.
│   → (EN) Defines Project Inventory licensing terms and usage rights in French.
│
├── LICENSE.md
│   → (FR) Définit les conditions de licence et les droits d’utilisation de Project Inventory en anglais.
│   → (EN) Defines Project Inventory licensing terms and usage rights in English.
│
└── app.py
    → (FR) Point d’entrée de Project Inventory, affichage des résultats dans le terminal et génération des rapports.
    → (EN) Project Inventory entry point, terminal output and report generation.
```

---

## État du produit

Project Inventory est opérationnel pour l'analyse statique et la cartographie technique des projets pris en charge, en environnement local comme sur un serveur disposant d'un accès direct aux fichiers du projet.

Les fonctionnalités disponibles comprennent notamment :

- l'analyse récursive des fichiers techniques  
- la détection et la consolidation des dépendances sur les écosystèmes pris en charge  
- l'identification des sous-projets  
- l'analyse de Docker Compose et des Dockerfiles  
- l'identification de composants d'infrastructure  
- l'extraction sécurisée des noms de variables d'environnement  
- l'identification de services externes à partir de signaux observés  
- l'analyse et la classification des imports Python  
- le rapprochement entre dépendances Python et références observées  
- la cartographie des relations entre composants  
- l'affichage des résultats dans le terminal  
- la génération de rapports HTML, JSON et CSV

L'analyse est statique : Project Inventory n'exécute pas le projet analysé et ne modifie pas ses fichiers.

---

## Philosophie

Project Inventory n'a pas vocation à exécuter le projet analysé.

Il construit une cartographie à partir des fichiers, déclarations, configurations et références qu'il peut observer statiquement.

Le rapport doit donc distinguer ce qui est :

- découvert  
- déclaré  
- verrouillé  
- référencé  
- interprété à partir d'une signature

sans transformer une observation statique en affirmation sur le comportement réel du logiciel.

---

© Palks Studio — voir LICENSE.md  
- https://palks-studio.com
