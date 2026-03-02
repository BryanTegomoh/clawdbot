---
summary: "Référence CLI pour `openclaw agent` (envoyer un tour d'agent via le Gateway)"
read_when:
  - Vous souhaitez exécuter un tour d'agent depuis des scripts (avec livraison optionnelle de la réponse)
title: "agent"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/agent.md
  workflow: manual
---

# `openclaw agent`

Exécuter un tour d'agent via le Gateway (utilisez `--local` pour le mode intégré).
Utilisez `--agent <id>` pour cibler directement un agent configuré.

Liens :

- Outil d'envoi d'agent : [Agent send](/tools/agent-send)

## Exemples

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
