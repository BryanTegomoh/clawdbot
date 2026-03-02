---
summary: "Utiliser les modèles Mistral et la transcription Voxtral avec OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles Mistral dans OpenClaw
  - Vous avez besoin de l'onboarding de la clé API Mistral et des références de modèles
title: "Mistral"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/mistral.md
  workflow: manual
---

# Mistral

OpenClaw prend en charge Mistral à la fois pour le routage de modèles texte/image (`mistral/...`) et pour la transcription audio via Voxtral dans la compréhension multimédia.
Mistral peut également être utilisé pour les embeddings mémoire (`memorySearch.provider = "mistral"`).

## Configuration CLI

```bash
openclaw onboard --auth-choice mistral-api-key
# ou en mode non interactif
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## Extrait de configuration (fournisseur LLM)

```json5
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## Extrait de configuration (transcription audio avec Voxtral)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## Notes

- L'authentification Mistral utilisé `MISTRAL_API_KEY`.
- L'URL de base du fournisseur est par défaut `https://api.mistral.ai/v1`.
- Le modèle par défaut de l'onboarding est `mistral/mistral-large-latest`.
- Le modèle audio par défaut pour la compréhension multimédia Mistral est `voxtral-mini-latest`.
- Le chemin de transcription multimédia utilisé `/v1/audio/transcriptions`.
- Le chemin des embeddings mémoire utilisé `/v1/embeddings` (modèle par défaut : `mistral-embed`).
