---
summary: "Utiliser l'API unifiée de Kilo Gateway pour accéder à de nombreux modèles dans OpenClaw"
read_when:
  - Vous souhaitez une seule clé API pour de nombreux LLMs
  - Vous voulez exécuter des modèles via Kilo Gateway dans OpenClaw
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/kilocode.md
  workflow: manual
---

# Kilo Gateway

Kilo Gateway fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles derrière un seul endpoint et une seule clé API. Elle est compatible OpenAI, donc la plupart des SDK OpenAI fonctionnent en changeant l'URL de base.

## Obtenir une clé API

1. Rendez-vous sur [app.kilo.ai](https://app.kilo.ai)
2. Connectez-vous ou créez un compte
3. Allez dans API Keys et générez une nouvelle clé

## Configuration CLI

```bash
openclaw onboard --kilocode-api-key <key>
```

Ou définissez la variable d'environnement :

```bash
export KILOCODE_API_KEY="your-api-key"
```

## Extrait de configuration

```json5
{
  env: { KILOCODE_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kilocode/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Références de modèles exposées

Le catalogue intégré de Kilo Gateway expose actuellement ces références de modèles :

- `kilocode/anthropic/claude-opus-4.6` (par défaut)
- `kilocode/z-ai/glm-5:free`
- `kilocode/minimax/minimax-m2.5:free`
- `kilocode/anthropic/claude-sonnet-4.5`
- `kilocode/openai/gpt-5.2`
- `kilocode/google/gemini-3-pro-preview`
- `kilocode/google/gemini-3-flash-preview`
- `kilocode/x-ai/grok-code-fast-1`
- `kilocode/moonshotai/kimi-k2.5`

## Notes

- Les références de modèles sont `kilocode/<provider>/<model>` (par exemple `kilocode/anthropic/claude-opus-4.6`).
- Modèle par défaut : `kilocode/anthropic/claude-opus-4.6`
- URL de base : `https://api.kilo.ai/api/gateway/`
- Pour plus d'options de modèles/fournisseurs, voir [/concepts/model-providers](/concepts/model-providers).
- Kilo Gateway utilisé un token Bearer avec votre clé API en interne.
