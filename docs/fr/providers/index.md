---
summary: "Fournisseurs de modèles (LLMs) pris en charge par OpenClaw"
read_when:
  - Vous souhaitez choisir un fournisseur de modèles
  - Vous avez besoin d'un aperçu rapide des backends LLM pris en charge
title: "Fournisseurs de modèles"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/index.md
  workflow: manual
---

# Fournisseurs de modèles

OpenClaw peut utiliser de nombreux fournisseurs de LLM. Choisissez un fournisseur, authentifiez-vous, puis définissez le modèle par défaut sous la forme `provider/model`.

Vous cherchez la documentation des canaux de discussion (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/etc.) ? Consultez [Canaux](/channels).

## En vedette : Venice (Venice AI)

Venice est notre configuration Venice AI recommandée pour une inférence axée sur la confidentialité, avec la possibilité d'utiliser Opus pour les tâches complexes.

- Par défaut : `venice/llama-3.3-70b`
- Meilleur choix global : `venice/claude-opus-45` (Opus reste le plus performant)

Voir [Venice AI](/providers/venice).

## Démarrage rapide

1. Authentifiez-vous auprès du fournisseur (généralement via `openclaw onboard`).
2. Définissez le modèle par défaut :

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Documentation des fournisseurs

- [OpenAI (API + Codex)](/providers/openai)
- [Anthropic (API + Claude Code CLI)](/providers/anthropic)
- [Qwen (OAuth)](/providers/qwen)
- [OpenRouter](/providers/openrouter)
- [LiteLLM (passerelle unifiée)](/providers/litellm)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Together AI](/providers/together)
- [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
- [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
- [Mistral](/providers/mistral)
- [OpenCode Zen](/providers/opencode)
- [Amazon Bedrock](/providers/bedrock)
- [Z.AI](/providers/zai)
- [Xiaomi](/providers/xiaomi)
- [Modèles GLM](/providers/glm)
- [MiniMax](/providers/minimax)
- [Venice (Venice AI, axé confidentialité)](/providers/venice)
- [Hugging Face (Inference)](/providers/huggingface)
- [Ollama (modèles locaux)](/providers/ollama)
- [vLLM (modèles locaux)](/providers/vllm)
- [Qianfan](/providers/qianfan)
- [NVIDIA](/providers/nvidia)

## Fournisseurs de transcription

- [Deepgram (transcription audio)](/providers/deepgram)

## Outils communautaires

- [Claude Max API Proxy](/providers/claude-max-api-proxy) - Utilisez votre abonnement Claude Max/Pro comme endpoint API compatible OpenAI

Pour le catalogue complet des fournisseurs (xAI, Groq, Mistral, etc.) et la configuration avancée, consultez [Fournisseurs de modèles](/concepts/model-providers).
