---
summary: "Référence CLI pour `openclaw voicecall` (surface de commande du plugin d'appel vocal)"
read_when:
  - Vous utilisez le plugin d'appel vocal et souhaitez les points d'entrée CLI
  - Vous souhaitez des exemples rapides pour `voicecall call|continue|status|tail|expose`
title: "voicecall"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/voicecall.md
  workflow: manual
---

# `openclaw voicecall`

`voicecall` est une commande fournie par un plugin. Elle n'apparaît que si le plugin d'appel vocal est installé et active.

Documentation principale :

- Plugin d'appel vocal : [Voice Call](/plugins/voice-call)

## Commandes courantes

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Exposer les webhooks (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

Note de sécurité : n'exposez le point de terminaison webhook qu'aux réseaux auxquels vous faites confiance. Préférez Tailscale Serve à Funnel quand c'est possible.
