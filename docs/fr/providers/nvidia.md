---
summary: "Utiliser l'API compatible OpenAI de NVIDIA dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles NVIDIA dans OpenClaw
  - Vous avez besoin de la configuration NVIDIA_API_KEY
title: "NVIDIA"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/nvidia.md
  workflow: manual
---

# NVIDIA

NVIDIA fournit une API compatible OpenAI à `https://integrate.api.nvidia.com/v1` pour les modèles Nemotron et NeMo. Authentifiez-vous avec une clé API depuis [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Configuration CLI

Exportez la clé une fois, puis lancez l'onboarding et définissez un modèle NVIDIA :

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Si vous passez toujours `--token`, n'oubliez pas qu'il apparaît dans l'historique du shell et la sortie de `ps` ; préférez la variable d'environnement autant que possible.

## Extrait de configuration

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Identifiants de modèles

- `nvidia/llama-3.1-nemotron-70b-instruct` (par défaut)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Notes

- Endpoint `/v1` compatible OpenAI ; utilisez une clé API de NVIDIA NGC.
- Le fournisseur s'active automatiquement lorsque `NVIDIA_API_KEY` est défini ; utilisé des valeurs par défaut statiques (fenêtre de contexte de 131 072 tokens, maximum de 4 096 tokens).
