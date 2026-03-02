---
title: Pipeline CI
description: Fonctionnement du pipeline CI d'OpenClaw
summary: "Graphe des jobs CI, portes de scope et commandes locales équivalentes"
read_when:
  - Vous devez comprendre pourquoi un job CI s'est exécuté ou non
  - Vous déboguez des vérifications GitHub Actions en échec
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/ci.md
  workflow: manual
---

# Pipeline CI

Le CI s'exécute à chaque push sur `main` et à chaque pull request. Il utilise un scoping intelligent pour ignorer les jobs coûteux quand seules les docs ou le code natif ont changé.

## Vue d'ensemble des jobs

| Job               | Objectif                                           | Quand il s'exécute          |
| ----------------- | -------------------------------------------------- | --------------------------- |
| `docs-scope`      | Détecter les changements docs uniquement           | Toujours                    |
| `changed-scope`   | Détecter les zones modifiées (node/macos/android)  | PRs non-docs                |
| `check`           | Types TypeScript, lint, format                     | Changements non-docs        |
| `check-docs`      | Lint Markdown + vérification de liens cassés        | Docs modifiées              |
| `code-analysis`   | Vérification de seuil de LOC (1000 lignes)         | PRs uniquement              |
| `secrets`         | Détecter les secrets divulgués                     | Toujours                    |
| `build-artifacts` | Build dist une fois, partagé avec les autres jobs  | Non-docs, changements node  |
| `release-check`   | Valider le contenu de npm pack                     | Après le build              |
| `checks`          | Tests Node/Bun + vérification de protocole         | Non-docs, changements node  |
| `checks-windows`  | Tests spécifiques Windows                          | Non-docs, changements node  |
| `macos`           | Lint/build/test Swift + tests TS                   | PRs avec changements macos  |
| `android`         | Build Gradle + tests                               | Non-docs, changements android |

## Ordre Fail-Fast

Les jobs sont ordonnés pour que les vérifications peu coûteuses échouent avant que les jobs coûteux ne s'exécutent :

1. `docs-scope` + `code-analysis` + `check` (parallèle, ~1-2 min)
2. `build-artifacts` (bloqué par les précédents)
3. `checks`, `checks-windows`, `macos`, `android` (bloqué par le build)

## Runners

| Runner                           | Jobs                                          |
| -------------------------------- | --------------------------------------------- |
| `blacksmith-16vcpu-ubuntu-2404`  | La plupart des jobs Linux, y compris la détection de scope |
| `blacksmith-16vcpu-windows-2025` | `checks-windows`                              |
| `macos-latest`                   | `macos`, `ios`                                |

## Équivalents locaux

```bash
pnpm check          # types + lint + format
pnpm test           # tests vitest
pnpm check:docs     # format docs + lint + liens cassés
pnpm release:check  # valider npm pack
```
