---
summary: "Aperçu de la famille de modèles GLM + comment l'utiliser dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles GLM dans OpenClaw
  - Vous avez besoin de la convention de nommage des modèles et de la configuration
title: "Modèles GLM"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/glm.md
  workflow: manual
---

# Modèles GLM

GLM est une **famille de modèles** (pas une entreprise) disponible via la plateforme Z.AI. Dans OpenClaw, les modèles GLM sont accessibles via le fournisseur `zai` avec des identifiants de modèle comme `zai/glm-5`.

## Configuration CLI

```bash
openclaw onboard --auth-choice zai-api-key
```

## Extrait de configuration

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

- Les versions et la disponibilité des GLM peuvent changer ; consultez la documentation de Z.AI pour les dernières informations.
- Les identifiants de modèle incluent par exemple `glm-5`, `glm-4.7` et `glm-4.6`.
- Pour les détails du fournisseur, voir [/providers/zai](/providers/zai).
