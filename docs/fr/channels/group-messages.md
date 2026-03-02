---
summary: "Comportement et configuration pour la gestion des messages de groupe WhatsApp (les mentionPatterns sont partagees entre les surfaces)"
read_when:
  - Modification des règles de messages de groupe ou des mentions
title: "Messages de groupe"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/group-messages.md"
  workflow: "manual"
---

# Messages de groupe (canal web WhatsApp)

Objectif : permettre a Clawd de rester dans les groupes WhatsApp, se reveiller uniquement quand il est mentionne, et garder ce fil séparé de la session DM personnelle.

Note : `agents.list[].groupChat.mentionPatterns` est maintenant utilisé aussi par Telegram/Discord/Slack/iMessage ; cette documentation se concentre sur le comportement spécifique a WhatsApp. Pour les configurations multi-agents, définissez `agents.list[].groupChat.mentionPatterns` par agent (ou utilisez `messages.groupChat.mentionPatterns` comme repli global).

## Ce qui est implemente (2025-12-03)

- Modes d'activation : `mention` (défaut) ou `always`. `mention` nécessite un ping (vraies @-mentions WhatsApp via `mentionedJids`, patterns regex, ou le E.164 du bot n'importe ou dans le texte). `always` reveille l'agent a chaque message mais il ne devrait répondre que quand il peut apporter une valeur significative ; sinon il retourne le jeton silencieux `NO_REPLY`. Les valeurs par défaut peuvent être définies dans la configuration (`channels.whatsapp.groups`) et surchargees par groupe via `/activation`. Quand `channels.whatsapp.groups` est défini, il agit aussi comme liste d'autorisation de groupes (incluez `"*"` pour autoriser tous).
- Politique de groupe : `channels.whatsapp.groupPolicy` contrôle si les messages de groupe sont acceptés (`open|disabled|allowlist`). `allowlist` utilisé `channels.whatsapp.groupAllowFrom` (repli : `channels.whatsapp.allowFrom` explicite). Le défaut est `allowlist` (bloque tant que vous n'ajoutez pas d'expéditeurs).
- Sessions par groupe : les clés de session ressemblent a `agent:<agentId>:whatsapp:group:<jid>` donc les commandes comme `/verbose on` ou `/think high` (envoyées comme messages indépendants) sont limitées a ce groupe ; l'état DM personnel est intact. Les heartbeats sont ignorés pour les fils de groupe.
- Injection de contexte : les messages de groupe **en attente uniquement** (défaut 50) qui n'ont _pas_ déclenché une exécution sont préfixes sous `[Chat messages since your last reply - for context]`, avec la ligne declenchante sous `[Current message - respond to this]`. Les messages déjà dans la session ne sont pas reinjectes.
- Affichage de l'expéditeur : chaque lot de groupe se termine maintenant par `[from: Sender Name (+E164)]` pour que Pi sache qui parle.
- Ephemere/a usage unique : nous deballons ceux-ci avant d'extraire le texte/mentions, donc les pings a l'interieur declenchent quand même.
- Prompt système de groupe : au premier tour d'une session de groupe (et chaque fois que `/activation` change le mode) nous injectons un court texte dans le prompt système comme `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), ... Activation: trigger-only ... Address the specific sender noted in the message context.` Si les métadonnées ne sont pas disponibles, nous informons quand même l'agent qu'il est dans un chat de groupe.

## Exemple de configuration (WhatsApp)

Ajoutez un bloc `groupChat` a `~/.openclaw/openclaw.json` pour que les pings par nom d'affichage fonctionnent même quand WhatsApp supprimé le `@` visuel dans le corps du texte :

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\+?15555550123"],
        },
      },
    ],
  },
}
```

Notes :

- Les regex sont insensibles a la casse ; elles couvrent un ping par nom d'affichage comme `@openclaw` et le numéro brut avec ou sans `+`/espaces.
- WhatsApp envoie toujours les mentions canoniques via `mentionedJids` quand quelqu'un appuie sur le contact, donc le repli par numéro est rarement nécessaire mais c'est un filet de sécurité utile.

### Commande d'activation (propriétaire uniquement)

Utilisez la commande de chat de groupe :

- `/activation mention`
- `/activation always`

Seul le numéro du propriétaire (depuis `channels.whatsapp.allowFrom`, ou le propre E.164 du bot quand non défini) peut changer cela. Envoyez `/status` comme message indépendant dans le groupe pour voir le mode d'activation actuel.

## Comment utiliser

1. Ajoutez votre compte WhatsApp (celui qui exécute OpenClaw) au groupe.
2. Dites `@openclaw ...` (ou incluez le numéro). Seuls les expéditeurs dans la liste d'autorisation peuvent déclencher sauf si vous définissez `groupPolicy: "open"`.
3. Le prompt de l'agent inclura le contexte recent du groupe plus le marqueur `[from: ...]` final pour qu'il puisse s'adresser a la bonne personne.
4. Les directives au niveau session (`/verbose on`, `/think high`, `/new` ou `/reset`, `/compact`) s'appliquent uniquement a la session de ce groupe ; envoyez-les comme messages indépendants pour qu'elles soient enregistrees. Votre session DM personnelle reste indépendante.

## Test / vérification

- Test manuel :
  - Envoyez un ping `@openclaw` dans le groupe et confirmez une réponse qui référence le nom de l'expéditeur.
  - Envoyez un second ping et verifiez que le bloc d'historique est inclus puis efface au tour suivant.
- Verifiez les logs du Gateway (lancez avec `--verbose`) pour voir les entrées `inbound web message` montrant `from: <groupJid>` et le suffixe `[from: ...]`.

## Considerations connues

- Les heartbeats sont intentionnellement ignorés pour les groupes afin d'eviter les diffusions bruyantes.
- La suppression d'echo utilisé la chaine de lot combinee ; si vous envoyez un texte identique deux fois sans mentions, seul le premier obtiendra une réponse.
- Les entrées du magasin de sessions apparaitront comme `agent:<agentId>:whatsapp:group:<jid>` dans le magasin de sessions (`~/.openclaw/agents/<agentId>/sessions/sessions.json` par défaut) ; une entrée manquante signifie simplement que le groupe n'a pas encore déclenché d'exécution.
- Les indicateurs de frappe dans les groupes suivent `agents.defaults.typingMode` (défaut : `message` quand non mentionne).
