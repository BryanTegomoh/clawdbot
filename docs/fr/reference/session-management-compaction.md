---
summary: "Approfondissement : stockage de sessions + transcriptions, cycle de vie, et mécanismes internes de (auto)compaction"
read_when:
  - Vous devez déboguer les identifiants de session, les transcriptions JSONL ou les champs de sessions.json
  - Vous modifiez le comportement d'auto-compaction ou ajoutez du nettoyage « pré-compaction »
  - Vous souhaitez implémenter des vidages mémoire ou des tours système silencieux
title: "Approfondissement sur la gestion de session"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/session-management-compaction.md
  workflow: manual
---

# Gestion de session et compaction (Approfondissement)

Ce document explique comment OpenClaw gère les sessions de bout en bout :

- **Routage de session** (comment les messages entrants correspondent à un `sessionKey`)
- **Stockage de session** (`sessions.json`) et ce qu'il enregistre
- **Persistance des transcriptions** (`*.jsonl`) et leur structure
- **Hygiène des transcriptions** (corrections spécifiques au fournisseur avant les exécutions)
- **Limités de contexte** (fenêtre de contexte vs jetons suivis)
- **Compaction** (manuelle + auto-compaction) et où insérer du travail pré-compaction
- **Nettoyage silencieux** (ex. écritures mémoire qui ne devraient pas produire de sortie visible pour l'utilisateur)

Si vous souhaitez d'abord un aperçu de plus haut niveau, commencez par :

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Source de vérité : le Gateway

OpenClaw est conçu autour d'un seul **processus Gateway** qui possède l'état de session.

- Les interfaces (application macOS, Control UI web, TUI) doivent interroger le Gateway pour les listes de sessions et les compteurs de jetons.
- En mode distant, les fichiers de session sont sur l'hôte distant ; « vérifier vos fichiers Mac locaux » ne reflétera pas ce que le Gateway utilise.

---

## Deux couches de persistance

OpenClaw persiste les sessions sur deux couches :

1. **Stockage de session (`sessions.json`)**
   - Carte clé/valeur : `sessionKey -> SessionEntry`
   - Petit, modifiable, sûr à éditer (ou supprimer des entrées)
   - Enregistre les métadonnées de session (identifiant de session actuel, dernière activité, bascules, compteurs de jetons, etc.)

2. **Transcription (`<sessionId>.jsonl`)**
   - Transcription en ajout seul avec structure arborescente (les entrées ont un `id` + `parentId`)
   - Stocke la conversation réelle + appels d'outils + résumés de compaction
   - Utilisé pour reconstruire le contexte du modèle pour les tours futurs

---

## Emplacements sur le disque

Par agent, sur l'hôte Gateway :

- Stockage : `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transcriptions : `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Sessions de topics Telegram : `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw résout ces chemins via `src/config/sessions.ts`.

---

## Maintenance du stockage et contrôles disque

La persistance de session dispose de contrôles de maintenance automatiques (`session.maintenance`) pour `sessions.json` et les artefacts de transcription :

- `mode` : `warn` (défaut) ou `enforce`
- `pruneAfter` : seuil d'ancienneté des entrées obsolètes (défaut `30d`)
- `maxEntries` : plafond d'entrées dans `sessions.json` (défaut `500`)
- `rotateBytes` : rotation de `sessions.json` lorsque surdimensionné (défaut `10mb`)
- `resetArchiveRetention` : rétention pour les archives de transcription `*.reset.<timestamp>` (défaut : identique à `pruneAfter` ; `false` désactive le nettoyage)
- `maxDiskBytes` : budget optionnel pour le répertoire sessions
- `highWaterBytes` : cible optionnelle après nettoyage (défaut `80%` de `maxDiskBytes`)

Ordre d'application pour le nettoyage du budget disque (`mode: "enforce"`) :

1. Supprimer d'abord les artefacts de transcription archivés ou orphelins les plus anciens.
2. Si toujours au-dessus de la cible, évincer les entrées de session les plus anciennes et leurs fichiers de transcription.
3. Continuer jusqu'à ce que l'utilisation soit à ou en dessous de `highWaterBytes`.

En `mode: "warn"`, OpenClaw signale les évictions potentielles mais ne modifie pas le stockage/les fichiers.

Exécuter la maintenance à la demande :

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

---

## Sessions cron et journaux d'exécution

Les exécutions cron isolées créent également des entrées/transcriptions de session, et elles ont des contrôles de rétention dédiés :

- `cron.sessionRetention` (défaut `24h`) élague les anciennes sessions d'exécution cron isolées du stockage de session (`false` pour désactiver).
- `cron.runLog.maxBytes` + `cron.runLog.keepLines` élaguent les fichiers `~/.openclaw/cron/runs/<jobId>.jsonl` (défauts : `2_000_000` octets et `2000` lignes).

---

## Clés de session (`sessionKey`)

Un `sessionKey` identifié _quel compartiment de conversation_ vous utilisez (routage + isolation).

Schémas courants :

- Chat principal/direct (par agent) : `agent:<agentId>:<mainKey>` (défaut `main`)
- Groupe : `agent:<agentId>:<channel>:group:<id>`
- Salon/canal (Discord/Slack) : `agent:<agentId>:<channel>:channel:<id>` ou `...:room:<id>`
- Cron : `cron:<job.id>`
- Webhook : `hook:<uuid>` (sauf surcharge)

Les règles canoniques sont documentées dans [/concepts/session](/concepts/session).

---

## Identifiants de session (`sessionId`)

Chaque `sessionKey` pointe vers un `sessionId` actuel (le fichier de transcription qui poursuit la conversation).

Règles pratiques :

- **Réinitialisation** (`/new`, `/reset`) crée un nouveau `sessionId` pour ce `sessionKey`.
- **Réinitialisation quotidienne** (par défaut 4h00 heure locale sur l'hôte Gateway) crée un nouveau `sessionId` au prochain message après la limité de réinitialisation.
- **Expiration d'inactivité** (`session.reset.idleMinutes` ou legacy `session.idleMinutes`) crée un nouveau `sessionId` quand un message arrive après la fenêtre d'inactivité. Quand réinitialisation quotidienne + inactivité sont toutes deux configurées, celle qui expire en premier l'emporte.
- **Garde de fork de thread parent** (`session.parentForkMaxTokens`, défaut `100000`) ignoré le fork de transcription parent quand la session parente est déjà trop volumineuse ; le nouveau thread démarre à vide. Définissez `0` pour désactiver.

Détail d'implémentation : la décision se fait dans `initSessionState()` dans `src/auto-reply/reply/session.ts`.

---

## Schéma du stockage de session (`sessions.json`)

Le type de valeur du stockage est `SessionEntry` dans `src/config/sessions.ts`.

Champs clés (non exhaustif) :

- `sessionId` : identifiant de transcription actuel (le nom de fichier en est dérivé sauf si `sessionFile` est défini)
- `updatedAt` : horodatage de dernière activité
- `sessionFile` : surcharge optionnelle explicite du chemin de transcription
- `chatType` : `direct | group | room` (aide les interfaces et la politique d'envoi)
- `provider`, `subject`, `room`, `space`, `displayName` : métadonnées pour le libellé des groupes/canaux
- Bascules :
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (surcharge par session)
- Sélection de modèle :
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Compteurs de jetons (meilleur effort / dépendant du fournisseur) :
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount` : nombre de fois que l'auto-compaction s'est terminée pour cette clé de session
- `memoryFlushAt` : horodatage du dernier vidage mémoire pré-compaction
- `memoryFlushCompactionCount` : compteur de compaction lors du dernier vidage

Le stockage est sûr à éditer, mais le Gateway fait autorité : il peut réécrire ou réhydrater les entrées lors de l'exécution des sessions.

---

## Structure de transcription (`*.jsonl`)

Les transcriptions sont gérées par le `SessionManager` de `@mariozechner/pi-coding-agent`.

Le fichier est en JSONL :

- Première ligne : en-tête de session (`type: "session"`, inclut `id`, `cwd`, `timestamp`, optionnel `parentSession`)
- Ensuite : entrées de session avec `id` + `parentId` (arborescence)

Types d'entrée notables :

- `message` : messages utilisateur/assistant/toolResult
- `custom_message` : messages injectés par extension qui _entrent_ dans le contexte du modèle (peuvent être masqués de l'interface)
- `custom` : état d'extension qui _n'entre pas_ dans le contexte du modèle
- `compaction` : résumé de compaction persisté avec `firstKeptEntryId` et `tokensBefore`
- `branch_summary` : résumé persisté lors de la navigation dans une branche d'arbre

OpenClaw ne « corrige » intentionnellement **pas** les transcriptions ; le Gateway utilise `SessionManager` pour les lire/écrire.

---

## Fenêtres de contexte vs jetons suivis

Deux concepts différents comptent :

1. **Fenêtre de contexte du modèle** : limité stricte par modèle (jetons visibles par le modèle)
2. **Compteurs du stockage de session** : statistiques cumulées écrites dans `sessions.json` (utilisées pour /status et les tableaux de bord)

Si vous ajustez les limités :

- La fenêtre de contexte provient du catalogue de modèles (et peut être surchargée via la configuration).
- `contextTokens` dans le stockage est une valeur estimée/de reporting à l'exécution ; ne la traitez pas comme une garantie stricte.

Pour en savoir plus, voir [/token-use](/reference/token-use).

---

## Compaction : de quoi il s'agit

La compaction résume les anciens échanges en une entrée `compaction` persistée dans la transcription et conserve les messages récents intacts.

Après compaction, les tours futurs voient :

- Le résumé de compaction
- Les messages après `firstKeptEntryId`

La compaction est **persistante** (contrairement à l'élagage de session). Voir [/concepts/session-pruning](/concepts/session-pruning).

---

## Quand l'auto-compaction se déclenche (runtime Pi)

Dans l'agent Pi intégré, l'auto-compaction se déclenche dans deux cas :

1. **Récupération de dépassement** : le modèle retourne une erreur de dépassement de contexte → compaction → nouvelle tentative.
2. **Maintenance de seuil** : après un tour réussi, quand :

`contextTokens > contextWindow - reserveTokens`

Où :

- `contextWindow` est la fenêtre de contexte du modèle
- `reserveTokens` est la marge réservée pour les prompts + la prochaine sortie du modèle

Ce sont des sémantiques du runtime Pi (OpenClaw consomme les événements, mais Pi décide quand compacter).

---

## Paramètres de compaction (`reserveTokens`, `keepRecentTokens`)

Les paramètres de compaction de Pi se trouvent dans les réglages Pi :

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw applique également un plancher de sécurité pour les exécutions intégrées :

- Si `compaction.reserveTokens < reserveTokensFloor`, OpenClaw le relève.
- Le plancher par défaut est de `20000` jetons.
- Définissez `agents.defaults.compaction.reserveTokensFloor: 0` pour désactiver le plancher.
- S'il est déjà plus élevé, OpenClaw le laisse tel quel.

Pourquoi : laisser suffisamment de marge pour le « nettoyage » multi-tours (comme les écritures mémoire) avant que la compaction ne devienne inévitable.

Implémentation : `ensurePiCompactionReserveTokens()` dans `src/agents/pi-settings.ts`
(appelé depuis `src/agents/pi-embedded-runner.ts`).

---

## Surfaces visibles par l'utilisateur

Vous pouvez observer la compaction et l'état de session via :

- `/status` (dans toute session de chat)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Mode verbeux : `🧹 Auto-compaction complète` + compteur de compaction

---

## Nettoyage silencieux (`NO_REPLY`)

OpenClaw prend en charge les tours « silencieux » pour les tâches d'arrière-plan où l'utilisateur ne devrait pas voir la sortie intermédiaire.

Convention :

- L'assistant commence sa sortie par `NO_REPLY` pour indiquer « ne pas délivrer de réponse à l'utilisateur ».
- OpenClaw supprime/masque cela dans la couche de livraison.

Depuis `2026.1.10`, OpenClaw supprime également le **streaming de brouillon/saisie** lorsqu'un fragment partiel commence par `NO_REPLY`, afin que les opérations silencieuses ne laissent pas fuiter de sortie partielle en cours de tour.

---

## Vidage mémoire pré-compaction (implémenté)

Objectif : avant que l'auto-compaction ne se produise, exécuter un tour agentic silencieux qui écrit l'état durable sur le disque (ex. `memory/YYYY-MM-DD.md` dans l'espace de travail de l'agent) afin que la compaction ne puisse pas effacer le contexte critique.

OpenClaw utilise l'approche de **vidage pré-seuil** :

1. Surveiller l'utilisation du contexte de session.
2. Quand elle franchit un « seuil souple » (en dessous du seuil de compaction de Pi), exécuter une directive silencieuse « écrire la mémoire maintenant » vers l'agent.
3. Utiliser `NO_REPLY` pour que l'utilisateur ne voie rien.

Configuration (`agents.defaults.compaction.memoryFlush`) :

- `enabled` (défaut : `true`)
- `softThresholdTokens` (défaut : `4000`)
- `prompt` (message utilisateur pour le tour de vidage)
- `systemPrompt` (prompt système supplémentaire ajouté pour le tour de vidage)

Notes :

- Le prompt/prompt système par défaut inclut un indice `NO_REPLY` pour supprimer la livraison.
- Le vidage s'exécute une fois par cycle de compaction (suivi dans `sessions.json`).
- Le vidage ne s'exécute que pour les sessions Pi intégrées (les backends CLI le sautent).
- Le vidage est ignoré quand l'espace de travail de session est en lecture seule (`workspaceAccess: "ro"` ou `"none"`).
- Voir [Memory](/concepts/memory) pour la disposition des fichiers de l'espace de travail et les schémas d'écriture.

Pi expose également un hook `session_before_compact` dans l'API d'extension, mais la logique de vidage d'OpenClaw réside côté Gateway aujourd'hui.

---

## Liste de vérification pour le dépannage

- Mauvaise clé de session ? Commencez par [/concepts/session](/concepts/session) et confirmez le `sessionKey` dans `/status`.
- Incohérence stockage vs transcription ? Confirmez l'hôte Gateway et le chemin du stockage via `openclaw status`.
- Compaction excessive ? Vérifiez :
  - fenêtre de contexte du modèle (trop petite)
  - paramètres de compaction (`reserveTokens` trop élevé pour la fenêtre du modèle peut provoquer une compaction anticipée)
  - gonflement des résultats d'outils : activez/ajustez l'élagage de session
- Tours silencieux qui fuient ? Confirmez que la réponse commence par `NO_REPLY` (jeton exact) et que vous êtes sur un build incluant le correctif de suppression du streaming.
