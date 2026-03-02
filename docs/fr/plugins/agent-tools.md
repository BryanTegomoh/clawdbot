---
summary: "Écrire des outils d'agent dans un plugin (schemas, outils optionnels, listes d'autorisation)"
read_when:
  - Vous souhaitez ajouter un nouvel outil d'agent dans un plugin
  - Vous devez rendre un outil optionnel via des listes d'autorisation
title: "Outils d'agent de plugin"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/plugins/agent-tools.md
  workflow: manual
---

# Outils d'agent de plugin

Les plugins OpenClaw peuvent enregistrer des **outils d'agent** (fonctions a schema JSON) qui sont exposes
au LLM lors des exécutions d'agent. Les outils peuvent être **requis** (toujours disponibles) ou
**optionnels** (activation manuelle).

Les outils d'agent sont configurés sous `tools` dans la configuration principale, ou par agent sous
`agents.list[].tools`. La politique de liste d'autorisation/liste de refus contrôle quels outils l'agent
peut appeler.

## Outil de base

```ts
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## Outil optionnel (activation manuelle)

Les outils optionnels ne sont **jamais** activés automatiquement. Les utilisateurs doivent les ajouter a une
liste d'autorisation d'agent.

```ts
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

Activez les outils optionnels dans `agents.list[].tools.allow` (ou global `tools.allow`) :

```json5
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // specific tool name
            "workflow", // plugin id (enables all tools from that plugin)
            "group:plugins", // all plugin tools
          ],
        },
      },
    ],
  },
}
```

Autres paramètres de configuration affectant la disponibilité des outils :

- Les listes d'autorisation qui ne nomment que des outils de plugin sont traitees comme des activations de plugin ; les outils principaux restent
  activés sauf si vous incluez également des outils principaux ou des groupes dans la liste d'autorisation.
- `tools.profile` / `agents.list[].tools.profile` (liste d'autorisation de base)
- `tools.byProvider` / `agents.list[].tools.byProvider` (autorisation/refus spécifique au fournisseur)
- `tools.sandbox.tools.*` (politique d'outils du bac a sable lorsque le bac a sable est actif)

## Règles et conseils

- Les noms d'outils ne doivent **pas** entrer en conflit avec les noms d'outils principaux ; les outils en conflit sont ignorés.
- Les identifiants de plugin utilisés dans les listes d'autorisation ne doivent pas entrer en conflit avec les noms d'outils principaux.
- Préférez `optional: true` pour les outils qui declenchent des effets secondaires ou nécessitent des
  binaires/identifiants supplementaires.
