---
summary: "Référence CLI pour `openclaw setup` (initialiser la configuration + l'espace de travail)"
read_when:
  - Vous faites la configuration initiale sans l'assistant d'intégration complet
  - Vous souhaitez définir le chemin de l'espace de travail par défaut
title: "setup"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/setup.md
  workflow: manual
---

# `openclaw setup`

Initialiser `~/.openclaw/openclaw.json` et l'espace de travail de l'agent.

Liens connexes :

- Démarrage : [Démarrage](/start/getting-started)
- Assistant : [Intégration](/start/onboarding)

## Exemples

```bash
openclaw setup
openclaw setup --workspace ~/.openclaw/workspace
```

Pour lancer l'assistant via setup :

```bash
openclaw setup --wizard
```
