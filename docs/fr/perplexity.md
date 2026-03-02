---
summary: "Configuration de Perplexity Sonar pour web_search"
read_when:
  - Vous souhaitez utiliser Perplexity Sonar pour la recherche web
  - Vous avez besoin d'une PERPLEXITY_API_KEY ou de la configuration OpenRouter
title: "Perplexity Sonar"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/perplexity.md
  workflow: manual
---

# Perplexity Sonar

OpenClaw peut utiliser Perplexity Sonar pour l'outil `web_search`. Vous pouvez vous connecter
via l'API directe de Perplexity ou via OpenRouter.

## Options d'API

### Perplexity (direct)

- URL de base : [https://api.perplexity.ai](https://api.perplexity.ai)
- Variable d'environnement : `PERPLEXITY_API_KEY`

### OpenRouter (alternative)

- URL de base : [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
- Variable d'environnement : `OPENROUTER_API_KEY`
- Prend en charge les crédits prépayés/crypto.

## Exemple de configuration

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Passage depuis Brave

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

Si `PERPLEXITY_API_KEY` et `OPENROUTER_API_KEY` sont tous deux définis, définissez
`tools.web.search.perplexity.baseUrl` (ou `tools.web.search.perplexity.apiKey`)
pour lever l'ambiguïté.

Si aucune URL de base n'est définie, OpenClaw choisit une valeur par défaut basée sur la source de la clé API :

- `PERPLEXITY_API_KEY` ou `pplx-...` → Perplexity direct (`https://api.perplexity.ai`)
- `OPENROUTER_API_KEY` ou `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
- Formats de clé inconnus → OpenRouter (repli sûr)

## Modèles

- `perplexity/sonar` — Q&R rapide avec recherche web
- `perplexity/sonar-pro` (par défaut) — raisonnement multi-étapes + recherche web
- `perplexity/sonar-reasoning-pro` — recherche approfondie

Voir [Outils web](/tools/web) pour la configuration complète de web_search.
