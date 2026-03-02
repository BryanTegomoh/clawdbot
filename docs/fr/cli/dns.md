---
summary: "Référence CLI pour `openclaw dns` (utilitaires de découverte étendue)"
read_when:
  - Vous souhaitez la découverte étendue (DNS-SD) via Tailscale + CoreDNS
  - Vous configurez un DNS fractionné pour un domaine de découverte personnalisé (exemple : openclaw.internal)
title: "dns"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/dns.md
  workflow: manual
---

# `openclaw dns`

Utilitaires DNS pour la découverte étendue (Tailscale + CoreDNS). Actuellement centré sur macOS + Homebrew CoreDNS.

Liens :

- Découverte du Gateway : [Discovery](/gateway/discovery)
- Configuration de découverte étendue : [Configuration](/gateway/configuration)

## Configuration

```bash
openclaw dns setup
openclaw dns setup --apply
```
