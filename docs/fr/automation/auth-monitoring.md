---
summary: "Surveiller l'expiration OAuth des fournisseurs de modèles"
read_when:
  - Configuration de la surveillance d'expiration d'authentification ou des alertes
  - Automatisation des vérifications de rafraîchissement OAuth pour Claude Code / Codex
title: "Surveillance de l'authentification"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/auth-monitoring.md
  workflow: manual
---

# Surveillance de l'authentification

OpenClaw expose l'état de santé de l'expiration OAuth via `openclaw models status`. Utilisez cela pour l'automatisation et les alertes ; les scripts sont des extras optionnels pour les workflows mobiles.

## Recommandé : vérification CLI (portable)

```bash
openclaw models status --check
```

Codes de sortie :

- `0` : OK
- `1` : identifiants expirés ou manquants
- `2` : expiration proche (dans les 24h)

Cela fonctionne avec cron/systemd et ne nécessite aucun script supplémentaire.

## Scripts optionnels (ops / workflows mobiles)

Ceux-ci se trouvent dans `scripts/` et sont **optionnels**. Ils supposent un accès SSH à l'hôte du Gateway et sont adaptés pour systemd + Termux.

- `scripts/claude-auth-status.sh` utilisé désormais `openclaw models status --json` comme source de vérité (avec repli sur la lecture directe des fichiers si le CLI n'est pas disponible), donc gardez `openclaw` dans le `PATH` pour les timers.
- `scripts/auth-monitor.sh` : cible pour timer cron/systemd ; envoie des alertes (ntfy ou téléphone).
- `scripts/systemd/openclaw-auth-monitor.{service,timer}` : timer utilisateur systemd.
- `scripts/claude-auth-status.sh` : vérificateur d'authentification Claude Code + OpenClaw (complet/json/simple).
- `scripts/mobile-reauth.sh` : flux de ré-authentification guidé via SSH.
- `scripts/termux-quick-auth.sh` : widget en un clic pour le statut + ouvrir l'URL d'authentification.
- `scripts/termux-auth-widget.sh` : flux complet du widget guidé.
- `scripts/termux-sync-widget.sh` : synchroniser les identifiants Claude Code vers OpenClaw.

Si vous n'avez pas besoin de l'automatisation mobile ou des timers systemd, ignorez ces scripts.
