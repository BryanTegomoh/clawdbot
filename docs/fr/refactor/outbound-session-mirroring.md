---
title: Refactorisation du mirroring de session sortante (Issue #1520)
description: Suivi des notes, décisions, tests et éléments ouverts de la refactorisation du mirroring de session sortante.
summary: "Notes de refactorisation pour le mirroring des envois sortants dans les sessions de canal cible"
read_when:
  - Travail sur le comportement de mirroring de transcription/session sortante
  - Debug de la dérivation de sessionKey pour les chemins d'outil send/message
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/refactor/outbound-session-mirroring.md
  workflow: manual
---

# Refactorisation du mirroring de session sortante (Issue #1520)

## Statut

- En cours.
- Le routage canal central + plugin mis à jour pour le mirroring sortant.
- L'envoi Gateway dérive maintenant la session cible quand sessionKey est omise.

## Contexte

Les envois sortants étaient mirrorés dans la session d'agent _courante_ (clé de session de l'outil) plutôt que dans la session de canal cible. Le routage entrant utilisé les clés de session canal/pair, donc les réponses sortantes atterrissaient dans la mauvaise session et les cibles de premier contact manquaient souvent d'entrées de session.

## Objectifs

- Mirrorer les messages sortants dans la clé de session du canal cible.
- Créer des entrées de session en sortant quand elles sont manquantes.
- Conserver le scoping thread/topic aligné avec les clés de session entrantes.
- Couvrir les canaux centraux plus les extensions bundlées.

## Résumé de l'implémentation

- Nouveau helper de routage de session sortante :
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` construit la sessionKey cible en utilisant `buildAgentSessionKey` (dmScope + identityLinks).
  - `ensureOutboundSessionEntry` écrit un `MsgContext` minimal via `recordSessionMetaFromInbound`.
- `runMessageAction` (send) dérive la sessionKey cible et la passe à `executeSendAction` pour le mirroring.
- `message-tool` ne mirrore plus directement ; il ne résout que l'agentId depuis la clé de session courante.
- Le chemin d'envoi du plugin mirrore via `appendAssistantMessageToSessionTranscript` en utilisant la sessionKey dérivée.
- L'envoi Gateway dérive une clé de session cible quand aucune n'est fournie (agent par défaut), et assuré une entrée de session.

## Gestion des threads/topics

- Slack : replyTo/threadId -> `resolveThreadSessionKeys` (suffixe).
- Discord : threadId/replyTo -> `resolveThreadSessionKeys` avec `useSuffix=false` pour correspondre à l'entrant (l'id de canal thread scope déjà la session).
- Telegram : les IDs de topic sont mappés vers `chatId:topic:<id>` via `buildTelegramGroupPeerId`.

## Extensions couvertes

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- Notes :
  - Les cibles Mattermost suppriment maintenant `@` pour le routage de clé de session DM.
  - Zalo Personal utilisé le type de pair DM pour les cibles 1:1 (groupe uniquement quand `group:` est présent).
  - Les cibles de groupe BlueBubbles suppriment les préfixes `chat_*` pour correspondre aux clés de session entrantes.
  - Le mirroring de thread auto Slack fait correspondre les ids de canal de manière insensible à la casse.
  - L'envoi Gateway met en minuscules les clés de session fournies avant le mirroring.

## Décisions

- **Dérivation de session d'envoi Gateway** : si `sessionKey` est fournie, l'utiliser. Si omise, dériver une sessionKey depuis la cible + l'agent par défaut et mirrorer là.
- **Création d'entrée de session** : toujours utiliser `recordSessionMetaFromInbound` avec `Provider/From/To/ChatType/AccountId/Originating*` alignés sur les formats entrants.
- **Normalisation de cible** : le routage sortant utilisé les cibles résolues (post `resolveChannelTarget`) quand disponibles.
- **Casse des clés de session** : canonicaliser les clés de session en minuscules à l'écriture et pendant les migrations.

## Tests ajoutés/mis à jour

- `src/infra/outbound/outbound.test.ts`
  - Clé de session thread Slack.
  - Clé de session topic Telegram.
  - identityLinks dmScope avec Discord.
- `src/agents/tools/message-tool.test.ts`
  - Dérive l'agentId depuis la clé de session (pas de sessionKey transmise).
- `src/gateway/server-methods/send.test.ts`
  - Dérive la clé de session quand omise et crée l'entrée de session.

## Éléments ouverts / suivis

- Le plugin voice-call utilisé des clés de session personnalisées `voice:<phone>`. Le mapping sortant n'est pas standardisé ici ; si message-tool devrait supporter les envois voice-call, ajouter un mapping explicite.
- Confirmer si un plugin externe utilise des formats `From/To` non standard au-delà de l'ensemble bundlé.

## Fichiers touchés

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- Tests dans :
  - `src/infra/outbound/outbound.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`
