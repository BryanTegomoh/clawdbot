---
summary: "Flux de messages, sessions, mise en file d'attente et visibilité du raisonnement"
read_when:
  - Explication de comment les messages entrants deviennent des réponses
  - Clarification des sessions, modes de file d'attente ou comportement de streaming
  - Documentation de la visibilité du raisonnement et implications d'utilisation
title: "Messages"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/messages.md
  workflow: manual
---

# Messages

Cette page lié ensemble la manière dont OpenClaw gère les messages entrants, les sessions, la mise en file d'attente, le streaming et la visibilité du raisonnement.

## Flux de messages (vue d'ensemble)

```
Message entrant
  -> routage/bindings -> clé de session
  -> file d'attente (si une exécution est active)
  -> exécution de l'agent (streaming + outils)
  -> réponses sortantes (limites du canal + découpage)
```

Les principaux paramètres résident dans la configuration :

- `messages.*` pour les préfixes, la mise en file d'attente et le comportement de groupe.
- `agents.defaults.*` pour le streaming par blocs et les paramètres de découpage par défaut.
- Surcharges par canal (`channels.whatsapp.*`, `channels.telegram.*`, etc.) pour les plafonds et les bascules de streaming.

Voir [Configuration](/gateway/configuration) pour le schéma complet.

## Déduplication entrante

Les canaux peuvent redistribuer le même message après des reconnexions. OpenClaw maintient un cache de courte durée indexé par canal/compte/pair/session/identifiant de message afin que les doublons ne déclenchent pas une autre exécution de l'agent.

## Anti-rebond entrant

Des messages consécutifs rapides du **même expéditeur** peuvent être regroupés en un seul tour d'agent via `messages.inbound`. L'anti-rebond est limité par canal + conversation et utilise le message le plus récent pour le threading/les identifiants de réponse.

Configuration (défaut global + surcharges par canal) :

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

Notes :

- L'anti-rebond s'applique aux messages **texte uniquement** ; les médias/pièces jointes déclenchent immédiatement.
- Les commandes de contrôle contournent l'anti-rebond pour qu'elles restent autonomes.

## Sessions et appareils

Les sessions sont détenues par le Gateway, pas par les clients.

- Les conversations directes se replient dans la clé de session principale de l'agent.
- Les groupes/canaux obtiennent leurs propres clés de session.
- Le magasin de sessions et les transcriptions résident sur l'hôte du Gateway.

Plusieurs appareils/canaux peuvent correspondre à la même session, mais l'historique n'est pas entièrement synchronisé vers chaque client. Recommandation : utiliser un appareil principal pour les longues conversations afin d'éviter un contexte divergent. L'interface de contrôle et le TUI affichent toujours la transcription de session du Gateway, donc ils sont la source de vérité.

Détails : [Gestion des sessions](/concepts/session).

## Corps entrants et contexte d'historique

OpenClaw sépare le **corps du prompt** du **corps de commande** :

- `Body` : texte du prompt envoyé à l'agent. Peut inclure des enveloppes de canal et des wrappers d'historique optionnels.
- `CommandBody` : texte brut de l'utilisateur pour l'analyse des directives/commandes.
- `RawBody` : alias hérité de `CommandBody` (conservé pour la compatibilité).

Quand un canal fournit l'historique, il utilise un wrapper partagé :

- `[Chat messages since your last reply - for context]`
- `[Current message - respond to this]`

Pour les **conversations non directes** (groupes/canaux/salons), le **corps du message actuel** est préfixé avec le label de l'expéditeur (même style utilisé pour les entrées d'historique). Cela maintient la cohérence des messages en temps réel et en file d'attente/historique dans le prompt de l'agent.

Les tampons d'historique sont **en attente uniquement** : ils incluent les messages de groupe qui n'ont _pas_ déclenché d'exécution (par exemple, les messages soumis à la mention) et **excluent** les messages déjà dans la transcription de session.

La suppression des directives s'applique uniquement à la section du **message actuel** pour que l'historique reste intact. Les canaux qui encapsulent l'historique doivent définir `CommandBody` (ou `RawBody`) au texte du message original et garder `Body` comme le prompt combiné. Les tampons d'historique sont configurables via `messages.groupChat.historyLimit` (défaut global) et les surcharges par canal comme `channels.slack.historyLimit` ou `channels.telegram.accounts.<id>.historyLimit` (mettre `0` pour désactiver).

## Mise en file d'attente et suivis

Si une exécution est déjà activé, les messages entrants peuvent être mis en file d'attente, orientés vers l'exécution en cours, ou collectés pour un tour de suivi.

- Configurer via `messages.queue` (et `messages.queue.byChannel`).
- Modes : `interrupt`, `steer`, `followup`, `collect`, plus les variantes backlog.

Détails : [File d'attente](/concepts/queue).

## Streaming, découpage et regroupement

Le streaming par blocs envoie des réponses partielles au fur et à mesure que le modèle produit des blocs de texte. Le découpage respecte les limités de texte du canal et évite de couper le code délimité.

Paramètres clés :

- `agents.defaults.blockStreamingDefault` (`on|off`, désactivé par défaut)
- `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
- `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
- `agents.defaults.blockStreamingCoalesce` (regroupement basé sur l'inactivité)
- `agents.defaults.humanDelay` (pause humaine entre les réponses par blocs)
- Surcharges par canal : `*.blockStreaming` et `*.blockStreamingCoalesce` (les canaux non-Telegram nécessitent un `*.blockStreaming: true` explicite)

Détails : [Streaming + découpage](/concepts/streaming).

## Visibilité du raisonnement et tokens

OpenClaw peut exposer ou masquer le raisonnement du modèle :

- `/reasoning on|off|stream` contrôle la visibilité.
- Le contenu de raisonnement compte toujours dans l'utilisation de tokens quand il est produit par le modèle.
- Telegram supporte le stream de raisonnement dans la bulle de brouillon.

Détails : [Thinking + directives de raisonnement](/tools/thinking) et [Utilisation de tokens](/reference/token-use).

## Préfixes, threading et réponses

Le formatage des messages sortants est centralisé dans `messages` :

- `messages.responsePrefix`, `channels.<channel>.responsePrefix`, et `channels.<channel>.accounts.<id>.responsePrefix` (cascade de préfixes sortants), plus `channels.whatsapp.messagePrefix` (préfixe entrant WhatsApp)
- Threading de réponse via `replyToMode` et paramètres par défaut par canal

Détails : [Configuration](/gateway/configuration#messages) et documentation des canaux.
