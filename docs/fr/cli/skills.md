---
summary: "Référence CLI pour `openclaw skills` (list/info/check) et éligibilité des Skills"
read_when:
  - Vous souhaitez voir quelles Skills sont disponibles et prêtes à s'exécuter
  - Vous souhaitez déboguer les binaires/env/config manquants pour les Skills
title: "skills"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/skills.md
  workflow: manual
---

# `openclaw skills`

Inspecter les Skills (intégrées + espace de travail + surcharges gérées) et voir celles qui sont éligibles par rapport aux prérequis manquants.

Liens connexes :

- Système de Skills : [Skills](/tools/skills)
- Configuration des Skills : [Configuration Skills](/tools/skills-config)
- Installations ClawHub : [ClawHub](/tools/clawhub)

## Commandes

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```
