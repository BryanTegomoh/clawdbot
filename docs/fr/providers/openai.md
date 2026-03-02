---
summary: "Utiliser OpenAI via des clés API ou un abonnement Codex dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles OpenAI dans OpenClaw
  - Vous préférez l'authentification par abonnement Codex plutôt que des clés API
title: "OpenAI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/openai.md
  workflow: manual
---

# OpenAI

OpenAI fournit des API développeur pour les modèles GPT. Codex prend en charge la **connexion ChatGPT** pour l'accès par abonnement ou la **connexion par clé API** pour l'accès à l'usage. Codex cloud nécessite la connexion ChatGPT.

## Option A : Clé API OpenAI (Plateforme OpenAI)

**Idéal pour :** l'accès direct à l'API avec facturation à l'usage.
Obtenez votre clé API depuis le tableau de bord OpenAI.

### Configuration CLI

```bash
openclaw onboard --auth-choice openai-api-key
# ou en mode non interactif
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### Extrait de configuration

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

## Option B : Abonnement OpenAI Code (Codex)

**Idéal pour :** utiliser l'accès par abonnement ChatGPT/Codex au lieu d'une clé API.
Codex cloud nécessite la connexion ChatGPT, tandis que le CLI Codex prend en charge la connexion ChatGPT ou par clé API.

### Configuration CLI (Codex OAuth)

```bash
# Lancer l'OAuth Codex dans l'assistant
openclaw onboard --auth-choice openai-codex

# Ou lancer l'OAuth directement
openclaw models auth login --provider openai-codex
```

### Extrait de configuration (abonnement Codex)

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### Transport Codex par défaut

OpenClaw utilise `pi-ai` pour le streaming des modèles. Pour les modèles `openai-codex/*`, vous pouvez définir
`agents.defaults.models.<provider/model>.params.transport` pour sélectionner le transport :

- Par défaut : `"auto"` (WebSocket en priorité, puis repli sur SSE).
- `"sse"` : forcer SSE
- `"websocket"` : forcer WebSocket
- `"auto"` : essayer WebSocket, puis repli sur SSE

```json5
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.3-codex" },
      models: {
        "openai-codex/gpt-5.3-codex": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

## Notes

- Les références de modèles utilisent toujours `provider/model` (voir [/concepts/models](/concepts/models)).
- Les détails d'authentification et les règles de réutilisation sont dans [/concepts/oauth](/concepts/oauth).
