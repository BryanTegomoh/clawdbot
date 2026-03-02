---
summary: "Utiliser l'API compatible Anthropic de Synthetic dans OpenClaw"
read_when:
  - Vous souhaitez utiliser Synthetic comme fournisseur de modèles
  - Vous avez besoin d'une clé API Synthetic ou de la configuration de l'URL de base
title: "Synthetic"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/synthetic.md
  workflow: manual
---

# Synthetic

Synthetic expose des endpoints compatibles Anthropic. OpenClaw l'enregistre comme fournisseur `synthetic` et utilise l'API Anthropic Messages.

## Configuration rapide

1. Définissez `SYNTHETIC_API_KEY` (ou lancez l'assistant ci-dessous).
2. Lancez l'onboarding :

```bash
openclaw onboard --auth-choice synthetic-api-key
```

Le modèle par défaut est défini sur :

```
synthetic/hf:MiniMaxAI/MiniMax-M2.1
```

## Exemple de configuration

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Note : Le client Anthropic d'OpenClaw ajoute `/v1` à l'URL de base, utilisez donc `https://api.synthetic.new/anthropic` (et non `/anthropic/v1`). Si Synthetic modifie son URL de base, surchargez `models.providers.synthetic.baseUrl`.

## Catalogue de modèles

Tous les modèles ci-dessous utilisent un coût de `0` (entrée/sortie/cache).

| ID du modèle                                           | Fenêtre de contexte | Tokens max | Raisonnement | Entrée       |
| ------------------------------------------------------ | ------------------- | ---------- | ------------ | ------------ |
| `hf:MiniMaxAI/MiniMax-M2.1`                            | 192000              | 65536      | false        | text         |
| `hf:moonshotai/Kimi-K2-Thinking`                       | 256000              | 8192       | true         | text         |
| `hf:zai-org/GLM-4.7`                                   | 198000              | 128000     | false        | text         |
| `hf:deepseek-ai/DeepSeek-R1-0528`                      | 128000              | 8192       | false        | text         |
| `hf:deepseek-ai/DeepSeek-V3-0324`                      | 128000              | 8192       | false        | text         |
| `hf:deepseek-ai/DeepSeek-V3.1`                         | 128000              | 8192       | false        | text         |
| `hf:deepseek-ai/DeepSeek-V3.1-Terminus`                | 128000              | 8192       | false        | text         |
| `hf:deepseek-ai/DeepSeek-V3.2`                         | 159000              | 8192       | false        | text         |
| `hf:meta-llama/Llama-3.3-70B-Instruct`                 | 128000              | 8192       | false        | text         |
| `hf:meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | 524000              | 8192       | false        | text         |
| `hf:moonshotai/Kimi-K2-Instruct-0905`                  | 256000              | 8192       | false        | text         |
| `hf:openai/gpt-oss-120b`                               | 128000              | 8192       | false        | text         |
| `hf:Qwen/Qwen3-235B-A22B-Instruct-2507`                | 256000              | 8192       | false        | text         |
| `hf:Qwen/Qwen3-Coder-480B-A35B-Instruct`               | 256000              | 8192       | false        | text         |
| `hf:Qwen/Qwen3-VL-235B-A22B-Instruct`                  | 250000              | 8192       | false        | text + image |
| `hf:zai-org/GLM-4.5`                                   | 128000              | 128000     | false        | text         |
| `hf:zai-org/GLM-4.6`                                   | 198000              | 128000     | false        | text         |
| `hf:deepseek-ai/DeepSeek-V3`                           | 128000              | 8192       | false        | text         |
| `hf:Qwen/Qwen3-235B-A22B-Thinking-2507`                | 256000              | 8192       | true         | text         |

## Notes

- Les références de modèles utilisent `synthetic/<modelId>`.
- Si vous activez une liste de modèles autorisés (`agents.defaults.models`), ajoutez chaque modèle que vous prévoyez d'utiliser.
- Voir [Fournisseurs de modèles](/concepts/model-providers) pour les règles des fournisseurs.
