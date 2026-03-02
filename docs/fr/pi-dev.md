---
title: "Workflow de développement Pi"
summary: "Workflow développeur pour l'intégration Pi : build, test et validation live"
read_when:
  - Travail sur le code ou les tests d'intégration Pi
  - Exécution des flux lint, typecheck et test live spécifiques à Pi
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/pi-dev.md
  workflow: manual
---

# Workflow de développement Pi

Ce guide résume un workflow efficace pour travailler sur l'intégration Pi dans OpenClaw.

## Vérification de types et linting

- Vérification de types et build : `pnpm build`
- Lint : `pnpm lint`
- Vérification du format : `pnpm format`
- Porte complète avant push : `pnpm lint && pnpm build && pnpm test`

## Exécuter les tests Pi

Exécutez le jeu de tests centré sur Pi directement avec Vitest :

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

Pour inclure l'exercice live du fournisseur :

```bash
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

Cela couvre les principales suites unitaires Pi :

- `src/agents/pi-*.test.ts`
- `src/agents/pi-embedded-*.test.ts`
- `src/agents/pi-tools*.test.ts`
- `src/agents/pi-settings.test.ts`
- `src/agents/pi-tool-définition-adapter.test.ts`
- `src/agents/pi-extensions/*.test.ts`

## Tests manuels

Flux recommandé :

- Exécutez le Gateway en mode dev :
  - `pnpm gateway:dev`
- Déclenchez l'agent directement :
  - `pnpm openclaw agent --message "Hello" --thinking low`
- Utilisez le TUI pour le débogage interactif :
  - `pnpm tui`

Pour le comportement des appels d'outils, demandez une action `read` ou `exec` afin de voir le streaming d'outils et la gestion des payloads.

## Réinitialisation complète

L'état se trouve dans le répertoire d'état d'OpenClaw. Par défaut c'est `~/.openclaw`. Si `OPENCLAW_STATE_DIR` est défini, utilisez ce répertoire à la place.

Pour tout réinitialiser :

- `openclaw.json` pour la config
- `credentials/` pour les profils d'authentification et les tokens
- `agents/<agentId>/sessions/` pour l'historique des sessions d'agent
- `agents/<agentId>/sessions.json` pour l'index des sessions
- `sessions/` si des chemins legacy existent
- `workspace/` si vous voulez un espace de travail vierge

Si vous souhaitez uniquement réinitialiser les sessions, supprimez `agents/<agentId>/sessions/` et `agents/<agentId>/sessions.json` pour cet agent. Gardez `credentials/` si vous ne voulez pas vous ré-authentifier.

## Références

- [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
- [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)
