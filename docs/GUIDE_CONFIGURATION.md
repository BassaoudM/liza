# Guide Complet de Configuration Liza

> Documentation détaillée pour configurer et utiliser Liza avec Claude Code, MCP et dans vos projets.

## Table des Matières

1. [Installation et Prérequis](#installation-et-prérequis)
2. [Configuration des Agents](#configuration-des-agents)
3. [Configuration des MCP](#configuration-des-mcp)
4. [Le Contrat Comportemental](#le-contrat-comportemental)
5. [Les Skills Liza](#les-skills-liza)
6. [Personnalisation du Pipeline YAML](#personnalisation-du-pipeline-yaml)
7. [Intégration avec les Outils Existants](#intégration-avec-les-outils-existants)
8. [Utilisation en Mode Pairing](#utilisation-en-mode-pairing)
9. [Utilisation en Mode Multi-Agent](#utilisation-en-mode-multi-agent)
10. [Patterns de Workflow Recommandés](#patterns-de-workflow-recommandés)
11. [Projets Existants vs Nouveaux Projets](#projets-existants-vs-nouveaux-projets)
12. [Meilleures Pratiques et Tips](#meilleures-pratiques-et-tips)
13. [Exemples Concrets](#exemples-concrets)
14. [Monitoring et Métriques](#monitoring-et-métriques)
15. [Sécurité et Bonnes Pratiques](#sécurité-et-bonnes-pratiques)
16. [Commandes Essentielles](#commandes-essentielles)
17. [Dépannage](#dépannage)
18. [Glossaire](#glossaire)
19. [FAQ](#faq)
20. [Ressources Complémentaires](#ressources-complémentaires)

---

## Installation et Prérequis

### Prérequis Système

- **Git 2.38+** - Pour le support complet des worktrees
- **Go 1.25.5+** - Pour la compilation depuis les sources (optionnel si binaires pré-compilés)
- **Un agent CLI installé** - Claude Code, Codex, Kimi, Mistral ou Gemini

### Installation Rapide

```bash
# Installation automatique (macOS/Linux)
curl -fsSL https://raw.githubusercontent.com/liza-mas/liza/main/install.sh | bash
```

### Options d'Installation

```bash
# Version spécifique
curl -fsSL https://raw.githubusercontent.com/liza-mas/liza/main/install.sh | VERSION=v1.0.0 bash

# Depuis une branche (nécessite Go et make)
curl -fsSL https://raw.githubusercontent.com/liza-mas/liza/main/install.sh | BRANCH=main bash

# Répertoire d'installation personnalisé
curl -fsSL https://raw.githubusercontent.com/liza-mas/liza/main/install.sh | INSTALL_DIR=~/.local/bin bash

# Depuis un clone local
git clone https://github.com/liza-mas/liza.git && cd liza
make install
```

### Vérification de l'Installation

```bash
liza version
```

---

## Configuration des Agents

### Agents Supportés

| Agent | Support | Notes |
|-------|---------|-------|
| Claude Code | ✅ Complet | Référence principale |
| Codex CLI | ✅ Complet | Équivalent à Claude |
| Kimi 2.5 | ⚠️ Compatible | Réactif aux retours d'outils |
| Mistral Devstral-2 | ⚠️ Partiel | Requiert activation explicite |
| Gemini 2.5 Flash | ❌ Incompatible | Limitation architecturale |

### Configuration Initiale

```bash
# Setup global (une seule fois)
liza setup
```

### Activation par Agent

```bash
# Claude Code uniquement
liza setup --claude

# Plusieurs agents
liza setup --claude --codex --gemini --mistral

# Avec fichier AGENT_TOOLS.md personnalisé
liza setup --agent-tools ~/mon-fichier-outils.md
```

### Structure des Fichiers de Configuration

Après `liza setup`, les fichiers suivants sont créés dans `~/.liza/`:

```
~/.liza/
├── AGENT_TOOLS.md          # Configuration des outils par agent
├── COLLABORATION_CONTINUITY.md  # Contexte inter-sessions
├── contract-activation.md   # Activation du contrat par agent
├── CONTRACT_FAILURE_MODE_MAP.md # Cartographie des modes d'échec
├── CORE.md                  # Contrat principal
├── MULTI_AGENT_MODE.md      # Documentation mode multi-agent
├── PAIRING_MODE.md          # Documentation mode pairing
├── pipeline.yaml           # Pipeline de travail
├── SUBAGENT_MODE.md         # Documentation mode subagent
└── skills/                  # 20 compétences réutilisables
    ├── adr-backfill/
    ├── architecture-planning/
    ├── code-review/
    ├── debugging/
    ├── epic-writing/
    ├── user-story-writing/
    └── ... (15 autres)
```

### Configuration par Agent - Détails

#### Claude Code (Recommandé)

```bash
# Installation de Claude Code si pas encore fait
# Vérifier qu'il est dans le PATH
which claude

# Configuration Liza avec Claude
liza setup --claude
```

**Note importante**: Assurez-vous que la variable `ANTHROPIC_API_KEY` n'est pas définie par défaut dans votre shell. Claude Code utilise votre abonnement existant sans facturation API séparée.

#### Codex CLI

```bash
# Installation de Codex CLI
# Codex utilise votre abonnement OpenAI

# Configuration Liza avec Codex
liza setup --codex
```

#### Kimi (Moonshot AI)

```bash
# Installation de Kimi CLI
# Utilise l'abonnement Kimi

# Configuration Liza avec Kimi
liza setup --kimi
```

#### Mistral

```bash
# Installation de Mistral CLI
# Utilise l'abonnement Mistral

# Configuration Liza avec Mistral
liza setup --mistral
```

#### Gemini

```bash
# Installation de Gemini CLI (gemini or gcloud)
# Utilise l'abonnement Google

# Configuration Liza avec Gemini
liza setup --gemini
```

---

## Configuration des MCP

### Qu'est-ce que MCP?

MCP (Model Context Protocol) est un protocole qui permet aux agents AI d'accéder à des outils externes comme:
- IDE (JetBrains, VS Code)
- Systèmes de fichiers
- Recherche web
- Bases de données
- Et plus encore

### MCP Inclus par Défaut

Liza est livré avec une configuration MCP optimisée dans `~/.liza/AGENT_TOOLS.md`:

| MCP | Utilisation | Priorité |
|-----|-------------|----------|
| JetBrains | Opérations indexées, recherche | Haute |
| Morph-MCP | Édition, recherche sémantique | Haute |
| Perplexity | Recherche web synthèse | Haute |
| Context7 | Documentation API | Moyenne |
| Filesystem | Opérations fichiers | Moyenne |
| Fetch | Contenu web brut | Basse |

### Configuration des MCP dans AGENT_TOOLS.md

Éditez `~/.liza/AGENT_TOOLS.md` pour personnaliser les MCP:

```markdown
### Tool Requirements by Operation

| Operation | Required Tool | Fallback | Use Fallback When |
|-----------|--------------|----------|-------------------|
| Read multiple files | `mcp__filesystem__read_multiple_files` | Read | Single file only |
| Directory exploration | `mcp__jetbrains__list_directory_tree` | Glob | JetBrains unavailable |
| Code search | `mcp__jetbrains__search_in_files_by_text` | Grep | Regex needed, or <3 files |
```

### Ajout de Nouveau MCP

#### Méthode 1: Via fichier AGENT_TOOLS.md personnalisé

Créez votre propre fichier de configuration:

```markdown
---
liza_version: "0.6.2"
---

# Mes Outils MCP Personnalisés

## MCP Servers

### Nouveau MCP
| Operation | Required Tool | Fallback |
|-----------|--------------|----------|
| Nouvelle opération | `mcp__mon-nouveau__outil` | Outil alternatif |
```

Puis activez-le:
```bash
liza setup --agent-tools ~/mon-fichier-outils.md
```

#### Méthode 2: Via variables d'environnement

Certains MCP peuvent être configurés via des variables d'environnement:
```bash
export MON_MCP_SERVER="serveur:port"
```

### MCP Recommandés

#### Pour le Développement

- **JetBrains MCP** - IntelliJ, WebStorm, GoLand, etc.
- **filesystem MCP** - Accès fichiers local
- **Sequential Thinking** - Raisonnement structuré

#### Pour la Recherche

- **Perplexity MCP** - Recherche web intelligente
- **Context7** - Documentation technique
- **Ref MCP** - Recherche large documentation

#### Pour les Bases de Données

- **PostgreSQL MCP** - Requêtes SQL en lecture seule
- **SQLite MCP** - Accès bases SQLite locales

---

## Utilisation en Mode Pairing

Le mode Pairing est le moyen le plus rapide de commencer avec Liza. L'agent collabore avec vous sous le contrat comportemental.

### Activation du Mode Pairing

Dans n'importe quel projet:

```bash
# Initialisation du projet (une seule fois)
liza init

# Puis lancez votre session d'agent
claude
# ou
codex
# ou tout autre agent configuré
```

### Comportement en Mode Pairing

Une fois activé, l'agent:
1. **Affiche un test canari** - Quatre mots de quatre fichiers de contrat différents
2. **Lit le contrat comportemental** - Comprend les 55+ modes d'échec
3. **Analyse avant d'agir** - Ne commence pas sans validation
4. **Présente des demandes d'approbation** - À chaque changement d'état
5. **Valide avant de terminer** - Vérifie le travail avant de le déclarer fait

### Postures de Pairing

Liza propose plusieurs postures:

| Posture | Description |
|---------|-------------|
| **Senior Peer** | Collabore comme un développeur senior |
| **Socratic Coach** | Pose des questions pour vous faire réfléchir |
| **Rubber Duck** | Explication passo à passo de votre code |
| **Challenger** | Remet en question vos décisions |

### Exemple de Session Pairing

```bash
cd mon-projet
liza init
claude

# Dans la session Claude:
# > Je vais créer une fonctionnalité de connexion
# Liza: "Analysons d'abord les requirements..."
```

---

## Utilisation en Mode Multi-Agent

Le mode Multi-Agent permet une autonomie complète avec plusieurs agents spécialisés.

### Initialisation d'un Projet Multi-Agent

```bash
# Avec un document de vision (goal)
liza init "Goal description" --spec specs/vision.md

# Pour sauter la phase specs et coder directement
liza init "Goal" --spec specs.md --entry-point detailed-spec

# Avec configuration de pipeline personnalisée
liza init "Goal" --spec s.md --config pipeline.yaml --entry-point epic-planning
```

### Structure du Pipeline

Liza fonctionne en trois phases avec 13 rôles:

```
Phase Spécification      Phase Coding           Phase Intégration
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Orchestrator    │    │ Orchestrator    │    │ Integration     │
│ Epic Planner ↔  │    │ Architect ↔     │    │ Analyst ↔       │
│ Epic Plan Review│    │ Architecture Rev│    │ Integration Rev │
│ US Writer ↔     │    │ Code Planner ↔  │    │                 │
│ US Reviewer      │    │ Code Plan Rev   │    │                 │
│                 │    │ Coder ↔         │    │                 │
│                 │    │ Code Reviewer   │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Interface TUI

```bash
# Lancer l'interface TUI
liza tui
```

La TUI affiche:
- État live du système (agents, tâches, alertes)
- Métriques du sprint
- Contrôles pour démarrer/arrêter des agents

### Commandes en Mode Multi-Agent

```bash
# Ajouter une tâche
liza add-task --id t1 --desc "Description" --spec "spec.md" --done "Critères" --scope "Fichiers"

# Passer à la phase suivante
liza proceed

# Point de contrôle du sprint
liza sprint-checkpoint

# Récupérer après crash
liza recover-agent <id>
liza recover-task <id>

# Analyser les patterns
liza analyze
```

---

## Projets Existants vs Nouveaux Projets

### Projet Nouveau

#### Étapes

1. **Créer le dossier du projet**
   ```bash
   mkdir mon-nouveau-projet
   cd mon-nouveau-projet
   git init
   ```

2. **Créer un fichier goal/vision**
   ```bash
   # Créer specs/vision.md avec la description du projet
   mkdir -p specs
   touch specs/vision.md
   ```

3. **Initialiser Liza**
   ```bash
   # Mode interactif (recommandé)
   liza init

   # Mode explicite
   liza init "Description du projet" --spec specs/vision.md
   ```

4. **Lancer la TUI**
   ```bash
   liza tui
   ```

#### Structure Créée

```
mon-nouveau-projet/
├── .liza/                    # Configuration Liza
│   ├── state.yaml           # Tableau noir (blackboard)
│   ├── log.yaml            # Historique d'activité
│   ├── alerts.log           # Logs du daemon
│   └── archive/             # Tâches terminées
├── .worktrees/              # Espaces de travail isolés
│   ├── task-1/
│   └── task-2/
└── specs/
    └── vision.md            # Document de vision
```

### Projet Existant

#### Étapes

1. **Naviguer vers le projet**
   ```bash
   cd /chemin/vers/projet-existant
   ```

2. **Initialiser Liza** (crée le dossier .liza mais ne modifie pas le code existant)
   ```bash
   liza init "Amélioration/Correction" --spec specs/goal.md
   ```

3. **Choisir le mode**
   - **Pairing**: Utilisez `claude`, `codex`, etc. directement
   - **Multi-Agent**: Utilisez `liza tui` pour orchestrer

#### Différences Clés

| Aspect | Projet Nouveau | Projet Existant |
|--------|----------------|------------------|
| Structure folders | Créée par Liza | Doit exister |
| Git | Initialisé ou utilisé | Deja initialisé |
| Worktrees | Créés automatiquement | Créés à la demande |
| État | Clean start | Peut avoir du contexte |

#### Intégration avec CLAUDE.md Existant

Si vous avez déjà un `CLAUDE.md`, Liza vous demandera:
- De le fusionner
- De le remplacer
- D'annuler

```bash
# Pour fusionner automatiquement
liza init --spec vision.md --merge-claude-md
```

---

## Commandes Essentielles

### Commandes Globales

```bash
liza version                  # Version installée
liza help                     # Aide
liza setup                    # Setup global
liza setup --claude           # Setup agent spécifique
liza upgrade                  # Mettre à jour Liza
```

### Commandes de Projet

```bash
liza init "goal" --spec file.md    # Initialiser projet
liza init --config pipeline.yaml   # Avec pipeline custom
liza status                        # Vue d'ensemble dashboard
liza validate                      # Valider l'état
liza get tasks                     # Requêter les tâches
```

### Contrôle du Système

```bash
liza proceed           # Passer à la phase suivante
liza pause            # Mettre en pause
liza resume           # Reprendre
liza stop             # Arrêter
liza start            # Démarrer
```

### Tâches et Agents

```bash
liza add-task --id t1 --desc "..." # Ajouter tâche
liza agent coder                   # Démarrer agent
liza tui                          # Interface TUI interactive
liza sprint-checkpoint            # Point de contrôle
liza recover-agent <id>           # Récupérer après crash
liza recover-task <id>            # Récupérer tâche
```

### Analyse

```bash
liza analyze           # Analyse circuit breaker
liza get tasks         # Liste des tâches
liza get agents        # Liste des agents
liza logs              # Voir les logs
```

---

## Dépannage

### Problèmes Courants

#### "Agent non trouvé"

```bash
# Vérifier que l'agent est installé
which claude  # ou codex, kimi, etc.

# Réinstaller si nécessaire
# Voir la documentation de l'agent concerné
```

#### "Config déjà exists"

```bash
# Forcer le re-setup
liza setup --force
```

#### Erreurs MCP

```bash
# Vérifier la configuration
cat ~/.liza/AGENT_TOOLS.md

# Tester les MCP individuellement
# Voir la doc de chaque MCP
```

#### Conflit avec CLAUDE.md existant

```bash
# Merger automatiquement
liza init --spec vision.md --merge-claude-md

# Ou garder les deux
liza init --spec vision.md
# Puis merger manuellement les contenu
```

### Logs

```bash
# Logs Liza
cat .liza/log.yaml

# Alerts
cat .liza/alerts.log

# Archives
ls .liza/archive/
```

---

## Le Contrat Comportemental

### Principes Fondamentaux

Le contrat comportemental de Liza est le cœur du système. Il enforced 55+ modes d'échec des LLMs:

```
~/.liza/CORE.md              # Contrat principal
~/.liza/CONTRACT_FAILURE_MODE_MAP.md  # Cartographie détaillée
~/.liza/contract-activation.md        # Activation par agent
```

### Les 55+ Modes d'Échec Cartographiés

Le contrat identifie les comportements problématiques:

| Catégorie | Exemples |
|-----------|----------|
| **Sycophancy** | Admission de solutions incorrectes pour faire plaisir |
| **Faking Progress** | Apparence de travail sans avancée réelle |
| **Reasoning Omission** | Sauter l'analyse pour aller au code |
| **False Confidence** |claim être certain alors que ne l'est pas |
| **Context Forgetting** | Perdre le fil des contraintes |
| **Tool Misuse** | Utiliser les outils incorrectement |

### Mécanismes d'Application

Le contrat utilise des "gates" (portes) que l'agent doit franchir:

```
Analyse → Plan → Soumission → Revue → Approbation → Merge
   ↑                                                 |
   └──────── Rejet (retour à l'analyse) ───────────┘
```

### Personnalisation du Contrat

```bash
# Créer des guardrails项目 spécifiques
# Éditer ~/.liza/CORE.md ou créer GUARDRAILS.md à la racine du projet
```

---

## Les Skills Liza

### Liste des 20 Skills

Liza ships with 20 composable skills in `~/.liza/skills/`:

| Skill | Utilisation |
|-------|-------------|
| `adr-backfill` | Backfill Architectural Decision Records |
| `architecture-planning` | Planification architecturale |
| `black-box-red-testing` | Tests de sécurité black-box |
| `clean-code` | Nettoyage et refactoring |
| `code-quality-assessment` | Évaluation qualité code |
| `code-review` | Revues de code |
| `code-spec-backfill` | Backfill specs de code |
| `debugging` | Debugging guidé |
| `detailed-spec-writing` | Écriture de specs détaillées |
| `epic-writing` | Écriture d'epics |
| `feynman` | Explication de concepts complexes |
| `generic-subagent` | Sous-agent générique |
| `have-you-considered` | Remue-méninges |
| `lesson-capture` | Capture de leçons apprises |
| `liza-logs` | Analyse des logs Liza |
| `software-architecture-review` | Revue architecturale |
| `spec-backfill` | Backfill de specifications |
| `spec-review` | Revue de specs |
| `systemic-thinking` | Pensée systémique |
| `user-story-writing` | Écriture de user stories |
| `white-box-red-testing` | Tests white-box |

### Utilisation des Skills

Dans une session Liza, invoquez un skill:

```markdown
# Utilisation dans le contexte
Utilisez le skill [debugging] pour diagnostiquer ce problème.
```

### Créer un Skill Personnalisé

```bash
# Créer ~/.liza/skills/mon-skill/SKILL.md
```

Structure:
```markdown
# Mon Skill

## Objectif
Description du skill

## Quand l'utiliser
Contexte d'utilisation

## Comment l'utiliser
Instructions détaillées
```

---

## Personnalisation du Pipeline YAML

### Structure du Pipeline

Le fichier `~/.liza/pipeline.yaml` définit le workflow:

```yaml
pipeline:
  name: default
  phases:
    - name: specification
      roles:
        - orchestrator
        - epic-planner
        - epic-plan-reviewer
        - us-writer
        - us-reviewer
    - name: coding
      roles:
        - orchestrator
        - architect
        - architecture-reviewer
        - code-planner
        - code-plan-reviewer
        - coder
        - code-reviewer
    - name: integration
      roles:
        - integration-analyst
        - integration-reviewer
        - coder
        - code-reviewer
```

### Pipeline Personnalisé

```bash
# Utiliser un pipeline personnalisé
liza init "goal" --config mon-pipeline.yaml
```

### Entry Points

| Entry Point | Description |
|-------------|-------------|
| `epic-planning` | Commencer par la planification épique |
| `us-writing` | Commencer par l'écriture des user stories |
| `architect` | Passer directement à l'architecture |
| `detailed-spec` | Sauter la phase specs, coder directement |

---

## Intégration avec les Outils Existants

### Intégration Git

#### Hooks Git Automatiques

Liza peut être intégré avec git:

```bash
# Ajouter un hook pre-commit pour validation Liza
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
liza validate || exit 1
EOF
chmod +x .git/hooks/pre-commit
```

#### Worktrees Git

Liza utilise des worktrees isolés pour chaque tâche:

```bash
# Voir les worktrees
git worktree list

# État du worktree pour une tâche
ls .worktrees/
```

### Intégration IDE

#### VS Code

Configurer `.vscode/settings.json`:

```json
{
  "claude.agentMode": "pairing",
  "files.associations": {
    "*.yaml": "liza"
  }
}
```

#### JetBrains

Liza fonctionne nativement avec JetBrains via le MCP. Assurez-vous que:
- Le plugin JetBrains MCP est installé
- L'IDE est lancé et indexé

### Intégration Claude Code

Liza s'intègre avec Claude Code via le fichier `CLAUDE.md`:

```markdown
# Configuration Claude Code pour Liza

## Posture
Senior Peer (collaboratif)

## Contrat
Le contrat Liza s'applique à cette session.
Lisez ~/.liza/CORE.md

## Outils
Préférer les MCP configurés dans ~/.liza/AGENT_TOOLS.md
```

---

## Patterns de Workflow Recommandés

### Pattern 1: Petites Tâches Rapides (Pairing)

```
1. liza init (une seule fois)
2. cd projet
3. claude
4. Travailler collaborativement
5. Validation automatique
```

### Pattern 2: Feature Multi-Agent

```
1. liza init "Nouvelle fonctionnalité" --spec specs/feature.md
2. liza tui
3. Laisser les agents trabajar (spec → code → review)
4. Revue humaine entre sprints
5. liza proceed pour passer à la phase suivante
```

### Pattern 3: Bug Fix Rapide

```
1. cd projet
2. liza init "Fix bug #123" --spec specs/bug123.md
3. liza agent coder --task-id <id>
4. Revue automatique
5. Merge automatique
```

### Pattern 4: Recherche et Documentation

```
1. liza init "Documenter composant X" --spec docs/comp-x.md
2. Utiliser le skill spec-backfill
3. Revue automatique
4. Génération automatique de la doc
```

---

## Meilleures Pratiques et Tips

### Conseils Productivité

1. **Context Management**
   - Gardez les fichiers de specs petits
   - Utilisez les skills pour les tâches récurrentes
   -RTK est activé pour comprimer les sorties

2. **Pairing Efficace**
   - Définissez clairement votre goal avant de commencer
   - Utilisez les postures (Coach, Challenger) selon le besoin
   - Faites des reviews humaines régulières

3. **Multi-Agent**
   - Utilisez la TUI pour surveiller l'état
   - Faites des checkpoints réguliers
   - Analysez les patterns avec `liza analyze`

### Astuces de Debug

```bash
# Logs détaillés
liza logs --verbose

# État actuel
liza status

# Valider l'état
liza validate

# Analyser les problèmes
liza analyze
```

### Variables d'Environnement Utiles

```bash
# Debug
export LIZA_DEBUG=1
export LIZA_LOG_LEVEL=debug

# Chemins personnalisés
export LIZA_CONFIG_DIR=~/.liza-custom
export LIZA_DATA_DIR=./.liza-data

# Agent par défaut
export LIZA_DEFAULT_AGENT=claude
```

---

## Exemples Concrets

### Exemple 1: Nouvelle API REST

```bash
# 1. Créer la spec
cat > specs/api-users.md << 'EOF'
# API Users - REST API

## Goal
Créer une API REST pour la gestion des utilisateurs

## Endpoints
- GET /api/users
- POST /api/users
- GET /api/users/:id
- PUT /api/users/:id
- DELETE /api/users/:id

## Constraints
- Auth JWT
- PostgreSQL
- OpenAPI specs
EOF

# 2. Initialiser Liza
liza init "API Users REST" --spec specs/api-users.md

# 3. Lancer le développement
liza tui
# Ou en pairing
claude
```

### Exemple 2: Bug Fix Production

```bash
# 1. Décrire le bug
cat > specs/bug-login.md << 'EOF'
# Bug: Échec login utilisateurs

## Symptôme
Les utilisateurs ne peuvent pas se connecter après la mise à jour 2.1.0

## Reproduction
1. Aller sur /login
2. Entrer identifiants valides
3. Erreur 500 retournée

## Logs
[Error] panic: nil pointer dereference in auth/service.go:142
EOF

# 2. Initialiser
liza init "Fix bug login" --spec specs/bug-login.md

# 3. Mode pairing pour un fix rapide
cd .worktrees/task-1
claude
```

### Exemple 3: Refactoring

```bash
# 1. Spec de refactoring
cat > specs/refactor-auth.md << 'EOF'
# Refactoring module Auth

## Objectif
Passer de monolithique à architecture hexagonale

## Changements
- Extraire interface UserRepository
- Créer adaptateurs PostgreSQL et Mock
- Injecter dépendances
- Ajouter tests unitaires

## Contrainte
Ne pas changer l'API publique
EOF

# 2. Initialiser
liza init "Refactoring auth" --spec specs/refactor-auth.md --entry-point architect

# 3. Lancer agents
liza tui
```

---

## Monitoring et Métriques

### Métriques Sprint

Liza tracks:
- Nombre de tâches complétées
- Temps moyen par tâche
- Nombre de rejets/revisions
- Taux d'approbation premier passage
- Tokens utilisés

### Commandes de Monitoring

```bash
# Dashboard
liza status

# Tâches
liza get tasks --status done

# Agents
liza get agents

# Métriques
liza metrics
```

---

## Sécurité et Bonnes Pratiques

### Bonnes Pratiques

1. **Ne pas exposer les clés API**
   - Utiliser des variables d'environnement
   - Ne pas commit les secrets

2. **Validation des accès**
   - Les agents ont des permissions limitées
   - Chaque action nécessite approbation

3. **Audit**
   - Toutes les actions sont logguées
   - Historique complet dans `log.yaml`

### Restrictions

Le contrat comportemental inclut:
- Pas d'opérations destructives sans approbation
- Pas d'accès à des ressources non autorisées
- Validation systématique avant exécution

---

## Ressources Complémentaires

- [README principal](../README.md)
- [Guide Pairing](./USAGE_PAIRING.md)
- [Guide Multi-Agent](./USAGE_MULTI_AGENTS.md)
- [Configuration Avancée](./CONFIGURATION.md)
- [Recettes](./RECIPES.md)
- [Dépannage](./TROUBLESHOOTING.md)
- [Contrats](./contracts/)
- [Skills](./skills/)

---

## Glossaire

| Terme | Définition |
|-------|------------|
| **Blackboard** | Fichier YAML `.liza/state.yaml` servant de tableau noir |
| **Contract** | Le contrat comportemental de Liza |
| **Doer/Reviewer** | Paire d'agents: un qui fait, un qui review |
| **Gate** | Porte/validation que l'agent doit franchir |
| **MCP** | Model Context Protocol - protocole d'outils externes |
| **Pairing** | Mode collaboration humain-agent |
| **Skill** | Compétence encapsulée réutilisable |
| **Sprint** | Cycle de travail dans Liza |
| **Supervisor** | Wrapper Go qui enforce les règles |
| **Worktree** | Clone Git isolé pour chaque tâche |
| **TUI** | Terminal User Interface de Liza |

---

## FAQ

### Quelle est la différence entre pairing et multi-agent?

- **Pairing**: Vous travailz directement avec un agent (votre "pair")
- **Multi-Agent**: Plusieurs agents automatisés travaillent ensemble

### Liza fonctionne-t-il avec tous les modèles?

Non. Voir le tableau de compatibilité dans la section Configuration des Agents.

### Liza modifie-t-il mon code?

Liza ne modifie jamais le code sans approbation humaine ou approbation du reviewer.

### Combien de tâches puis-je exécuter en parallèle?

Via la TUI, vous pouvez lancer plusieurs agents simultanément. Le nombre dépend de vos ressources.

### Liza est-il prêt pour la production?

Le mode Pairing est battle-tested (~90% du code en production). Le mode Multi-Agent est en amélioration continue.

---

*Document généré pour Guide de Configuration Liza v0.6.2*