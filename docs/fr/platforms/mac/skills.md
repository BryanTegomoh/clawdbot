---
summary: "UI des paramètres Skills macOS et statut supporte par le gateway"
read_when:
  - Mise a jour de l'UI des paramètres Skills macOS
  - Modification du contrôle d'accès ou du comportement d'installation des skills
title: "Skills"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/skills.md
  workflow: manual
---

# Skills (macOS)

L'application macOS affiche les skills OpenClaw via le gateway ; elle n'analyse pas les skills localement.

## Source de données

- `skills.status` (gateway) retourne tous les skills plus l'eligibilite et les prerequis manquants
  (incluant les blocages de liste blanche pour les skills intégrés).
- Les prerequis sont derives de `metadata.openclaw.requires` dans chaque `SKILL.md`.

## Actions d'installation

- `metadata.openclaw.install` définit les options d'installation (brew/node/go/uv).
- L'application appelle `skills.install` pour exécuter les installateurs sur l'hôte du gateway.
- Le gateway affiche un seul installateur préféré lorsque plusieurs sont fournis
  (brew lorsque disponible, sinon le gestionnaire node depuis `skills.install`, npm par défaut).

## Env/clés API

- L'application stocke les clés dans `~/.openclaw/openclaw.json` sous `skills.entries.<skillKey>`.
- `skills.update` met a jour `enabled`, `apiKey` et `env`.

## Mode distant

- Les installations et mises a jour de configuration se font sur l'hôte du gateway (pas sur le Mac local).
