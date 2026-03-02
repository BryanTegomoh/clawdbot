---
title: "Vercel AI Gateway"
summary: "Configuration de Vercel AI Gateway (authentification + sélection de modèle)"
read_when:
  - Vous souhaitez utiliser Vercel AI Gateway avec OpenClaw
  - Vous avez besoin de la variable d'environnement de clé API ou du choix d'authentification CLI
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/vercel-ai-gateway.md
  workflow: manual
---

# Vercel AI Gateway

Le [Vercel AI Gateway](https://vercel.com/ai-gateway) fournit une API unifiée pour accéder à des centaines de modèles via un seul endpoint.

- Fournisseur : `vercel-ai-gateway`
- Authentification : `AI_GATEWAY_API_KEY`
- API : Compatible Anthropic Messages

## Démarrage rapide

1. Définissez la clé API (recommandé : stockez-la pour le Gateway) :

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Définissez un modèle par défaut :

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## Note sur l'environnement

Si le Gateway s'exécute en tant que daemon (launchd/systemd), assurez-vous que `AI_GATEWAY_API_KEY` est accessible par ce processus (par exemple, dans `~/.openclaw/.env` ou via `env.shellEnv`).

## Raccourci d'identifiant de modèle

OpenClaw accepte les raccourcis de référence de modèle Vercel Claude et les normalise à l'exécution :

- `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
- `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`
