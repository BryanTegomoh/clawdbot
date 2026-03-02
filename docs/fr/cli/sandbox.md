---
title: CLI Bac à sable
summary: "Gérer les conteneurs bac à sable et inspecter la politique de bac à sable effective"
read_when: "Vous gérez des conteneurs bac à sable ou déboguez le comportement bac à sable/politique d'outils."
status: activé
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/sandbox.md
  workflow: manual
---

# CLI Bac à sable

Gérer les conteneurs Docker de bac à sable pour l'exécution isolée des agents.

## Aperçu

OpenClaw peut exécuter les agents dans des conteneurs Docker isolés pour la sécurité. Les commandes `sandbox` vous aident à gérer ces conteneurs, notamment après les mises à jour ou les changements de configuration.

## Commandes

### `openclaw sandbox explain`

Inspecter le mode/la portée/l'accès à l'espace de travail du bac à sable **effectif**, la politique d'outils du bac à sable et les portes élevées (avec les chemins de clés de configuration pour correction).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### `openclaw sandbox list`

Lister tous les conteneurs bac à sable avec leur statut et configuration.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # Lister uniquement les conteneurs navigateur
openclaw sandbox list --json     # Sortie JSON
```

**La sortie inclut :**

- Nom du conteneur et statut (en cours d'exécution/arrêté)
- Image Docker et si elle correspond à la configuration
- Ancienneté (temps depuis la création)
- Temps d'inactivité (temps depuis la dernière utilisation)
- Session/agent associé

### `openclaw sandbox recreate`

Supprimer les conteneurs bac à sable pour forcer la recréation avec des images/configurations mises à jour.

```bash
openclaw sandbox recreate --all                # Recréer tous les conteneurs
openclaw sandbox recreate --session main       # Session spécifique
openclaw sandbox recreate --agent mybot        # Agent spécifique
openclaw sandbox recreate --browser            # Uniquement les conteneurs navigateur
openclaw sandbox recreate --all --force        # Ignorer la confirmation
```

**Options :**

- `--all` : recréer tous les conteneurs bac à sable
- `--session <key>` : recréer le conteneur pour une session spécifique
- `--agent <id>` : recréer les conteneurs pour un agent spécifique
- `--browser` : recréer uniquement les conteneurs navigateur
- `--force` : ignorer l'invite de confirmation

**Important :** Les conteneurs sont automatiquement recréés lorsque l'agent est utilisé ensuite.

## Cas d'utilisation

### Après la mise à jour des images Docker

```bash
# Tirer la nouvelle image
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# Mettre à jour la configuration pour utiliser la nouvelle image
# Modifier la configuration : agents.defaults.sandbox.docker.image (ou agents.list[].sandbox.docker.image)

# Recréer les conteneurs
openclaw sandbox recreate --all
```

### Après un changement de configuration du bac à sable

```bash
# Modifier la configuration : agents.defaults.sandbox.* (ou agents.list[].sandbox.*)

# Recréer pour appliquer la nouvelle configuration
openclaw sandbox recreate --all
```

### Après un changement de setupCommand

```bash
openclaw sandbox recreate --all
# ou juste un agent :
openclaw sandbox recreate --agent family
```

### Pour un agent spécifique uniquement

```bash
# Mettre à jour uniquement les conteneurs d'un agent
openclaw sandbox recreate --agent alfred
```

## Pourquoi est-ce nécessaire ?

**Problème :** Lorsque vous mettez à jour les images Docker ou la configuration du bac à sable :

- Les conteneurs existants continuent de fonctionner avec les anciens paramètres
- Les conteneurs ne sont élagués qu'après 24h d'inactivité
- Les agents utilisés régulièrement conservent indéfiniment les anciens conteneurs

**Solution :** Utilisez `openclaw sandbox recreate` pour forcer la suppression des anciens conteneurs. Ils seront recréés automatiquement avec les paramètres actuels lorsque nécessaire.

Astuce : préférez `openclaw sandbox recreate` à `docker rm` manuel. Cela utilise le nommage des conteneurs du Gateway et évite les incohérences lorsque les clés de portée/session changent.

## Configuration

Les paramètres du bac à sable se trouvent dans `~/.openclaw/openclaw.json` sous `agents.defaults.sandbox` (les surcharges par agent vont dans `agents.list[].sandbox`) :

```jsonc
{