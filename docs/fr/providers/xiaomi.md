---
summary: "Utiliser Xiaomi MiMo (mimo-v2-flash) avec OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles Xiaomi MiMo dans OpenClaw
  - Vous avez besoin de la configuration XIAOMI_API_KEY
title: "Xiaomi MiMo"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/xiaomi.md
  workflow: manual
---

# Xiaomi MiMo

Xiaomi MiMo est la plateforme API pour les modèles **MiMo**. Elle fournit des API REST compatibles avec les formats OpenAI et Anthropic et utilise des clés API pour l'authentification. Créez votre clé API dans la [console Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw utilise le fournisseur `xiaomi` avec une clé API Xiaomi MiMo.

## Aperçu du modèle

- **mimo-v2-flash** : fenêtre de contexte de 262144 tokens, compatible API Anthropic Messages.
- URL de base : `https://api.xiaomimimo.com/anthropic`
- Autorisation : `Bearer $XIAOMI_API_KEY`

## Configuration CLI

```bash
openclaw onboard --auth-choice xiaomi-api-key
# ou en mode non interactif
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## Extrait de configuration

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Notes

- Référence du modèle : `xiaomi/mimo-v2-flash`.
- Le fournisseur est injecté automatiquement lorsque `XIAOMI_API_KEY` est défini (ou qu'un profil d'authentification existe).
- Voir [/concepts/model-providers](/concepts/model-providers) pour les règles des fournisseurs.
