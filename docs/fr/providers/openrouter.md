---
summary: "Utiliser l'API unifiée d'OpenRouter pour accéder à de nombreux modèles dans OpenClaw"
read_when:
  - Vous souhaitez une seule clé API pour de nombreux LLMs
  - Vous voulez exécuter des modèles via OpenRouter dans OpenClaw
title: "OpenRouter"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/openrouter.md
  workflow: manual
---

# OpenRouter

OpenRouter fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles derrière un seul endpoint et une seule clé API. Elle est compatible OpenAI, donc la plupart des SDK OpenAI fonctionnent en changeant l'URL de base.

## Configuration CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Extrait de configuration

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Notes

- Les références de modèles sont `openrouter/<provider>/<model>`.
- Pour plus d'options de modèles/fournisseurs, voir [/concepts/model-providers](/concepts/model-providers).
- OpenRouter utilisé un token Bearer avec votre clé API en interne.
