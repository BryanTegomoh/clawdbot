---
summary: "Référence CLI pour `openclaw models` (statut/liste/définir/scanner, alias, replis, authentification)"
read_when:
  - Vous souhaitez changer les modèles par défaut ou voir le statut d'authentification du fournisseur
  - Vous souhaitez scanner les modèles/fournisseurs disponibles et déboguer les profils d'authentification
title: "models"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/models.md
  workflow: manual
---

# `openclaw models`

Découverte, analysé et configuration de modèles (modèle par défaut, replis, profils d'authentification).

Liens :

- Fournisseurs + modèles : [Models](/providers/models)
- Configuration d'authentification du fournisseur : [Getting started](/start/getting-started)

## Commandes courantes

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` affiche les valeurs par défaut/replis résolus plus un aperçu de l'authentification.
Lorsque des instantanés d'utilisation du fournisseur sont disponibles, la section de statut OAuth/jeton inclut les en-têtes d'utilisation du fournisseur.
Ajoutez `--probe` pour exécuter des sondes d'authentification en direct contre chaque profil de fournisseur configuré.
Les sondes sont des requêtes reelles (peuvent consommer des jetons et déclencher des limités de taux).
Utilisez `--agent <id>` pour inspecter l'état modèle/authentification d'un agent configuré. Lorsque omis, la commande utilise `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` si défini, sinon l'agent par défaut configuré.

Notes :

- `models set <model-or-alias>` accepte `provider/model` ou un alias.
- Les références de modèles sont analysées en divisant sur le **premier** `/`. Si l'identifiant du modèle inclut `/` (style OpenRouter), incluez le préfixe du fournisseur (exemple : `openrouter/moonshotai/kimi-k2`).
- Si vous omettez le fournisseur, OpenClaw traite l'entrée comme un alias ou un modèle pour le **fournisseur par défaut** (fonctionne uniquement lorsqu'il n'y a pas de `/` dans l'identifiant du modèle).

### `models status`

Options :

- `--json`
- `--plain`
- `--check` (sortie 1=expire/manquant, 2=en cours d'expiration)
- `--probe` (sonde en direct des profils d'authentification configurés)
- `--probe-provider <name>` (sonder un fournisseur)
- `--probe-profile <id>` (répéter ou identifiants de profils séparés par virgules)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`
- `--agent <id>` (identifiant d'agent configuré ; surcharge `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`)

## Alias + replis

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## Profils d'authentification

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` exécute le flux d'authentification d'un plugin fournisseur (OAuth/clé API). Utilisez `openclaw plugins list` pour voir quels fournisseurs sont installés.

Notes :

- `setup-token` demande une valeur de jeton de configuration (génèrez-le avec `claude setup-token` sur n'importe quelle machine).
- `paste-token` accepte une chaine de jeton génèree ailleurs ou par automatisation.
