---
summary: "Backends CLI : repli texte seul via des CLI IA locaux"
read_when:
  - Vous souhaitez un repli fiable quand les fournisseurs API échouent
  - Vous exécutez le CLI Claude Code ou d'autres CLI IA locaux et souhaitez les réutiliser
  - Vous avez besoin d'un chemin texte seul, sans outils, qui prend en charge les sessions et les images
title: "Backends CLI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/cli-backends.md
  workflow: manual
---

# Backends CLI (runtime de repli)

OpenClaw peut exécuter des **CLI IA locaux** comme **repli texte seul** lorsque les fournisseurs API sont hors service, limités en débit ou temporairement défaillants. C'est intentionnellement conservateur :

- **Les outils sont désactivés** (pas d'appels d'outils).
- **Texte en entrée → texte en sortie** (fiable).
- **Les sessions sont prises en charge** (les tours suivants restent cohérents).
- **Les images peuvent être transmises** si le CLI accepte les chemins d'images.

Ceci est conçu comme un **filet de sécurité** plutôt qu'un chemin principal. Utilisez-le lorsque vous souhaitez des réponses texte « toujours fonctionnelles » sans dépendre d'API externes.

## Démarrage rapide pour débutants

Vous pouvez utiliser le CLI Claude Code **sans aucune configuration** (OpenClaw inclut un défaut intégré) :

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

Le CLI Codex fonctionne aussi directement :

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.3-codex
```

Si votre gateway fonctionne sous launchd/systemd et que le PATH est minimal, ajoutez simplement le chemin de la commande :

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

C'est tout. Pas de clés ni de configuration d'authentification supplémentaire au-delà du CLI lui-même.

## Utilisation comme repli

Ajoutez un backend CLI à votre liste de repli pour qu'il ne s'exécute que lorsque les modèles principaux échouent :

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/opus-4.6", "claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/opus-4.6": {},
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

Notes :

- Si vous utilisez `agents.defaults.models` (liste autorisée), vous devez inclure `claude-cli/...`.
- Si le fournisseur principal échoue (authentification, limités de débit, timeouts), OpenClaw essaiera le backend CLI ensuite.

## Aperçu de la configuration

Tous les backends CLI se trouvent sous :

```
agents.defaults.cliBackends
```

Chaque entrée est indexée par un **identifiant de fournisseur** (par exemple `claude-cli`, `my-cli`). L'identifiant de fournisseur devient la partie gauche de votre référence de modèle :

```
<fournisseur>/<modèle>
```

### Exemple de configuration

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-6": "opus",
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## Fonctionnement

1. **Sélectionne un backend** selon le préfixe du fournisseur (`claude-cli/...`).
2. **Construit un prompt système** en utilisant le même prompt OpenClaw + contexte de l'espace de travail.
3. **Exécute le CLI** avec un identifiant de session (si pris en charge) pour que l'historique reste cohérent.
4. **Analyse la sortie** (JSON ou texte brut) et renvoie le texte final.
5. **Persiste les identifiants de session** par backend, de sorte que les tours suivants réutilisent la même session CLI.

## Sessions

- Si le CLI prend en charge les sessions, définissez `sessionArg` (par exemple `--session-id`) ou `sessionArgs` (placeholder `{sessionId}`) lorsque l'identifiant doit être inséré dans plusieurs flags.
- Si le CLI utilisé une **sous-commande de reprise** avec des flags différents, définissez `resumeArgs` (remplace `args` lors de la reprise) et optionnellement `resumeOutput` (pour les reprises non-JSON).
- `sessionMode` :
  - `always` : toujours envoyer un identifiant de session (nouvel UUID si aucun n'est stocké).
  - `existing` : envoyer un identifiant de session uniquement si un a été stocké précédemment.
  - `none` : ne jamais envoyer d'identifiant de session.

## Images (transmission)

Si votre CLI accepte les chemins d'images, définissez `imageArg` :

```json5
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw écrira les images base64 dans des fichiers temporaires. Si `imageArg` est défini, ces chemins sont passés comme arguments CLI. Si `imageArg` est absent, OpenClaw ajoute les chemins de fichiers au prompt (injection de chemin), ce qui suffit pour les CLI qui chargent automatiquement les fichiers locaux depuis des chemins bruts (comportement du CLI Claude Code).

## Entrées / sorties

- `output: "json"` (défaut) essaie de parser le JSON et d'extraire le texte + l'identifiant de session.
- `output: "jsonl"` parse les flux JSONL (CLI Codex `--json`) et extrait le dernier message agent plus `thread_id` lorsqu'il est présent.
- `output: "text"` traite stdout comme la réponse finale.

Modes d'entrée :

- `input: "arg"` (défaut) passe le prompt comme dernier argument CLI.
- `input: "stdin"` envoie le prompt via stdin.
- Si le prompt est très long et que `maxPromptArgChars` est défini, stdin est utilisé.

## Valeurs par défaut (intégrées)

OpenClaw inclut un défaut pour `claude-cli` :

- `command: "claude"`
- `args: ["-p", "--output-format", "json", "--dangerously-skip-permissions"]`
- `resumeArgs: ["-p", "--output-format", "json", "--dangerously-skip-permissions", "--résumé", "{sessionId}"]`
- `modelArg: "--model"`
- `systemPromptArg: "--append-system-prompt"`
- `sessionArg: "--session-id"`
- `systemPromptWhen: "first"`
- `sessionMode: "always"`

OpenClaw inclut également un défaut pour `codex-cli` :

- `command: "codex"`
- `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `resumeArgs: ["exec","résumé","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
- `output: "jsonl"`
- `resumeOutput: "text"`
- `modelArg: "--model"`
- `imageArg: "--image"`
- `sessionMode: "existing"`

Ne remplacez que si nécessaire (courant : chemin absolu de `command`).

## Limitations

- **Pas d'outils OpenClaw** (le backend CLI ne reçoit jamais d'appels d'outils). Certains CLI peuvent toutefois exécuter leur propre outillage agent.
- **Pas de streaming** (la sortie CLI est collectée puis renvoyée).
- **Les sorties structurées** dépendent du format JSON du CLI.
- **Les sessions CLI Codex** reprennent via la sortie texte (pas JSONL), ce qui est moins structuré que l'exécution initiale `--json`. Les sessions OpenClaw fonctionnent toujours normalement.

## Dépannage

- **CLI introuvable** : définissez `command` avec un chemin complet.
- **Nom de modèle incorrect** : utilisez `modelAliases` pour mapper `fournisseur/modèle` → modèle CLI.
- **Pas de continuité de session** : assurez-vous que `sessionArg` est défini et que `sessionMode` n'est pas `none` (le CLI Codex ne peut actuellement pas reprendre avec une sortie JSON).
- **Images ignorées** : définissez `imageArg` (et vérifiez que le CLI prend en charge les chemins de fichiers).
