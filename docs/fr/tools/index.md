---
summary: "Surface d'outils agent pour OpenClaw (navigateur, canvas, nodes, message, cron) remplaçant les anciens skills `openclaw-*`"
read_when:
  - Ajout ou modification d'outils agent
  - Retrait ou modification des skills `openclaw-*`
title: "Outils"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/index.md
  workflow: manual
---

# Outils (OpenClaw)

OpenClaw expose des **outils agent de première classe** pour le navigateur, le canvas, les nodes et le cron.
Ceux-ci remplacent les anciens skills `openclaw-*` : les outils sont typés, sans passage par le shell,
et l'agent doit s'appuyer directement sur eux.

## Désactivation des outils

Vous pouvez globalement autoriser/interdire des outils via `tools.allow` / `tools.deny` dans `openclaw.json`
(deny l'emporte). Cela empêche les outils non autorisés d'être envoyés aux fournisseurs de modèles.

```json5
{
  tools: { deny: ["browser"] },
}
```

Notes :

- La correspondance est insensible à la casse.
- Les jokers `*` sont pris en charge (`"*"` signifie tous les outils).
- Si `tools.allow` ne référence que des noms d'outils de plugins inconnus ou non chargés, OpenClaw journalise un avertissement et ignore la liste blanche afin que les outils principaux restent disponibles.

## Profils d'outils (liste blanche de base)

`tools.profile` définit une **liste blanche de base** avant `tools.allow`/`tools.deny`.
Surcharge par agent : `agents.list[].tools.profile`.

Profils :

- `minimal` : `session_status` uniquement
- `coding` : `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging` : `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full` : aucune restriction (identique à non défini)

Exemple (messagerie uniquement par défaut, autoriser aussi les outils Slack + Discord) :

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Exemple (profil coding, mais interdire exec/process partout) :

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

Exemple (profil coding global, agent support en messagerie uniquement) :

```json5
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```

## Politique d'outils spécifique au fournisseur

Utilisez `tools.byProvider` pour **restreindre davantage** les outils pour des fournisseurs spécifiques
(ou un seul `provider/model`) sans modifier vos valeurs par défaut globales.
Surcharge par agent : `agents.list[].tools.byProvider`.

Ceci est appliqué **après** le profil d'outils de base et **avant** les listes allow/deny,
donc il ne peut que réduire l'ensemble d'outils.
Les clés de fournisseur acceptent soit `provider` (ex. `google-antigravity`) soit
`provider/model` (ex. `openai/gpt-5.2`).

Exemple (conserver le profil coding global, mais outils minimaux pour Google Antigravity) :

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Exemple (liste blanche spécifique provider/model pour un endpoint instable) :

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

Exemple (surcharge spécifique à un agent pour un seul fournisseur) :

```json5
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```

## Groupes d'outils (raccourcis)

Les politiques d'outils (globales, par agent, bac à sable) prennent en charge les entrées `group:*` qui se développent en plusieurs outils.
Utilisez-les dans `tools.allow` / `tools.deny`.

Groupes disponibles :

- `group:runtime` : `exec`, `bash`, `process`
- `group:fs` : `read`, `write`, `edit`, `apply_patch`
- `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory` : `memory_search`, `memory_get`
- `group:web` : `web_search`, `web_fetch`
- `group:ui` : `browser`, `canvas`
- `group:automation` : `cron`, `gateway`
- `group:messaging` : `message`
- `group:nodes` : `nodes`
- `group:openclaw` : tous les outils OpenClaw intégrés (exclut les plugins de fournisseurs)

Exemple (autoriser uniquement les outils fichier + navigateur) :

```json5
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

## Plugins + outils

Les plugins peuvent enregistrer des **outils supplémentaires** (et des commandes CLI) au-delà de l'ensemble principal.
Voir [Plugins](/tools/plugin) pour l'installation + la configuration, et [Skills](/tools/skills) pour la manière dont
les indications d'utilisation des outils sont injectées dans les prompts. Certains plugins fournissent leurs propres skills
en plus des outils (par exemple, le plugin d'appel vocal).

Outils de plugins optionnels :

- [Lobster](/tools/lobster) : runtime de workflows typé avec approbations pouvant être reprises (nécessite le CLI Lobster sur l'hôte du Gateway).
- [LLM Task](/tools/llm-task) : étape LLM en JSON uniquement pour une sortie structurée de workflow (validation de schéma optionnelle).

## Inventaire des outils

### `apply_patch`

Appliquer des patches structurés sur un ou plusieurs fichiers. À utiliser pour les éditions multi-hunk.
Expérimental : activer via `tools.exec.applyPatch.enabled` (modèles OpenAI uniquement).
`tools.exec.applyPatch.workspaceOnly` est défini à `true` par défaut (contenu dans le workspace). Définissez-le à `false` uniquement si vous souhaitez intentionnellement que `apply_patch` écrive/supprimé en dehors du répertoire du workspace.

### `exec`

Exécuter des commandes shell dans le workspace.

Paramètres principaux :

- `command` (obligatoire)
- `yieldMs` (passage en arrière-plan automatique après timeout, défaut 10000)
- `background` (arrière-plan immédiat)
- `timeout` (secondes ; tue le processus si dépassé, défaut 1800)
- `elevated` (bool ; exécuter sur l'hôte si le mode élevé est activé/autorisé ; ne change le comportement que lorsque l'agent est dans un bac à sable)
- `host` (`sandbox | gateway | node`)
- `security` (`deny | allowlist | full`)
- `ask` (`off | on-miss | always`)
- `node` (id/nom du node pour `host=node`)
- Besoin d'un vrai TTY ? Définissez `pty: true`.

Notes :

- Retourne `status: "running"` avec un `sessionId` en mode arrière-plan.
- Utilisez `process` pour interroger/journaliser/écrire/tuer/nettoyer les sessions en arrière-plan.
- Si `process` est interdit, `exec` s'exécute de manière synchrone et ignore `yieldMs`/`background`.
- `elevated` est conditionné par `tools.elevated` plus toute surcharge `agents.list[].tools.elevated` (les deux doivent autoriser) et est un alias pour `host=gateway` + `security=full`.
- `elevated` ne change le comportement que lorsque l'agent est dans un bac à sable (sinon c'est un no-op).
- `host=node` peut cibler une application compagnon macOS ou un hôte node headless (`openclaw node run`).
- Approbations gateway/node et listes blanches : [Approbations Exec](/tools/exec-approvals).

### `process`

Gérer les sessions exec en arrière-plan.

Actions principales :

- `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

Notes :

- `poll` retourne la nouvelle sortie et le statut de fin lorsque terminé.
- `log` prend en charge `offset`/`limit` basés sur les lignes (omettre `offset` pour récupérer les N dernières lignes).
- `process` est limité par agent ; les sessions d'autres agents ne sont pas visibles.

### `loop-détection` (garde-fous contre les boucles d'appels d'outils)

OpenClaw suit l'historique récent des appels d'outils et bloque ou avertit lorsqu'il détecte des boucles répétitives sans progression.
Activer avec `tools.loopDetection.enabled: true` (défaut : `false`).

```json5
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

- `genericRepeat` : motif d'appel répété du même outil + mêmes paramètres.
- `knownPollNoProgress` : outils de type poll répétés avec des sorties identiques.
- `pingPong` : motifs `A/B/A/B` alternés sans progression.
- Surcharge par agent : `agents.list[].tools.loopDetection`.

### `web_search`

Rechercher sur le web via l'API Brave Search.

Paramètres principaux :

- `query` (obligatoire)
- `count` (1-10 ; défaut depuis `tools.web.search.maxResults`)

Notes :

- Nécessite une clé API Brave (recommandé : `openclaw configuré --section web`, ou définir `BRAVE_API_KEY`).
- Activer via `tools.web.search.enabled`.
- Les réponses sont mises en cache (défaut 15 min).
- Voir [Outils web](/tools/web) pour la configuration.

### `web_fetch`

Récupérer et extraire le contenu lisible d'une URL (HTML vers markdown/texte).

Paramètres principaux :

- `url` (obligatoire)
- `extractMode` (`markdown` | `text`)
- `maxChars` (tronquer les pages longues)

Notes :

- Activer via `tools.web.fetch.enabled`.
- `maxChars` est plafonné par `tools.web.fetch.maxCharsCap` (défaut 50000).
- Les réponses sont mises en cache (défaut 15 min).
- Pour les sites riches en JS, préférez l'outil navigateur.
- Voir [Outils web](/tools/web) pour la configuration.
- Voir [Firecrawl](/tools/firecrawl) pour le fallback anti-bot optionnel.

### `browser`

Contrôler le navigateur dédié géré par OpenClaw.

Actions principales :

- `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
- `snapshot` (aria/ai)
- `screenshot` (retourne un bloc image + `MEDIA:<path>`)
- `act` (actions UI : click/type/press/hover/drag/select/fill/resize/wait/evaluate)
- `navigate`, `console`, `pdf`, `upload`, `dialog`

Gestion des profils :

- `profiles` — lister tous les profils de navigateur avec leur statut
- `create-profile` — créer un nouveau profil avec port auto-attribué (ou `cdpUrl`)
- `delete-profile` — arrêter le navigateur, supprimer les données utilisateur, retirer de la config (local uniquement)
- `reset-profile` — tuer le processus orphelin sur le port du profil (local uniquement)

Paramètres courants :

- `profile` (optionnel ; défaut à `browser.defaultProfile`)
- `target` (`sandbox` | `host` | `node`)
- `node` (optionnel ; choisir un id/nom de node spécifique)
  Notes :
- Nécessite `browser.enabled=true` (défaut : `true` ; mettre à `false` pour désactiver).
- Toutes les actions acceptent le paramètre optionnel `profile` pour le support multi-instances.
- Quand `profile` est omis, utilisé `browser.defaultProfile` (défaut "chrome").
- Noms de profil : minuscules alphanumériques + tirets uniquement (max 64 caractères).
- Plage de ports : 18800-18899 (environ 100 profils max).
- Les profils distants sont en mode attachement uniquement (pas de start/stop/reset).
- Si un node capable de navigateur est connecté, l'outil peut rediriger automatiquement vers lui (sauf si vous fixez `target`).
- `snapshot` utilisé par défaut `ai` quand Playwright est installé ; utilisez `aria` pour l'arbre d'accessibilité.
- `snapshot` prend aussi en charge les options de snapshot par rôle (`interactive`, `compact`, `depth`, `selector`) qui retournent des refs comme `e12`.
- `act` nécessite `ref` depuis `snapshot` (`12` numérique depuis les snapshots AI, ou `e12` depuis les snapshots par rôle) ; utilisez `evaluate` pour les besoins rares de sélecteurs CSS.
- Évitez `act` puis `wait` par défaut ; utilisez-le uniquement dans des cas exceptionnels (pas d'état UI fiable à attendre).
- `upload` peut optionnellement passer un `ref` pour cliquer automatiquement après l'armement.
- `upload` prend aussi en charge `inputRef` (ref aria) ou `élément` (sélecteur CSS) pour définir directement `<input type="file">`.

### `canvas`

Piloter le Canvas du node (présent, eval, snapshot, A2UI).

Actions principales :

- `présent`, `hide`, `navigate`, `eval`
- `snapshot` (retourne un bloc image + `MEDIA:<path>`)
- `a2ui_push`, `a2ui_reset`

Notes :

- Utilisé `node.invoke` du gateway sous le capot.
- Si aucun `node` n'est fourni, l'outil choisit un défaut (node unique connecté ou node mac local).
- A2UI est en v0.8 uniquement (pas de `createSurface`) ; le CLI rejette le JSONL v0.9 avec des erreurs de ligne.
- Test rapide : `openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"`.

### `nodes`

Découvrir et cibler les nodes appairés ; envoyer des notifications ; capturer caméra/écran.

Actions principales :

- `status`, `describe`
- `pending`, `approve`, `reject` (appairage)
- `notify` (macOS `system.notify`)
- `run` (macOS `system.run`)
- `camera_snap`, `camera_clip`, `screen_record`
- `location_get`

Notes :

- Les commandes caméra/écran nécessitent que l'application node soit au premier plan.
- Les images retournent des blocs image + `MEDIA:<path>`.
- Les vidéos retournent `FILE:<path>` (mp4).
- La localisation retourne un payload JSON (lat/lon/précision/horodatage).
- Paramètres de `run` : tableau argv `command` ; optionnel `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`.

Exemple (`run`) :

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### `image`

Analyser une image avec le modèle d'image configuré.

Paramètres principaux :

- `image` (chemin ou URL obligatoire)
- `prompt` (optionnel ; défaut "Describe the image.")
- `model` (surcharge optionnelle)
- `maxBytesMb` (limité de taille optionnelle)

Notes :

- Disponible uniquement quand `agents.defaults.imageModel` est configuré (principal ou fallbacks), ou quand un modèle d'image implicite peut être déduit de votre modèle par défaut + authentification configurée (appairage best-effort).
- Utilisé directement le modèle d'image (indépendant du modèle de chat principal).

### `message`

Envoyer des messages et des actions de canal à travers Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams.

Actions principales :

- `send` (texte + média optionnel ; MS Teams prend aussi en charge `card` pour les Adaptive Cards)
- `poll` (sondages WhatsApp/Discord/MS Teams)
- `react` / `reactions` / `read` / `edit` / `delete`
- `pin` / `unpin` / `list-pins`
- `permissions`
- `thread-create` / `thread-list` / `thread-reply`
- `search`
- `sticker`
- `member-info` / `rôle-info`
- `emoji-list` / `emoji-upload` / `sticker-upload`
- `rôle-add` / `rôle-remove`
- `channel-info` / `channel-list`
- `voice-status`
- `event-list` / `event-create`
- `timeout` / `kick` / `ban`

Notes :

- `send` route WhatsApp via le Gateway ; les autres canaux passent en direct.
- `poll` utilisé le Gateway pour WhatsApp et MS Teams ; les sondages Discord passent en direct.
- Quand un appel d'outil message est lié à une session de chat activé, les envois sont contraints à la cible de cette session pour éviter les fuites inter-contextes.

### `cron`

Gérer les tâches cron et les réveils du Gateway.

Actions principales :

- `status`, `list`
- `add`, `update`, `remove`, `run`, `runs`
- `wake` (mettre en file un événement système + heartbeat immédiat optionnel)

Notes :

- `add` attend un objet de tâche cron complet (même schéma que le RPC `cron.add`).
- `update` utilisé `{ jobId, patch }` (`id` accepté pour compatibilité).

### `gateway`

Redémarrer ou appliquer des mises à jour au processus Gateway en cours (sur place).

Actions principales :

- `restart` (autorisé + envoie `SIGUSR1` pour un redémarrage in-process ; `openclaw gateway` redémarrage sur place)
- `config.get` / `config.schema`
- `config.apply` (valider + écrire la config + redémarrer + réveiller)
- `config.patch` (fusionner une mise à jour partielle + redémarrer + réveiller)
- `update.run` (exécuter la mise à jour + redémarrer + réveiller)

Notes :

- Utilisez `delayMs` (défaut 2000) pour éviter d'interrompre une réponse en cours.
- `restart` est activé par défaut ; définissez `commands.restart: false` pour le désactiver.

### `sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `session_status`

Lister les sessions, inspecter l'historique de la transcription, ou envoyer à une autre session.

Paramètres principaux :

- `sessions_list` : `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = aucun)
- `sessions_history` : `sessionKey` (ou `sessionId`), `limit?`, `includeTools?`
- `sessions_send` : `sessionKey` (ou `sessionId`), `message`, `timeoutSeconds?` (0 = fire-and-forget)
- `sessions_spawn` : `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`
- `session_status` : `sessionKey?` (défaut courant ; accepte `sessionId`), `model?` (`default` efface la surcharge)

Notes :

- `main` est la clé de chat direct canonique ; global/unknown sont masqués.
- `messageLimit > 0` récupère les N derniers messages par session (messages d'outils filtrés).
- Le ciblage de session est contrôlé par `tools.sessions.visibility` (défaut `tree` : session courante + sessions de sous-agents générés). Si vous exécutez un agent partagé pour plusieurs utilisateurs, envisagez de définir `tools.sessions.visibility: "self"` pour empêcher la navigation inter-sessions.
- `sessions_send` attend la fin de l'exécution quand `timeoutSeconds > 0`.
- La livraison/annonce se produit après l'achèvement et est best-effort ; `status: "ok"` confirme que l'exécution de l'agent est terminée, pas que l'annonce a été livrée.
- `sessions_spawn` supporte `runtime: "subagent" | "acp"` (`subagent` par défaut). Pour le comportement du runtime ACP, voir [Agents ACP](/tools/acp-agents).
- `sessions_spawn` démarre un sous-agent et publie une réponse d'annonce au chat demandeur.
  - Prend en charge le mode one-shot (`mode: "run"`) et le mode persistant lié à un thread (`mode: "session"` avec `thread: true`).
  - Si `thread: true` et `mode` est omis, le mode par défaut est `session`.
  - `mode: "session"` nécessite `thread: true`.
  - Si `runTimeoutSeconds` est omis, OpenClaw utilise `agents.defaults.subagents.runTimeoutSeconds` quand défini ; sinon le timeout par défaut est `0` (pas de timeout).
  - Les flux liés aux threads Discord dépendent de `session.threadBindings.*` et `channels.discord.threadBindings.*`.
  - Le format de réponse inclut `Status`, `Result` et des statistiques compactes.
  - `Result` est le texte de complétion de l'assistant ; si absent, le dernier `toolResult` est utilisé en fallback.
- Les lancements en mode complétion manuelle envoient d'abord directement, avec fallback en file d'attente et retry sur les erreurs transitoires (`status: "ok"` signifie que l'exécution est terminée, pas que l'annonce a été livrée).
- `sessions_spawn` est non-bloquant et retourne `status: "accepted"` immédiatement.
- `sessions_send` exécute un ping-pong de réponse (répondre `REPLY_SKIP` pour arrêter ; max de tours via `session.agentToAgent.maxPingPongTurns`, 0-5).
- Après le ping-pong, l'agent cible exécute une **étape d'annonce** ; répondre `ANNOUNCE_SKIP` pour supprimer l'annonce.
- Restriction bac à sable : quand la session courante est dans un bac à sable et `agents.defaults.sandbox.sessionToolsVisibility: "spawned"`, OpenClaw restreint `tools.sessions.visibility` à `tree`.

### `agents_list`

Lister les ids d'agents que la session courante peut cibler avec `sessions_spawn`.

Notes :

- Le résultat est restreint aux listes blanches par agent (`agents.list[].subagents.allowAgents`).
- Quand `["*"]` est configuré, l'outil inclut tous les agents configurés et marque `allowAny: true`.

## Paramètres (communs)

Outils supportés par le Gateway (`canvas`, `nodes`, `cron`) :

- `gatewayUrl` (défaut `ws://127.0.0.1:18789`)
- `gatewayToken` (si l'authentification est activée)
- `timeoutMs`

Note : quand `gatewayUrl` est défini, incluez `gatewayToken` explicitement. Les outils n'héritent pas de la config
ni des identifiants d'environnement pour les surcharges, et des identifiants explicites manquants provoquent une erreur.

Outil navigateur :

- `profile` (optionnel ; défaut à `browser.defaultProfile`)
- `target` (`sandbox` | `host` | `node`)
- `node` (optionnel ; fixer un id/nom de node spécifique)

## Flux d'agent recommandés

Automatisation du navigateur :

1. `browser` puis `status` / `start`
2. `snapshot` (ai ou aria)
3. `act` (click/type/press)
4. `screenshot` si vous avez besoin d'une confirmation visuelle

Rendu canvas :

1. `canvas` puis `présent`
2. `a2ui_push` (optionnel)
3. `snapshot`

Ciblage de node :

1. `nodes` puis `status`
2. `describe` sur le node choisi
3. `notify` / `run` / `camera_snap` / `screen_record`

## Sécurité

- Évitez `system.run` direct ; utilisez `nodes` puis `run` uniquement avec le consentement explicite de l'utilisateur.
- Respectez le consentement de l'utilisateur pour la capture caméra/écran.
- Utilisez `status/describe` pour vérifier les permissions avant d'invoquer les commandes média.

## Comment les outils sont présentés à l'agent

Les outils sont exposés sur deux canaux parallèles :

1. **Texte du prompt système** : une liste lisible par un humain + des indications.
2. **Schéma d'outils** : les définitions de fonctions structurées envoyées à l'API du modèle.

Cela signifie que l'agent voit à la fois "quels outils existent" et "comment les appeler". Si un outil
n'apparaît ni dans le prompt système ni dans le schéma, le modèle ne peut pas l'appeler.
