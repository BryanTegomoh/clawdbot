---
summary: "Conception de la file d'attente de commandes qui sérialise les exécutions d'auto-réponse entrantes"
read_when:
  - Modification de l'exécution d'auto-réponse ou de la concurrence
title: "File d'attente de commandes"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/queue.md
  workflow: manual
---

# File d'attente de commandes (2026-01-16)

Nous sérialisons les exécutions d'auto-réponse entrantes (tous les canaux) à travers une petite file d'attente en processus pour empêcher plusieurs exécutions d'agent de se percuter, tout en permettant un parallélisme sûr entre les sessions.

## Pourquoi

- Les exécutions d'auto-réponse peuvent être coûteuses (appels LLM) et peuvent se percuter quand plusieurs messages entrants arrivent à des moments rapprochés.
- La sérialisation évite la compétition pour les ressources partagées (fichiers de session, journaux, stdin du CLI) et réduit les chances de limités de débit en amont.

## Comment ça fonctionne

- Une file FIFO orientée voies draine chaque voie avec un plafond de concurrence configurable (défaut 1 pour les voies non configurées ; main par défaut 4, subagent 8).
- `runEmbeddedPiAgent` met en file d'attente par **clé de session** (voie `session:<key>`) pour garantir une seule exécution active par session.
- Chaque exécution de session est ensuite mise en file dans une **voie globale** (`main` par défaut) pour que le parallélisme global soit plafonné par `agents.defaults.maxConcurrent`.
- Quand la journalisation verbose est activée, les exécutions en file d'attente émettent un court avis si elles ont attendu plus de ~2s avant de démarrer.
- Les indicateurs de frappe se déclenchent toujours immédiatement à la mise en file (quand supporté par le canal) pour que l'expérience utilisateur soit inchangée pendant l'attente.

## Modes de file d'attente (par canal)

Les messages entrants peuvent orienter l'exécution en cours, attendre un tour de suivi, ou faire les deux :

- `steer` : injecter immédiatement dans l'exécution en cours (annule les appels d'outils en attente après la prochaine frontière d'outil). Si pas en streaming, replie vers followup.
- `followup` : mettre en file pour le prochain tour d'agent après la fin de l'exécution en cours.
- `collect` : fusionner tous les messages en file en un **seul** tour de suivi (par défaut). Si les messages ciblent différents canaux/threads, ils se drainent individuellement pour préserver le routage.
- `steer-backlog` (alias `steer+backlog`) : orienter maintenant **et** préserver le message pour un tour de suivi.
- `interrupt` (ancien) : abandonner l'exécution active pour cette session, puis exécuter le message le plus récent.
- `queue` (alias ancien) : identique à `steer`.

Steer-backlog signifie que vous pouvez obtenir une réponse de suivi après l'exécution orientée, donc les surfaces de streaming peuvent ressembler à des doublons. Préférez `collect`/`steer` si vous voulez une seule réponse par message entrant.
Envoyez `/queue collect` comme commande autonome (par session) ou définissez `messages.queue.byChannel.discord: "collect"`.

Valeurs par défaut (quand non défini dans la configuration) :

- Toutes les surfaces -> `collect`

Configurez globalement ou par canal via `messages.queue` :

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```

## Options de file d'attente

Les options s'appliquent à `followup`, `collect` et `steer-backlog` (et à `steer` quand il replie vers followup) :

- `debounceMs` : attendre le silence avant de démarrer un tour de suivi (empêche « continue, continue »).
- `cap` : messages max en file par session.
- `drop` : politique de débordement (`old`, `new`, `summarize`).

Summarize garde une courte liste à puces des messages supprimés et l'injecte comme prompt de suivi synthétique.
Valeurs par défaut : `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

## Surcharges par session

- Envoyez `/queue <mode>` comme commande autonome pour stocker le mode pour la session en cours.
- Les options peuvent être combinées : `/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` ou `/queue reset` efface la surcharge de session.

## Portée et garanties

- S'applique aux exécutions d'agent d'auto-réponse à travers tous les canaux entrants qui utilisent le pipeline de réponse du Gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, etc.).
- La voie par défaut (`main`) est à l'échelle du processus pour les entrants + heartbeats principaux ; définir `agents.defaults.maxConcurrent` pour permettre plusieurs sessions en parallèle.
- Des voies supplémentaires peuvent exister (par ex. `cron`, `subagent`) pour que les jobs en arrière-plan puissent s'exécuter en parallèle sans bloquer les réponses entrantes.
- Les voies par session garantissent qu'une seule exécution d'agent touche une session donnée à la fois.
- Pas de dépendances externes ni de threads de travail en arrière-plan ; TypeScript pur + promises.

## Dépannage

- Si les commandes semblent bloquées, activez les journaux verbose et cherchez les lignes « queued for ...ms » pour confirmer que la file se draine.
- Si vous avez besoin de la profondeur de la file, activez les journaux verbose et surveillez les lignes de timing de la file.
