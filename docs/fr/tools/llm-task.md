---
summary: "Tâches LLM en JSON uniquement pour les workflows (outil plugin optionnel)"
read_when:
  - Vous souhaitez une étape LLM en JSON uniquement dans les workflows
  - Vous avez besoin d'une sortie LLM validée par schéma pour l'automatisation
title: "Tâche LLM"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/llm-task.md
  workflow: manual
---

# Tâche LLM

`llm-task` est un **outil plugin optionnel** qui exécute une tâche LLM en JSON uniquement et
renvoie une sortie structurée (optionnellement validée contre un JSON Schema).

C'est idéal pour les moteurs de workflow comme Lobster : vous pouvez ajouter une seule étape LLM
sans écrire de code OpenClaw personnalisé pour chaque workflow.

## Activer le plugin

1. Activez le plugin :

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2. Ajoutez l'outil à la liste d'autorisation (il est enregistré avec `optional: true`) :

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## Configuration (optionnelle)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.3-codex"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` est une liste d'autorisation de chaînes `fournisseur/modèle`. Si définie, toute requête
en dehors de la liste est rejetée.

## Paramètres de l'outil

- `prompt` (chaîne, obligatoire)
- `input` (tout type, optionnel)
- `schema` (objet, JSON Schema optionnel)
- `provider` (chaîne, optionnel)
- `model` (chaîne, optionnel)
- `authProfileId` (chaîne, optionnel)
- `temperature` (nombre, optionnel)
- `maxTokens` (nombre, optionnel)
- `timeoutMs` (nombre, optionnel)

## Sortie

Renvoie `details.json` contenant le JSON analysé (et le valide contre
`schema` lorsqu'il est fourni).

## Exemple : étape de workflow Lobster

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## Notes de sécurité

- L'outil est **JSON uniquement** et instruit le modèle de ne produire que du JSON (pas
  de blocs de code, pas de commentaire).
- Aucun outil n'est exposé au modèle pour cette exécution.
- Traitez la sortie comme non fiable sauf si vous la validez avec `schema`.
- Placez des approbations avant toute étape à effet de bord (envoi, publication, exec).
