---
summary: "Référence CLI pour `openclaw tui` (interface terminal connectée au Gateway)"
read_when:
  - Vous souhaitez une interface terminal pour le Gateway (compatible avec le mode distant)
  - Vous souhaitez passer url/token/session depuis des scripts
title: "tui"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/tui.md
  workflow: manual
---

# `openclaw tui`

Ouvrir l'interface terminal connectée au Gateway.

Liens connexes :

- Guide TUI : [TUI](/web/tui)

## Exemples

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```
