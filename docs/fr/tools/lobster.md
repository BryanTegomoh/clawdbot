---
title: Lobster
summary: "Runtime de workflow typé pour OpenClaw avec portes d'approbation reprenables."
description: Runtime de workflow typé pour OpenClaw — pipelines composables avec portes d'approbation.
read_when:
  - Vous voulez des workflows multi-étapes déterministes avec des approbations explicites
  - Vous devez reprendre un workflow sans relancer les étapes précédentes
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/lobster.md
  workflow: manual
---

# Lobster

Lobster est un shell de workflow qui permet à OpenClaw d'exécuter des séquences d'outils multi-étapes comme une seule opération déterministe avec des points de contrôle d'approbation explicites.

## Accroche

Votre assistant peut construire les outils qui le gèrent lui-même. Demandez un workflow, et 30 minutes plus tard vous avez un CLI plus des pipelines qui s'exécutent en un seul appel. Lobster est la pièce manquante : des pipelines déterministes, des approbations explicites et un état pouvant être repris.

## Pourquoi

Aujourd'hui, les workflows complexes nécessitent de nombreux allers-retours d'appels d'outils. Chaque appel coûte des tokens, et le LLM doit orchestrer chaque étape. Lobster déplace cette orchestration dans un runtime typé :

- **Un appel au lieu de plusieurs** : OpenClaw exécute un seul appel d'outil Lobster et obtient un résultat structuré.
- **Approbations intégrées** : Les effets de bord (envoyer un email, poster un commentaire) suspendent le workflow jusqu'à approbation explicite.
- **Reprise possible** : Les workflows suspendus retournent un token ; approuvez et reprenez sans tout relancer.

## Pourquoi un DSL plutôt que de simples programmes ?

Lobster est intentionnellement petit. L'objectif n'est pas « un nouveau langage », c'est une spécification de pipeline prédictible et compatible IA avec des approbations de première classe et des tokens de reprise.

- **Approve/résumé est intégré** : Un programme normal peut solliciter un humain, mais il ne peut pas _se mettre en pause et reprendre_ avec un token durable sans que vous inventiez ce runtime vous-même.
- **Déterminisme + auditabilité** : Les pipelines sont des données, donc faciles à journaliser, comparer, rejouer et réviser.
- **Surface contrainte pour l'IA** : Une grammaire minimale + du piping JSON réduit les chemins de code « créatifs » et rend la validation réaliste.
- **Politique de sécurité intégrée** : Les timeouts, les limités de sortie, les vérifications de bac à sable et les listes d'autorisation sont appliqués par le runtime, pas par chaque script.
- **Toujours programmable** : Chaque étape peut appeler n'importe quel CLI ou script. Si vous voulez du JS/TS, générez des fichiers `.lobster` depuis le code.

## Comment ça fonctionne

OpenClaw lance le CLI local `lobster` en **mode outil** et parse une enveloppe JSON depuis stdout.
Si le pipeline se met en pause pour approbation, l'outil retourne un `resumeToken` pour que vous puissiez continuer plus tard.

## Pattern : petit CLI + pipes JSON + approbations

Construisez de petites commandes qui parlent JSON, puis chaînez-les en un seul appel Lobster. (Noms de commandes d'exemple ci-dessous — remplacez par les vôtres.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Si le pipeline demande une approbation, reprenez avec le token :

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

L'IA déclenche le workflow ; Lobster exécute les étapes. Les portes d'approbation gardent les effets de bord explicites et auditables.

Exemple : mapper des éléments d'entrée vers des appels d'outils :

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## Étapes LLM JSON uniquement (llm-task)

Pour les workflows qui nécessitent une **étape LLM structurée**, activez l'outil plugin optionnel `llm-task` et appelez-le depuis Lobster. Cela garde le workflow déterministe tout en vous permettant de classifier/résumer/rédiger avec un modèle.

Activer l'outil :

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
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

L'utiliser dans un pipeline :

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
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

Voir [LLM Task](/tools/llm-task) pour les détails et les options de configuration.

## Fichiers de workflow (.lobster)

Lobster peut exécuter des fichiers de workflow YAML/JSON avec les champs `name`, `args`, `steps`, `env`, `condition` et `approval`. Dans les appels d'outils OpenClaw, définissez `pipeline` sur le chemin du fichier.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Notes :

- `stdin: $step.stdout` et `stdin: $step.json` passent la sortie d'une étape précédente.
- `condition` (ou `when`) peut conditionner des étapes sur `$step.approved`.

## Installer Lobster

Installez le CLI Lobster sur le **même hôte** qui exécute le Gateway OpenClaw (voir le [dépôt Lobster](https://github.com/openclaw/lobster)), et assurez-vous que `lobster` est dans le `PATH`.

## Activer l'outil

Lobster est un outil plugin **optionnel** (non active par défaut).

Recommandé (additif, sûr) :

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Ou par agent :

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Évitez d'utiliser `tools.allow: ["lobster"]` sauf si vous avez l'intention de fonctionner en mode liste d'autorisation restrictif.

Note : les listes d'autorisation sont opt-in pour les plugins optionnels. Si votre liste d'autorisation ne nomme que des outils de plugins (comme `lobster`), OpenClaw garde les outils de base activés. Pour restreindre les outils de base, incluez aussi les outils ou groupes de base souhaités dans la liste d'autorisation.

## Exemple : Triage des emails

Sans Lobster :

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

Avec Lobster :

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Retourne une enveloppe JSON (tronquée) :

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

L'utilisateur approuve → reprise :

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Un seul workflow. Déterministe. Sûr.

## Paramètres de l'outil

### `run`

Exécuter un pipeline en mode outil.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Exécuter un fichier de workflow avec des arguments :

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### `résumé`

Continuer un workflow suspendu après approbation.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### Entrées optionnelles

- `cwd` : Répertoire de travail relatif pour le pipeline (doit rester dans le répertoire de travail du processus courant).
- `timeoutMs` : Tuer le sous-processus s'il dépasse cette durée (défaut : 20000).
- `maxStdoutBytes` : Tuer le sous-processus si stdout dépasse cette taille (défaut : 512000).
- `argsJson` : Chaîne JSON passée à `lobster run --args-json` (fichiers de workflow uniquement).

## Enveloppe de sortie

Lobster retourne une enveloppe JSON avec l'un des trois statuts :

- `ok` → terminé avec succès
- `needs_approval` → en pause ; `requiresApproval.resumeToken` est requis pour reprendre
- `cancelled` → explicitement refusé ou annulé

L'outil expose l'enveloppe à la fois dans `content` (JSON formaté) et `details` (objet brut).

## Approbations

Si `requiresApproval` est présent, inspectez le prompt et décidez :

- `approve: true` → reprendre et continuer les effets de bord
- `approve: false` → annuler et finaliser le workflow

Utilisez `approve --preview-from-stdin --limit N` pour attacher un aperçu JSON aux demandes d'approbation sans assemblage jq/heredoc personnalisé. Les tokens de reprise sont désormais compacts : Lobster stocke l'état de reprise du workflow dans son répertoire d'état et renvoie une petite clé de token.

## OpenProse

OpenProse se marie bien avec Lobster : utilisez `/prose` pour orchestrer la préparation multi-agents, puis exécutez un pipeline Lobster pour des approbations déterministes. Si un programme Prose a besoin de Lobster, autorisez l'outil `lobster` pour les sous-agents via `tools.subagents.tools`. Voir [OpenProse](/prose).

## Sécurité

- **Sous-processus local uniquement** — pas d'appels réseau depuis le plugin lui-même.
- **Pas de secrets** — Lobster ne gère pas OAuth ; il appelle des outils OpenClaw qui le font.
- **Compatible bac à sable** — désactivé quand le contexte d'outil est sandboxé.
- **Renforcé** — nom d'exécutable fixe (`lobster`) dans le `PATH` ; timeouts et limités de sortie appliqués.

## Dépannage

- **`lobster subprocess timed out`** → augmentez `timeoutMs`, ou découpez un long pipeline.
- **`lobster output exceeded maxStdoutBytes`** → augmentez `maxStdoutBytes` ou réduisez la taille de sortie.
- **`lobster returned invalid JSON`** → assurez-vous que le pipeline s'exécute en mode outil et n'affiche que du JSON.
- **`lobster failed (code …)`** → exécutez le même pipeline dans un terminal pour inspecter stderr.

## En savoir plus

- [Plugins](/tools/plugin)
- [Authoring d'outils plugin](/plugins/agent-tools)

## Étude de cas : workflows communautaires

Un exemple public : un CLI « second cerveau » + des pipelines Lobster qui gèrent trois vaults Markdown (personnel, partenaire, partagé). Le CLI émet du JSON pour les statistiques, les listes de boîte de réception et les scans de contenu périmé ; Lobster chaîne ces commandes en workflows comme `weekly-review`, `inbox-triage`, `memory-consolidation` et `shared-task-sync`, chacun avec des portes d'approbation. L'IA gère le jugement (catégorisation) quand elle est disponible et se rabat sur des règles déterministes sinon.

- Thread : [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
- Dépôt : [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)
