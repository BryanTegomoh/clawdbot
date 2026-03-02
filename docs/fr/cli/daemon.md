---
summary: "Référence CLI pour `openclaw daemon` (alias historique pour la gestion du service Gateway)"
read_when:
  - Vous utilisez encore `openclaw daemon ...` dans vos scripts
  - Vous avez besoin des commandes de cycle de vie du service (install/start/stop/restart/status)
title: "daemon"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/daemon.md
  workflow: manual
---

# `openclaw daemon`

Alias historique pour les commandes de gestion du service Gateway.

`openclaw daemon ...` correspond à la même surface de contrôle de service que les commandes de service `openclaw gateway ...`.

## Utilisation

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## Sous-commandes

- `status` : afficher l'état d'installation du service et sonder la santé du Gateway
- `install` : installer le service (`launchd`/`systemd`/`schtasks`)
- `uninstall` : supprimer le service
- `start` : démarrer le service
- `stop` : arrêter le service
- `restart` : redémarrer le service

## Options communes

- `status` : `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
- `install` : `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
- cycle de vie (`uninstall|start|stop|restart`) : `--json`

## A préférer

Utilisez [`openclaw gateway`](/cli/gateway) pour la documentation et les exemples actuels.
