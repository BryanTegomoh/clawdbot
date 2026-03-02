---
summary: "Workflow Bun (expérimental) : installation et problèmes connus vs pnpm"
read_when:
  - Vous voulez la boucle de développement locale la plus rapide (bun + watch)
  - Vous rencontrez des problèmes d'installation/patch/scripts de cycle de vie avec Bun
title: "Bun (Expérimental)"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/bun.md
  workflow: manual
---

# Bun (expérimental)

Objectif : exécuter ce dépôt avec **Bun** (optionnel, non recommandé pour WhatsApp/Telegram)
sans diverger des workflows pnpm.

**Non recommandé pour le runtime Gateway** (bugs WhatsApp/Telegram). Utilisez Node en production.

## Statut

- Bun est un runtime local optionnel pour exécuter TypeScript directement (`bun run …`, `bun --watch …`).
- `pnpm` est la valeur par défaut pour les compilations et reste entièrement supporté (et utilise par certains outils de documentation).
- Bun ne peut pas utiliser `pnpm-lock.yaml` et l'ignorera.

## Installation

Par défaut :

```sh
bun install
```

Note : `bun.lock`/`bun.lockb` sont dans le gitignore, donc pas de bruit dans le dépôt dans les deux cas. Si vous ne voulez _aucune écriture de fichier lock_ :

```sh
bun install --no-save
```

## Compilation / Test (Bun)

```sh
bun run build
bun run vitest run
```

## Scripts de cycle de vie Bun (bloqués par défaut)

Bun peut bloquer les scripts de cycle de vie des dépendances sauf s'ils sont explicitement approuvés (`bun pm untrusted` / `bun pm trust`).
Pour ce dépôt, les scripts couramment bloqués ne sont pas nécessaires :

- `@whiskeysockets/baileys` `preinstall` : vérifie que Node majeur >= 20 (nous utilisons Node 22+).
- `protobufjs` `postinstall` : émet des avertissements sur des schémas de version incompatibles (pas d'artefacts de compilation).

Si vous rencontrez un vrai problème d'exécution nécessitant ces scripts, approuvez-les explicitement :

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## Limitations

- Certains scripts utilisent encore pnpm en dur (par ex. `docs:build`, `ui:*`, `protocol:check`). Exécutez-les via pnpm pour l'instant.
