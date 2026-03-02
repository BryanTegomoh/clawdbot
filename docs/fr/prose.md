---
summary: "OpenProse : workflows .prose, commandes slash et état dans OpenClaw"
read_when:
  - Vous souhaitez exécuter ou écrire des workflows .prose
  - Vous souhaitez activer le plugin OpenProse
  - Vous avez besoin de comprendre le stockage d'état
title: "OpenProse"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/prose.md
  workflow: manual
---

# OpenProse

OpenProse est un format de workflow portable, markdown-first, pour orchestrer des sessions IA. Dans OpenClaw, il est livré comme un plugin qui installe un pack de Skills OpenProse plus une commande slash `/prose`. Les programmes vivent dans des fichiers `.prose` et peuvent générer plusieurs sous-agents avec un flux de contrôle explicite.

Site officiel : [https://www.prose.md](https://www.prose.md)

## Ce qu'il peut faire

- Recherche multi-agents + synthèse avec parallélisme explicite.
- Workflows reproductibles et sûrs en termes d'approbation (revue de code, triage d'incidents, pipelines de contenu).
- Programmes `.prose` réutilisables que vous pouvez exécuter sur les runtimes d'agents supportés.

## Installation + activation

Les plugins bundlés sont désactivés par défaut. Activez OpenProse :

```bash
openclaw plugins enable open-prose
```

Redémarrez le Gateway après avoir activé le plugin.

Dev/checkout local : `openclaw plugins install ./extensions/open-prose`

Documentation associée : [Plugins](/tools/plugin), [Manifeste de plugin](/plugins/manifest), [Skills](/tools/skills).

## Commande slash

OpenProse enregistre `/prose` comme commande skill invocable par l'utilisateur. Elle route vers les instructions de la VM OpenProse et utilise les outils OpenClaw sous le capot.

Commandes courantes :

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Exemple : un fichier `.prose` simple

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## Emplacements des fichiers

OpenProse conserve l'état sous `.prose/` dans votre espace de travail :

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Les agents persistants au niveau utilisateur se trouvent dans :

```
~/.prose/agents/
```

## Modes d'état

OpenProse supporte plusieurs backends d'état :

- **filesystem** (par défaut) : `.prose/runs/...`
- **in-context** : transitoire, pour les petits programmes
- **sqlite** (expérimental) : nécessite le binaire `sqlite3`
- **postgres** (expérimental) : nécessite `psql` et une chaîne de connexion

Notes :

- sqlite/postgres sont opt-in et expérimentaux.
- Les identifiants postgres se retrouvent dans les logs des sous-agents ; utilisez une BD dédiée avec les moindres privilèges.

## Programmes distants

`/prose run <handle/slug>` se résout en `https://p.prose.md/<handle>/<slug>`.
Les URLs directes sont récupérées telles quelles. Cela utilise l'outil `web_fetch` (ou `exec` pour POST).

## Mapping runtime OpenClaw

Les programmes OpenProse se mappent aux primitives OpenClaw :

| Concept OpenProse             | Outil OpenClaw   |
| ----------------------------- | ---------------- |
| Spawn session / Outil Task    | `sessions_spawn` |
| Lecture/écriture de fichier   | `read` / `write` |
| Récupération web              | `web_fetch`      |

Si votre allowlist d'outils bloque ces outils, les programmes OpenProse échoueront. Voir [Configuration des Skills](/tools/skills-config).

## Sécurité + approbations

Traitez les fichiers `.prose` comme du code. Relisez-les avant de les exécuter. Utilisez les allowlists d'outils et les portes d'approbation d'OpenClaw pour contrôler les effets de bord.

Pour les workflows déterministes et gatés par approbation, comparez avec [Lobster](/tools/lobster).
