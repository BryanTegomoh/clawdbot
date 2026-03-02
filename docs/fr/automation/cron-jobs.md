---
summary: "Tâches cron + réveils pour le planificateur du Gateway"
read_when:
  - Planification de tâches de fond ou de réveils
  - Câblage d'automatisation devant fonctionner avec ou à côté des heartbeats
  - Choix entre heartbeat et cron pour les tâches planifiées
title: "Tâches cron"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/cron-jobs.md
  workflow: manual
---

# Tâches cron (planificateur du Gateway)

> **Cron ou Heartbeat ?** Consultez [Cron vs Heartbeat](/automation/cron-vs-heartbeat) pour savoir quand utiliser chacun.

Cron est le planificateur intégré du Gateway. Il persiste les tâches, réveille l'agent au bon moment et peut optionnellement renvoyer la sortie vers un chat.

Si vous voulez _"exécuter ceci chaque matin"_ ou _"solliciter l'agent dans 20 minutes"_, cron est le mécanisme à utiliser.

Dépannage : [/automation/troubleshooting](/automation/troubleshooting)

## TL;DR

- Cron s'exécute **à l'intérieur du Gateway** (pas à l'intérieur du modèle).
- Les tâches persistent sous `~/.openclaw/cron/` donc les redémarrages ne perdent pas les planifications.
- Deux styles d'exécution :
  - **Session principale** : mettre en file d'attente un événement système, puis exécuter au prochain heartbeat.
  - **Isolé** : exécuter un tour d'agent dédié dans `cron:<jobId>`, avec livraison (annonce par défaut ou aucune).
- Les réveils sont de première classe : une tâche peut demander "réveil immédiat" vs "prochain heartbeat".
- L'envoi webhook est par tâche via `delivery.mode = "webhook"` + `delivery.to = "<url>"`.
- Le repli hérité reste pour les tâches stockées avec `notify: true` quand `cron.webhook` est défini, migrez ces tâches vers le mode de livraison webhook.

## Démarrage rapide (actionnable)

Créer un rappel ponctuel, vérifier qu'il existe, puis l'exécuter immédiatement :

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

Planifier une tâche isolée récurrente avec livraison :

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## Équivalents par appel d'outil (outil cron du Gateway)

Pour les formes JSON canoniques et les exemples, consultez [Schéma JSON pour les appels d'outil](/automation/cron-jobs#json-schema-for-tool-calls).

## Où sont stockées les tâches cron

Les tâches cron sont persistées sur l'hôte du Gateway dans `~/.openclaw/cron/jobs.json` par défaut.
Le Gateway charge le fichier en mémoire et le réécrit lors des modifications, donc les éditions manuelles
ne sont sûres que lorsque le Gateway est arrêté. Préférez `openclaw cron add/edit` ou l'API d'appel d'outil cron pour les modifications.

## Présentation pour débutants

Pensez à une tâche cron comme : **quand** exécuter + **quoi** faire.

1. **Choisir une planification**
   - Rappel ponctuel : `schedule.kind = "at"` (CLI : `--at`)
   - Tâche récurrente : `schedule.kind = "every"` ou `schedule.kind = "cron"`
   - Si votre horodatage ISO omet un fuseau horaire, il est traité comme **UTC**.

2. **Choisir où cela s'exécute**
   - `sessionTarget: "main"` : exécuter pendant le prochain heartbeat avec le contexte principal.
   - `sessionTarget: "isolated"` : exécuter un tour d'agent dédié dans `cron:<jobId>`.

3. **Choisir la charge utile**
   - Session principale : `payload.kind = "systemEvent"`
   - Session isolée : `payload.kind = "agentTurn"`

Optionnel : les tâches ponctuelles (`schedule.kind = "at"`) se suppriment après succès par défaut. Mettez
`deleteAfterRun: false` pour les conserver (elles se désactiveront après succès).

## Concepts

### Tâches

Une tâche cron est un enregistrement stocké avec :

- une **planification** (quand elle doit s'exécuter),
- une **charge utile** (ce qu'elle doit faire),
- un **mode de livraison** optionnel (`announce`, `webhook`, ou `none`).
- une **liaison d'agent** optionnelle (`agentId`) : exécuter la tâche sous un agent spécifique ; si
  absent ou inconnu, le Gateway se rabat sur l'agent par défaut.

Les tâches sont identifiées par un `jobId` stable (utilisé par les APIs CLI/Gateway).
Dans les appels d'outil d'agent, `jobId` est canonique ; l'ancien `id` est accepté pour compatibilité.
Les tâches ponctuelles se suppriment automatiquement après succès par défaut ; mettez `deleteAfterRun: false` pour les conserver.

### Planifications

Cron supporte trois types de planification :

- `at` : horodatage ponctuel via `schedule.at` (ISO 8601).
- `every` : intervalle fixe (ms).
- `cron` : expression cron à 5 champs (ou 6 champs avec secondes) avec fuseau horaire IANA optionnel.

Les expressions cron utilisent `croner`. Si un fuseau horaire est omis, le fuseau horaire
local de l'hôte du Gateway est utilisé.

Pour réduire les pics de charge en début d'heure sur plusieurs gateways, OpenClaw applique une
fenêtre de décalage déterministe par tâche allant jusqu'à 5 minutes pour les expressions récurrentes
en début d'heure (par exemple `0 * * * *`, `0 */2 * * *`). Les expressions à heure fixe
comme `0 7 * * *` restent exactes.

Pour toute planification cron, vous pouvez définir une fenêtre de décalage explicite avec `schedule.staggerMs`
(`0` conserve le timing exact). Raccourcis CLI :

- `--stagger 30s` (ou `1m`, `5m`) pour définir une fenêtre de décalage explicite.
- `--exact` pour forcer `staggerMs = 0`.

### Exécution principale vs isolée

#### Tâches en session principale (événements système)

Les tâches principales mettent en file d'attente un événement système et réveillent optionnellement le moteur heartbeat.
Elles doivent utiliser `payload.kind = "systemEvent"`.

- `wakeMode: "now"` (par défaut) : l'événement déclenche une exécution heartbeat immédiate.
- `wakeMode: "next-heartbeat"` : l'événement attend le prochain heartbeat planifié.

C'est le meilleur choix quand vous voulez le prompt heartbeat normal + le contexte de session principale.
Voir [Heartbeat](/gateway/heartbeat).

#### Tâches isolées (sessions cron dédiées)

Les tâches isolées exécutent un tour d'agent dédié dans la session `cron:<jobId>`.

Comportements clés :

- Le prompt est préfixé par `[cron:<jobId> <nom de tâche>]` pour la traçabilité.
- Chaque exécution démarre un **nouvel identifiant de session** (pas de report de conversation précédente).
- Comportement par défaut : si `delivery` est omis, les tâches isolées annoncent un résumé (`delivery.mode = "announce"`).
- `delivery.mode` choisit ce qui se passe :
  - `announce` : livrer un résumé au canal cible et publier un bref résumé dans la session principale.
  - `webhook` : envoyer par POST la charge utile de l'événement terminé à `delivery.to` quand l'événement terminé inclut un résumé.
  - `none` : interne uniquement (pas de livraison, pas de résumé en session principale).
- `wakeMode` contrôle quand le résumé de session principale est publié :
  - `now` : heartbeat immédiat.
  - `next-heartbeat` : attend le prochain heartbeat planifié.

Utilisez les tâches isolées pour les corvées bruyantes, fréquentes ou "de fond" qui ne devraient pas
encombrer l'historique de votre chat principal.

### Formes de charge utile (ce qui s'exécute)

Deux types de charge utile sont supportés :

- `systemEvent` : session principale uniquement, acheminé via le prompt heartbeat.
- `agentTurn` : session isolée uniquement, exécute un tour d'agent dédié.

Champs communs de `agentTurn` :

- `message` : prompt texte requis.
- `model` / `thinking` : surcharges optionnelles (voir ci-dessous).
- `timeoutSeconds` : surcharge de timeout optionnelle.

Configuration de livraison :

- `delivery.mode` : `none` | `announce` | `webhook`.
- `delivery.channel` : `last` ou un canal spécifique.
- `delivery.to` : cible spécifique au canal (announce) ou URL webhook (mode webhook).
- `delivery.bestEffort` : éviter de mettre la tâche en échec si la livraison announce échoue.

La livraison announce supprime les envois par outil de messagerie pour l'exécution ; utilisez `delivery.channel`/`delivery.to`
pour cibler le chat à la place. Quand `delivery.mode = "none"`, aucun résumé n'est publié dans la session principale.

Si `delivery` est omis pour les tâches isolées, OpenClaw utilise par défaut `announce`.

#### Flux de livraison announce

Quand `delivery.mode = "announce"`, cron livre directement via les adaptateurs de canal sortant.
L'agent principal n'est pas démarré pour rédiger ou transférer le message.

Détails de comportement :

- Contenu : la livraison utilise les charges utiles sortantes de l'exécution isolée (texte/media) avec le découpage normal et
  le formatage de canal.
- Les réponses uniquement heartbeat (`HEARTBEAT_OK` sans contenu réel) ne sont pas livrées.
- Si l'exécution isolée a déjà envoyé un message à la même cible via l'outil de message, la livraison est
  ignorée pour éviter les doublons.
- Les cibles de livraison manquantes ou invalides mettent la tâche en échec sauf si `delivery.bestEffort = true`.
- Un court résumé est publié dans la session principale uniquement quand `delivery.mode = "announce"`.
- Le résumé de session principale respecte `wakeMode` : `now` déclenche un heartbeat immédiat et
  `next-heartbeat` attend le prochain heartbeat planifié.

#### Flux de livraison webhook

Quand `delivery.mode = "webhook"`, cron envoie par POST la charge utile de l'événement terminé à `delivery.to` quand l'événement terminé inclut un résumé.

Détails de comportement :

- Le point de terminaison doit être une URL HTTP(S) valide.
- Aucune livraison de canal n'est tentée en mode webhook.
- Aucun résumé de session principale n'est publié en mode webhook.
- Si `cron.webhookToken` est défini, l'en-tête d'authentification est `Authorization: Bearer <cron.webhookToken>`.
- Repli déprécié : les anciennes tâches stockées avec `notify: true` envoient encore à `cron.webhook` (si configuré), avec un avertissement pour que vous puissiez migrer vers `delivery.mode = "webhook"`.

### Surcharges de modèle et de réflexion

Les tâches isolées (`agentTurn`) peuvent surcharger le modèle et le niveau de réflexion :

- `model` : chaîne fournisseur/modèle (par ex., `anthropic/claude-sonnet-4-20250514`) ou alias (par ex., `opus`)
- `thinking` : niveau de réflexion (`off`, `minimal`, `low`, `medium`, `high`, `xhigh` ; modèles GPT-5.2 + Codex uniquement)

Note : vous pouvez aussi définir `model` sur les tâches de session principale, mais cela change le modèle
partagé de la session principale. Nous recommandons les surcharges de modèle uniquement pour les tâches isolées afin d'éviter
des changements de contexte inattendus.

Priorité de résolution :

1. Surcharge de charge utile de tâche (la plus haute)
2. Valeurs par défaut spécifiques au hook (par ex., `hooks.gmail.model`)
3. Configuration par défaut de l'agent

### Livraison (canal + cible)

Les tâches isolées peuvent livrer la sortie à un canal via la configuration `delivery` de niveau supérieur :

- `delivery.mode` : `announce` (livraison par canal), `webhook` (POST HTTP), ou `none`.
- `delivery.channel` : `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`.
- `delivery.to` : cible de destinataire spécifique au canal.

La livraison `announce` n'est valide que pour les tâches isolées (`sessionTarget: "isolated"`).
La livraison `webhook` est valide pour les tâches principales et isolées.

Si `delivery.channel` ou `delivery.to` est omis, cron peut se rabattre sur la
"dernière route" de la session principale (le dernier endroit où l'agent a répondu).

Rappels de format de cible :

- Les cibles Slack/Discord/Mattermost (plugin) doivent utiliser des préfixes explicites (par ex. `channel:<id>`, `user:<id>`) pour éviter toute ambiguïté.
- Les sujets Telegram doivent utiliser la forme `:topic:` (voir ci-dessous).

#### Cibles de livraison Telegram (sujets / fils de forum)

Telegram supporte les sujets de forum via `message_thread_id`. Pour la livraison cron, vous pouvez encoder
le sujet/fil dans le champ `to` :

- `-1001234567890` (identifiant de chat uniquement)
- `-1001234567890:topic:123` (recommandé : marqueur de sujet explicite)
- `-1001234567890:123` (raccourci : suffixe numérique)

Les cibles préfixées comme `telegram:...` / `telegram:group:...` sont aussi acceptées :

- `telegram:group:-1001234567890:topic:123`

## Schéma JSON pour les appels d'outil

Utilisez ces formes lors de l'appel direct des outils `cron.*` du Gateway (appels d'outil d'agent ou RPC).
Les flags CLI acceptent des durées lisibles comme `20m`, mais les appels d'outil doivent utiliser une chaîne ISO 8601
pour `schedule.at` et des millisecondes pour `schedule.everyMs`.

### Paramètres cron.add

Tâche ponctuelle en session principale (événement système) :

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

Tâche isolée récurrente avec livraison :

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates."
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

Notes :

- `schedule.kind` : `at` (`at`), `every` (`everyMs`), ou `cron` (`expr`, `tz` optionnel).
- `schedule.at` accepte ISO 8601 (fuseau horaire optionnel ; traité comme UTC quand omis).
- `everyMs` est en millisecondes.
- `sessionTarget` doit être `"main"` ou `"isolated"` et doit correspondre à `payload.kind`.
- Champs optionnels : `agentId`, `description`, `enabled`, `deleteAfterRun` (par défaut true pour `at`),
  `delivery`.
- `wakeMode` par défaut à `"now"` quand omis.

### Paramètres cron.update

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

Notes :

- `jobId` est canonique ; `id` est accepté pour compatibilité.
- Utilisez `agentId: null` dans le patch pour effacer une liaison d'agent.

### Paramètres cron.run et cron.remove

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## Stockage et historique

- Magasin de tâches : `~/.openclaw/cron/jobs.json` (JSON géré par le Gateway).
- Historique des exécutions : `~/.openclaw/cron/runs/<jobId>.jsonl` (JSONL, élagué automatiquement par taille et nombre de lignes).
- Les sessions d'exécution cron isolées dans `sessions.json` sont élaguées par `cron.sessionRetention` (par défaut `24h` ; mettre `false` pour désactiver).
- Chemin de magasin personnalisé : `cron.store` dans la configuration.

## Configuration

```json5
{
  cron: {
    enabled: true, // par défaut true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // par défaut 1
    webhook: "https://example.invalid/legacy", // repli déprécié pour les tâches stockées avec notify:true
    webhookToken: "replace-with-dedicated-webhook-token", // jeton bearer optionnel pour le mode webhook
    sessionRetention: "24h", // chaîne de durée ou false
    runLog: {
      maxBytes: "2mb", // par défaut 2 000 000 octets
      keepLines: 2000, // par défaut 2000
    },
  },
}
```

Comportement d'élagage des journaux d'exécution :

- `cron.runLog.maxBytes` : taille maximale du fichier de journal d'exécution avant élagage.
- `cron.runLog.keepLines` : lors de l'élagage, ne conserver que les N lignes les plus récentes.
- Les deux s'appliquent aux fichiers `cron/runs/<jobId>.jsonl`.

Comportement webhook :

- Recommandé : définir `delivery.mode: "webhook"` avec `delivery.to: "https://..."` par tâche.
- Les URL webhook doivent être des URL `http://` ou `https://` valides.
- Lors de l'envoi, la charge utile est le JSON de l'événement terminé cron.
- Si `cron.webhookToken` est défini, l'en-tête d'authentification est `Authorization: Bearer <cron.webhookToken>`.
- Si `cron.webhookToken` n'est pas défini, aucun en-tête `Authorization` n'est envoyé.
- Repli déprécié : les anciennes tâches stockées avec `notify: true` utilisent encore `cron.webhook` quand présent.

Désactiver cron entièrement :

- `cron.enabled: false` (configuration)
- `OPENCLAW_SKIP_CRON=1` (environnement)

## Maintenance

Cron dispose de deux chemins de maintenance intégrés : rétention des sessions d'exécution isolées et élagage des journaux d'exécution.

### Valeurs par défaut

- `cron.sessionRetention` : `24h` (mettre `false` pour désactiver l'élagage des sessions d'exécution)
- `cron.runLog.maxBytes` : `2 000 000` octets
- `cron.runLog.keepLines` : `2000`

### Comment ça fonctionne

- Les exécutions isolées créent des entrées de session (`...:cron:<jobId>:run:<uuid>`) et des fichiers de transcription.
- Le récupérateur supprimé les entrées de session d'exécution expirées plus anciennes que `cron.sessionRetention`.
- Pour les sessions d'exécution supprimées qui ne sont plus référencées par le magasin de sessions, OpenClaw archive les fichiers de transcription et purge les anciennes archives supprimées sur la même fenêtre de rétention.
- Après chaque ajout d'exécution, `cron/runs/<jobId>.jsonl` est vérifié en taille :
  - si la taille du fichier dépasse `runLog.maxBytes`, il est réduit aux `runLog.keepLines` lignes les plus récentes.

### Avertissement de performance pour les planificateurs à haut volume

Les configurations cron à haute fréquence peuvent générer d'importants volumes de sessions d'exécution et de journaux. La maintenance est intégrée, mais des limités trop lâches peuvent encore créer du travail d'E/S et de nettoyage évitable.

Ce qu'il faut surveiller :

- de longues fenêtres `cron.sessionRetention` avec de nombreuses exécutions isolées
- un `cron.runLog.keepLines` élevé combiné à un grand `runLog.maxBytes`
- de nombreuses tâches récurrentes bruyantes écrivant dans le même `cron/runs/<jobId>.jsonl`

Ce qu'il faut faire :

- garder `cron.sessionRetention` aussi court que vos besoins de débogage/audit le permettent
- garder les journaux d'exécution bornés avec des `runLog.maxBytes` et `runLog.keepLines` modérés
- déplacer les tâches de fond bruyantes en mode isolé avec des règles de livraison qui évitent le bavardage inutile
- vérifier la croissance périodiquement avec `openclaw cron runs` et ajuster la rétention avant que les journaux ne deviennent volumineux

### Exemples de personnalisation

Conserver les sessions d'exécution pendant une semaine et permettre des journaux d'exécution plus grands :

```json5
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```

Désactiver l'élagage des sessions d'exécution isolées mais conserver l'élagage des journaux d'exécution :

```json5
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

Paramétrer pour une utilisation cron à haut volume (exemple) :

```json5
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```

## Démarrage rapide CLI

Rappel ponctuel (ISO UTC, suppression automatique après succès) :

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

Rappel ponctuel (session principale, réveil immédiat) :

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Tâche isolée récurrente (announce vers WhatsApp) :

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Tâche cron récurrente avec décalage explicite de 30 secondes :

```bash
openclaw cron add \
  --name "Minute watcher" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Run minute watcher checks." \
  --announce
```

Tâche isolée récurrente (livraison vers un sujet Telegram) :

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Tâche isolée avec surcharge de modèle et de réflexion :

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Sélection d'agent (configurations multi-agents) :

```bash
# Associer une tâche à l'agent "ops" (se rabat sur l'agent par défaut si cet agent est absent)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# Changer ou effacer l'agent sur une tâche existante
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

Exécution manuelle (force est le comportement par défaut, utilisez `--due` pour n'exécuter que si planifié) :

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

Modifier une tâche existante (patcher les champs) :

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Forcer une tâche cron existante à s'exécuter exactement selon le planning (sans décalage) :

```bash
openclaw cron edit <jobId> --exact
```

Historique des exécutions :

```bash
openclaw cron runs --id <jobId> --limit 50
```

Événement système immédiat sans créer de tâche :

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## Surface API du Gateway

- `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
- `cron.run` (force ou due), `cron.runs`
  Pour les événements système immédiats sans tâche, utilisez [`openclaw system event`](/cli/system).

## Dépannage

### "Rien ne s'exécute"

- Vérifier que cron est activé : `cron.enabled` et `OPENCLAW_SKIP_CRON`.
- Vérifier que le Gateway tourne en continu (cron s'exécute dans le processus Gateway).
- Pour les planifications `cron` : confirmer le fuseau horaire (`--tz`) par rapport au fuseau horaire de l'hôte.

### Une tâche récurrente continue de se retarder après des échecs

- OpenClaw applique un backoff exponentiel de réessai pour les tâches récurrentes après des erreurs consécutives :
  30s, 1m, 5m, 15m, puis 60m entre les réessais.
- Le backoff se réinitialise automatiquement après la prochaine exécution réussie.
- Les tâches ponctuelles (`at`) se désactivent après une exécution terminale (`ok`, `error`, ou `skipped`) et ne réessaient pas.

### Telegram livre au mauvais endroit

- Pour les sujets de forum, utilisez `-100...:topic:<id>` pour être explicite et sans ambiguïté.
- Si vous voyez des préfixes `telegram:...` dans les journaux ou les cibles "dernière route" stockées, c'est normal ;
  la livraison cron les accepte et analyse toujours correctement les identifiants de sujet.

### Réessais de livraison announce de sous-agent

- Quand une exécution de sous-agent se termine, le Gateway annonce le résultat à la session demandeuse.
- Si le flux d'annonce renvoie `false` (par ex. la session demandeuse est occupée), le Gateway réessaie jusqu'à 3 fois avec suivi via `announceRetryCount`.
- Les annonces plus anciennes que 5 minutes après `endedAt` sont expirées de force pour empêcher les entrées périmées de boucler indéfiniment.
- Si vous voyez des livraisons announce répétées dans les journaux, vérifiez le registre de sous-agents pour les entrées avec des valeurs `announceRetryCount` élevées.
