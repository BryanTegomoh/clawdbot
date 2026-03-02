---
summary: "Référence CLI pour `openclaw completion` (générer/installer les scripts de completion shell)"
read_when:
  - Vous souhaitez la completion shell pour zsh/bash/fish/PowerShell
  - Vous devez mettre en cache les scripts de completion sous l'état OpenClaw
title: "completion"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/completion.md
  workflow: manual
---

# `openclaw completion`

Générer des scripts de completion shell et optionnellement les installer dans votre profil shell.

## Utilisation

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## Options

- `-s, --shell <shell>` : cible shell (`zsh`, `bash`, `powershell`, `fish` ; défaut : `zsh`)
- `-i, --install` : installer la completion en ajoutant une ligne source à votre profil shell
- `--write-state` : écrire le(s) script(s) de completion dans `$OPENCLAW_STATE_DIR/completions` sans afficher sur stdout
- `-y, --yes` : ignorer les invites de confirmation d'installation

## Notes

- `--install` écrit un petit bloc "OpenClaw Completion" dans votre profil shell et le dirige vers le script mis en cache.
- Sans `--install` ou `--write-state`, la commande affiche le script sur stdout.
- La génération de la completion charge immédiatement les arbres de commandes pour que les sous-commandes imbriquées soient incluses.
