---
summary: "Outils de session d'agent pour lister les sessions, récupérer l'historique et envoyer des messages inter-sessions"
read_when:
  - Ajout ou modification des outils de session
title: "Outils de session"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/session-tool.md
  workflow: manual
---

# Outils de session

Objectif : un petit ensemble d'outils difficiles à mal utiliser pour que les agents puissent lister les sessions, récupérer l'historique et envoyer des messages à une autre session.

## Noms des outils

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

## Modèle de clés

- Le compartiment principal de chat direct est toujours la clé litterale `"main"` (résolue vers la clé principale de l'agent actuel).
- Les chats de groupe utilisent `agent:<agentId>:<channel>:group:<id>` ou `agent:<agentId>:<channel>:channel:<id>` (passer la clé complète).
- Les taches cron utilisent `cron:<job.id>`.
- Les hooks utilisent `hook:<uuid>` sauf indication explicite contraire.
- Les sessions de nœuds utilisent `node-<nodeId>` sauf indication explicite contraire.

`global` et `unknown` sont des valeurs réservées et ne sont jamais listées. Si `session.scope = "global"`, nous l'aliasons à `main` pour tous les outils afin que les appelants ne voient jamais `global`.

## sessions_list

Lister les sessions sous forme de tableau de lignes.

Paramètres :

- `kinds?: string[]` filtré : parmi `"main" | "group" | "cron" | "hook" | "node" | "other"`
- `limit?: number` nombre maximum de lignes (par défaut : valeur par défaut du serveur, plafonné par ex. a 200)
- `activeMinutes?: number` uniquement les sessions mises à jour dans les N dernières minutes
- `messageLimit?: number` 0 = pas de messages (par défaut 0) ; >0 = inclure les N derniers messages

Comportement :

- `messageLimit > 0` récupère `chat.history` par session et inclut les N derniers messages.
- Les résultats d'outils sont filtrés dans la sortie de la liste ; utilisez `sessions_history` pour les messages d'outils.
- Lorsqu'elle s'exécute dans une session d'agent **en bac à sable**, les outils de session utilisent par défaut la **visibilité des sessions générées uniquement** (voir ci-dessous).

Forme de ligne (JSON) :

- `key` : clé de session (chaine)
- `kind` : `main | group | cron | hook | node | other`
- `channel` : `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
- `displayName` (libellé d'affichage du groupe si disponible)
- `updatedAt` (ms)
- `sessionId`
- `model`, `contextTokens`, `totalTokens`
- `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
- `sendPolicy` (surcharge de session si définie)
- `lastChannel`, `lastTo`
- `deliveryContext` (normalisé `{ channel, to, accountId }` lorsque disponible)
- `transcriptPath` (chemin au mieux dérivé du répertoire du store + sessionId)
- `messages?` (uniquement lorsque `messageLimit > 0`)

## sessions_history

Récupérer la transcription d'une session.

Paramètres :

- `sessionKey` (obligatoire ; accepte la clé de session ou le `sessionId` de `sessions_list`)
- `limit?: number` nombre maximum de messages (plafonné par le serveur)
- `includeTools?: boolean` (par défaut false)

Comportement :

- `includeTools=false` filtre les messages `rôle: "toolResult"`.
- Retourne un tableau de messages au format brut de la transcription.
- Lorsqu'un `sessionId` est fourni, OpenClaw le résout vers la clé de session correspondante (les identifiants manquants provoquent une erreur).

## sessions_send

Envoyer un message dans une autre session.

Paramètres :

- `sessionKey` (obligatoire ; accepte la clé de session ou le `sessionId` de `sessions_list`)
- `message` (obligatoire)
- `timeoutSeconds?: number` (par défaut >0 ; 0 = envoi sans attente)

Comportement :

- `timeoutSeconds = 0` : met en file d'attente et retourne `{ runId, status: "accepted" }`.
- `timeoutSeconds > 0` : attend jusqu'à N secondes la fin de l'exécution, puis retourne `{ runId, status: "ok", reply }`.
- Si l'attente expire : `{ runId, status: "timeout", error }`. L'exécution continue ; appelez `sessions_history` plus tard.
- Si l'exécution échoue : `{ runId, status: "error", error }`.
- Les exécutions d'annonce de livraison se font après la fin de l'exécution principale et sont au mieux ; `status: "ok"` ne garantit pas que l'annonce a été livrée.
- L'attente se fait via `agent.wait` du Gateway (côté serveur) afin que les reconnexions ne perdent pas l'attente.
- Le contexte de message agent-a-agent est injecté pour l'exécution principale.
- Les messages inter-sessions sont persistés avec `message.provenance.kind = "inter_session"` afin que les lecteurs de transcription puissent distinguer les instructions d'agent routées des saisies utilisateur externes.
- Après la fin de l'exécution principale, OpenClaw lance une **boucle de réponse** :
  - Les tours 2+ alternent entre l'agent demandeur et l'agent cible.
  - Répondez exactement `REPLY_SKIP` pour arrêter le ping-pong.
  - Le nombre maximum de tours est `session.agentToAgent.maxPingPongTurns` (0-5, par défaut 5).
- Une fois la boucle terminée, OpenClaw exécute l'**étape d'annonce agent-a-agent** (agent cible uniquement) :
  - Répondez exactement `ANNOUNCE_SKIP` pour rester silencieux.
  - Toute autre réponse est envoyée au canal cible.
  - L'étape d'annonce inclut la requête originale + la réponse du tour 1 + la dernière réponse ping-pong.

## Champ canal

- Pour les groupes, `channel` est le canal enregistré sur l'entrée de session.
- Pour les chats directs, `channel` est mappé depuis `lastChannel`.
- Pour cron/hook/nœud, `channel` est `internal`.
- Si absent, `channel` est `unknown`.

## Sécurité / Politique d'envoi

Blocage basé sur la politique par canal/type de chat (pas par identifiant de session).

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

Surcharge à l'exécution (par entrée de session) :

- `sendPolicy: "allow" | "deny"` (non défini = hériter de la configuration)
- Configurable via `sessions.patch` ou `/send on|off|inherit` (message autonome, propriétaire uniquement).

Points d'application :

- `chat.send` / `agent` (Gateway)
- Logique de livraison de réponse automatique

## sessions_spawn

Lancer une exécution de sous-agent dans une session isolée et annoncer le résultat dans le canal de chat du demandeur.

Paramètres :

- `task` (obligatoire)
- `label?` (optionnel ; utilisé pour les logs/l'interface)
- `agentId?` (optionnel ; lancer sous un autre identifiant d'agent si autorisé)
- `model?` (optionnel ; surcharge le modèle du sous-agent ; les valeurs invalides provoquent une erreur)
- `thinking?` (optionnel ; surcharge le niveau de réflexion pour l'exécution du sous-agent)
- `runTimeoutSeconds?` (par défaut `agents.defaults.subagents.runTimeoutSeconds` lorsque défini, sinon `0` ; lorsque défini, interrompt l'exécution du sous-agent après N secondes)
- `thread?` (par défaut false ; demander un routage lié au fil pour ce spawn lorsque supporté par le canal/plugin)
- `mode?` (`run|session` ; par défaut `run`, mais par défaut `session` lorsque `thread=true` ; `mode="session"` nécessite `thread=true`)
- `cleanup?` (`delete|keep`, par défaut `keep`)

Liste d'autorisation :

- `agents.list[].subagents.allowAgents` : liste des identifiants d'agents autorisés via `agentId` (`["*"]` pour autoriser tous). Par défaut : uniquement l'agent demandeur.

Découverte :

- Utilisez `agents_list` pour découvrir quels identifiants d'agents sont autorisés pour `sessions_spawn`.

Comportement :

- Démarre une nouvelle session `agent:<agentId>:subagent:<uuid>` avec `deliver: false`.
- Les sous-agents utilisent par défaut l'ensemble complet d'outils **moins les outils de session** (configurable via `tools.subagents.tools`).
- Les sous-agents ne sont pas autorisés a appeler `sessions_spawn` (pas de génération sous-agent vers sous-agent).
- Toujours non bloquant : retourne `{ status: "accepted", runId, childSessionKey }` immédiatement.
- Avec `thread=true`, les plugins de canal peuvent lier la livraison/le routage a une cible de fil (le support Discord est contrôlé par `session.threadBindings.*` et `channels.discord.threadBindings.*`).
- Après la fin, OpenClaw exécute une **étape d'annonce** du sous-agent et publie le résultat dans le canal de chat du demandeur.
  - Si la réponse finale de l'assistant est vide, le dernier `toolResult` de l'historique du sous-agent est inclus comme `Result`.
- Répondez exactement `ANNOUNCE_SKIP` pendant l'étape d'annonce pour rester silencieux.
- Les réponses d'annonce sont normalisées en `Status`/`Result`/`Notes` ; `Status` provient du résultat de l'exécution (pas du texte du modèle).
- Les sessions de sous-agents sont auto-archivées après `agents.defaults.subagents.archiveAfterMinutes` (par défaut : 60).
- Les réponses d'annonce incluent une ligne de statistiques (temps d'exécution, tokens, sessionKey/sessionId, chemin de la transcription et coût optionnel).

## Visibilité des sessions en bac à sable

Les outils de session peuvent être délimités pour réduire l'accès inter-sessions.

Comportement par défaut :

- `tools.sessions.visibility` est par défaut `tree` (session actuelle + sessions de sous-agents générées).
- Pour les sessions en bac à sable, `agents.defaults.sandbox.sessionToolsVisibility` peut plafonnér la visibilité.

Configuration :

```json5
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // par défaut : "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // par défaut : "spawned"
        sessionToolsVisibility: "spawned", // ou "all"
      },
    },
  },
}
```

Notes :

- `self` : uniquement la clé de session actuelle.
- `tree` : session actuelle + sessions générées par la session actuelle.
- `agent` : toute session appartenant à l'identifiant d'agent actuel.
- `all` : toute session (l'accès inter-agents nécessite toujours `tools.agentToAgent`).
- Lorsqu'une session est en bac à sable et `sessionToolsVisibility="spawned"`, OpenClaw plafonné la visibilité a `tree` même si vous définissez `tools.sessions.visibility="all"`.
