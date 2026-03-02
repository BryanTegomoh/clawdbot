---
summary: "Utiliser Z.AI (modèles GLM) avec OpenClaw"
read_when:
  - Vous souhaitez utiliser Z.AI / les modèles GLM dans OpenClaw
  - Vous avez besoin d'une configuration simple ZAI_API_KEY
title: "Z.AI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/zai.md
  workflow: manual
---

# Z.AI

Z.AI est la plateforme API pour les modèles **GLM**. Elle fournit des API REST pour GLM et utilise des clés API pour l'authentification. Créez votre clé API dans la console Z.AI. OpenClaw utilise le fournisseur `zai` avec une clé API Z.AI.

## Configuration CLI

```bash
openclaw onboard --auth-choice zai-api-key
# ou en mode non interactif
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Extrait de configuration

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

- Les modèles GLM sont disponibles sous la forme `zai/<model>` (exemple : `zai/glm-5`).
- `tool_stream` est activé par défaut pour le streaming d'appels d'outils Z.AI. Définissez
  `agents.defaults.models["zai/<model>"].params.tool_stream` à `false` pour le désactiver.
- Voir [/providers/glm](/providers/glm) pour l'aperçu de la famille de modèles.
- Z.AI utilisé l'authentification Bearer avec votre clé API.
