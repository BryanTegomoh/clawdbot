---
summary: "Utilisez les sessions runtime ACP pour Pi, Claude Code, Codex, OpenCode, Gemini CLI et d'autres agents harness"
read_when:
  - Exécution de harness de codage via ACP
  - Configuration de sessions ACP liées aux threads sur les canaux compatibles
  - Dépannage du backend ACP et du câblage de plugin
  - Utilisation des commandes /acp depuis le chat
title: "Agents ACP"
---

# Agents ACP

Les sessions [Agent Client Protocol (ACP)](https://agentclientprotocol.com/) permettent à OpenClaw d'exécuter des harness de codage externes (par exemple Pi, Claude Code, Codex, OpenCode et Gemini CLI) via un plugin backend ACP.

Si vous demandez à OpenClaw en langage naturel d'"exécuter ceci dans Codex" ou de "démarrer Claude Code dans un thread", OpenClaw devrait router cette requête vers le runtime ACP (pas le runtime de sous-agent natif).

## Flux opérateur rapide

Utilisez ceci lorsque vous voulez un guide pratique `/acp` :

1. Créez une session :
   - `/acp spawn codex --mode persistent --thread auto`
2. Travaillez dans le thread lié (ou ciblez explicitement cette clé de session).
3. Vérifiez l'état du runtime :
   - `/acp status`
4. Ajustez les options du runtime selon les besoins :
   - `/acp model <provider/model>`
   - `/acp permissions <profile>`
   - `/acp timeout <seconds>`
5. Orientez une session active sans remplacer le contexte :
   - `/acp steer tighten logging and continue`
6. Arrêtez le travail :
   - `/acp cancel` (arrêter le tour en cours), ou
   - `/acp close` (fermer la session + supprimer les liaisons)

## Démarrage rapide pour les humains

Exemples de requêtes naturelles :

- "Démarre une session Codex persistante dans un thread ici et garde-la concentrée."
- "Exécute ceci comme une session ACP Claude Code en une seule fois et résume le résultat."
- "Utilise Gemini CLI pour cette tâche dans un thread, puis garde les suivis dans ce même thread."

Ce qu'OpenClaw devrait faire :

1. Choisir `runtime: "acp"`.
2. Résoudre la cible harness demandée (`agentId`, par exemple `codex`).
3. Si la liaison au thread est demandée et que le canal actuel le supporte, lier la session ACP au thread.
4. Router les messages de suivi du thread vers la même session ACP jusqu'à la perte de focus/fermeture/expiration.

## ACP versus sous-agents

Utilisez ACP lorsque vous voulez un runtime harness externe. Utilisez les sous-agents lorsque vous voulez des exécutions déléguées natives d'OpenClaw.

| Domaine       | Session ACP                           | Exécution de sous-agent              |
| ------------- | ------------------------------------- | ------------------------------------ |
| Runtime       | Plugin backend ACP (par exemple acpx) | Runtime de sous-agent natif OpenClaw |
| Clé de session| `agent:<agentId>:acp:<uuid>`          | `agent:<agentId>:subagent:<uuid>`    |
| Commandes     | `/acp ...`                            | `/subagents ...`                     |
| Outil spawn   | `sessions_spawn` avec `runtime:"acp"` | `sessions_spawn` (runtime par défaut)|

Voir aussi [Sous-agents](/tools/subagents).

## Sessions liées aux threads (agnostique du canal)

Lorsque les liaisons de thread sont activées pour un adaptateur de canal, les sessions ACP peuvent être liées aux threads :

- OpenClaw lie un thread à une session ACP cible.
- Les messages de suivi dans ce thread sont routés vers la session ACP liée.
- La sortie ACP est livrée dans le même thread.
- La perte de focus/fermeture/archivage/expiration du timeout d'inactivité ou de l'âge maximal supprime la liaison.

La prise en charge de la liaison au thread est spécifique à l'adaptateur. Si l'adaptateur de canal actif ne prend pas en charge les liaisons de thread, OpenClaw retourne un message clair de non-pris en charge/indisponible.

Drapeaux de fonctionnalité requis pour ACP lié aux threads :

- `acp.enabled=true`
- `acp.dispatch.enabled=true`
- Drapeau de spawn de thread ACP de l'adaptateur de canal activé (spécifique à l'adaptateur)
  - Discord : `channels.discord.threadBindings.spawnAcpSessions=true`

### Canaux prenant en charge les threads

- Tout adaptateur de canal qui expose la capacité de liaison session/thread.
- Prise en charge intégrée actuelle : Discord.
- Les canaux plugin peuvent ajouter la prise en charge via la même interface de liaison.

## Démarrer des sessions ACP (interfaces)

### Depuis `sessions_spawn`

Utilisez `runtime: "acp"` pour démarrer une session ACP depuis un tour d'agent ou un appel d'outil.

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

Remarques :

- `runtime` utilise par défaut `subagent`, donc définissez `runtime: "acp"` explicitement pour les sessions ACP.
- Si `agentId` est omis, OpenClaw utilise `acp.defaultAgent` lorsqu'il est configuré.
- `mode: "session"` nécessite `thread: true` pour maintenir une conversation liée persistante.

Détails de l'interface :

- `task` (requis) : invite initiale envoyée à la session ACP.
- `runtime` (requis pour ACP) : doit être `"acp"`.
- `agentId` (optionnel) : id du harness cible ACP. Se rabat sur `acp.defaultAgent` si défini.
- `thread` (optionnel, défaut `false`) : demander le flux de liaison au thread lorsque supporté.
- `mode` (optionnel) : `run` (une seule fois) ou `session` (persistante).
  - le défaut est `run`
  - si `thread: true` et mode omis, OpenClaw peut utiliser par défaut le comportement persistant selon le chemin runtime
  - `mode: "session"` nécessite `thread: true`
- `cwd` (optionnel) : répertoire de travail runtime demandé (validé par la politique du backend/runtime).
- `label` (optionnel) : libellé orienté opérateur utilisé dans le texte de session/bannière.

### Depuis la commande `/acp`

Utilisez `/acp spawn` pour le contrôle explicite de l'opérateur depuis le chat lorsque nécessaire.

```text
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

Drapeaux clés :

- `--mode persistent|oneshot`
- `--thread auto|here|off`
- `--cwd <absolute-path>`
- `--label <name>`

Voir [Commandes slash](/tools/slash-commands).

## Résolution de la cible de session

La plupart des actions `/acp` acceptent une cible de session optionnelle (`session-key`, `session-id` ou `session-label`).

Ordre de résolution :

1. Argument cible explicite (ou `--session` pour `/acp steer`)
   - essaie la clé
   - puis l'id de session en forme UUID
   - puis le libellé
2. Liaison du thread actuel (si cette conversation/thread est lié à une session ACP)
3. Repli vers la session du demandeur actuel

Si aucune cible ne se résout, OpenClaw retourne une erreur claire (`Unable to resolve session target: ...`).

## Modes de thread au spawn

`/acp spawn` prend en charge `--thread auto|here|off`.

| Mode   | Comportement                                                                                              |
| ------ | --------------------------------------------------------------------------------------------------------- |
| `auto` | Dans un thread actif : lier ce thread. Hors d'un thread : créer/lier un thread enfant lorsque supporté.   |
| `here` | Nécessite un thread actif actuel ; échoue sinon.                                                          |
| `off`  | Aucune liaison. La session démarre sans liaison.                                                          |

Remarques :

- Sur les surfaces sans liaison de thread, le comportement par défaut est effectivement `off`.
- Le spawn lié au thread nécessite la prise en charge de la politique du canal (pour Discord : `channels.discord.threadBindings.spawnAcpSessions=true`).

## Contrôles ACP

Famille de commandes disponibles :

- `/acp spawn`
- `/acp cancel`
- `/acp steer`
- `/acp close`
- `/acp status`
- `/acp set-mode`
- `/acp set`
- `/acp cwd`
- `/acp permissions`
- `/acp timeout`
- `/acp model`
- `/acp reset-options`
- `/acp sessions`
- `/acp doctor`
- `/acp install`

`/acp status` affiche les options runtime effectives et, lorsque disponibles, les identifiants de session au niveau du runtime et du backend.

Certains contrôles dépendent des capacités du backend. Si un backend ne prend pas en charge un contrôle, OpenClaw retourne une erreur claire de contrôle non supporte.

## Guide de commandes ACP

| Commande             | Ce qu'elle fait                                           | Exemple                                                        |
| -------------------- | --------------------------------------------------------- | -------------------------------------------------------------- |
| `/acp spawn`         | Crée une session ACP ; liaison de thread optionnelle.     | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel`        | Annule le tour en cours pour la session cible.            | `/acp cancel agent:codex:acp:<uuid>`                           |
| `/acp steer`         | Envoie une instruction d'orientation à la session active. | `/acp steer --session support inbox prioritize failing tests`  |
| `/acp close`         | Ferme la session et délie les cibles de thread.           | `/acp close`                                                   |
| `/acp status`        | Affiche le backend, mode, état, options, capacités.       | `/acp status`                                                  |
| `/acp set-mode`      | Définit le mode runtime pour la session cible.            | `/acp set-mode plan`                                           |
| `/acp set`           | Écriture générique d'option de config runtime.            | `/acp set model openai/gpt-5.2`                                |
| `/acp cwd`           | Définit le répertoire de travail runtime.                 | `/acp cwd /Users/user/Projects/repo`                           |
| `/acp permissions`   | Définit le profil de politique d'approbation.             | `/acp permissions strict`                                      |
| `/acp timeout`       | Définit le timeout runtime (secondes).                    | `/acp timeout 120`                                             |
| `/acp model`         | Définit le modèle runtime.                                | `/acp model anthropic/claude-opus-4-5`                         |
| `/acp reset-options` | Supprime les options runtime de session.                  | `/acp reset-options`                                           |
| `/acp sessions`      | Liste les sessions ACP récentes du magasin.               | `/acp sessions`                                                |
| `/acp doctor`        | Santé du backend, capacités, correctifs.                  | `/acp doctor`                                                  |
| `/acp install`       | Affiche les étapes d'installation déterministes.          | `/acp install`                                                 |

## Mapping des options runtime

`/acp` dispose de commandes de commodité et d'un setter générique.

Opérations équivalentes :

- `/acp model <id>` mappe vers la clé de config runtime `model`.
- `/acp permissions <profile>` mappe vers la clé de config runtime `approval_policy`.
- `/acp timeout <seconds>` mappe vers la clé de config runtime `timeout`.
- `/acp cwd <path>` met à jour directement le remplacement cwd du runtime.
- `/acp set <key> <value>` est le chemin générique.
  - Cas spécial : `key=cwd` utilise le chemin de remplacement cwd.
- `/acp reset-options` efface tous les remplacements runtime pour la session cible.

## Prise en charge du harness acpx (actuel)

Alias de harness intégrés acpx actuels :

- `pi`
- `claude`
- `codex`
- `opencode`
- `gemini`

Lorsqu'OpenClaw utilise le backend acpx, préférez ces valeurs pour `agentId` sauf si votre configuration acpx définit des alias d'agent personnalisés.

L'utilisation directe du CLI acpx peut également cibler des adaptateurs arbitraires via `--agent <command>`, mais cette échappatoire brute est une fonctionnalité du CLI acpx (pas le chemin `agentId` normal d'OpenClaw).

## Configuration requise

Configuration ACP de base :

```json5
{
  acp: {
    enabled: true,
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

La configuration de liaison au thread est spécifique à l'adaptateur de canal. Exemple pour Discord :

```json5
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

Si le spawn ACP lié au thread ne fonctionne pas, vérifiez d'abord le drapeau de fonctionnalité de l'adaptateur :

- Discord : `channels.discord.threadBindings.spawnAcpSessions=true`

Voir [Référence de configuration](/gateway/configuration-reference).

## Configuration du plugin pour le backend acpx

Installer et activer le plugin :

```bash
openclaw plugins install @openclaw/acpx
openclaw config set plugins.entries.acpx.enabled true
```

Installation dans l'espace de travail local pendant le développement :

```bash
openclaw plugins install ./extensions/acpx
```

Puis vérifiez la santé du backend :

```text
/acp doctor
```

### Stratégie d'installation acpx épinglée (comportement actuel)

`@openclaw/acpx` applique maintenant un modèle d'épinglage strict local au plugin :

1. L'extension épingle une dépendance acpx exacte dans `extensions/acpx/package.json`.
2. La commande runtime est fixée au binaire local du plugin (`extensions/acpx/node_modules/.bin/acpx`), pas au `PATH` global.
3. La config du plugin n'expose pas `command` ou `commandArgs`, donc la dérive de la commande runtime est bloquée.
4. Le démarrage enregistre immédiatement le backend ACP comme non prêt.
5. Un job de vérification en arrière-plan vérifie `acpx --version` par rapport à la version épinglée.
6. S'il est manquant/incompatible, il exécute l'installation locale du plugin (`npm install --omit=dev --no-save acpx@<pinned>`) et re-vérifie avant d'être sain.

Remarques :

- Le démarrage d'OpenClaw reste non bloquant pendant l'exécution de la vérification acpx.
- Si le réseau/l'installation échoue, le backend reste indisponible et `/acp doctor` signale un correctif applicable.

Voir [Plugins](/tools/plugin).

## Dépannage

| Symptôme                                                                | Cause probable                                 | Correctif                                                  |
| ----------------------------------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| `ACP runtime backend is not configured`                                 | Plugin backend manquant ou désactivé.          | Installez et activez le plugin backend, puis exécutez `/acp doctor`. |
| `ACP is disabled by policy (acp.enabled=false)`                         | ACP globalement désactivé.                     | Définissez `acp.enabled=true`.                             |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)`       | Dispatch depuis les messages de thread normaux désactivé. | Définissez `acp.dispatch.enabled=true`.              |
| `ACP agent "<id>" is not allowed by policy`                             | Agent absent de la liste d'autorisation.       | Utilisez un `agentId` autorisé ou mettez à jour `acp.allowedAgents`. |
| `Unable to resolve session target: ...`                                 | Mauvais token clé/id/libellé.                  | Exécutez `/acp sessions`, copiez la clé/le libellé exact, réessayez. |
| `--thread here requires running /acp spawn inside an active ... thread` | `--thread here` utilisé hors d'un contexte de thread. | Déplacez-vous vers le thread cible ou utilisez `--thread auto`/`off`. |
| `Only <user-id> can rebind this thread.`                                | Un autre utilisateur possède la liaison du thread. | Reliez en tant que propriétaire ou utilisez un thread différent. |
| `Thread bindings are unavailable for <channel>.`                        | L'adaptateur ne dispose pas de la capacité de liaison de thread. | Utilisez `--thread off` ou passez à un adaptateur/canal supporté. |
| Métadonnées ACP manquantes pour la session liée                         | Métadonnées de session ACP obsolètes/supprimées. | Recréez avec `/acp spawn`, puis reliez/focalisez le thread. |
