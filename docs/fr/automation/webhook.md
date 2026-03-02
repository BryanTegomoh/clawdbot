---
summary: "Ingress webhook pour le réveil et les exécutions d'agent isolées"
read_when:
  - Ajout ou modification de points de terminaison webhook
  - Câblage de systèmes externes dans OpenClaw
title: "Webhooks"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/webhook.md
  workflow: manual
---

# Webhooks

Le Gateway peut exposer un petit point de terminaison HTTP webhook pour les déclencheurs externes.

## Activer

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // Optionnel : restreindre le routage explicite par `agentId` à cette liste d'autorisation.
    // Omettez ou incluez "*" pour autoriser n'importe quel agent.
    // Mettez [] pour refuser tout routage explicite par `agentId`.
    allowedAgentIds: ["hooks", "main"],
  },
}
```

Notes :

- `hooks.token` est requis quand `hooks.enabled=true`.
- `hooks.path` par défaut à `/hooks`.

## Authentification

Chaque requête doit inclure le jeton de hook. Préférez les en-têtes :

- `Authorization: Bearer <token>` (recommandé)
- `x-openclaw-token: <token>`
- Les jetons en query-string sont rejetés (`?token=...` renvoie `400`).

## Points de terminaison

### `POST /hooks/wake`

Charge utile :

```json
{ "text": "System line", "mode": "now" }
```

- `text` **requis** (string) : la description de l'événement (par ex., "New email received").
- `mode` optionnel (`now` | `next-heartbeat`) : déclencher un heartbeat immédiat (par défaut `now`) ou attendre la prochaine vérification périodique.

Effet :

- Met en file d'attente un événement système pour la session **principale**
- Si `mode=now`, déclenche un heartbeat immédiat

### `POST /hooks/agent`

Charge utile :

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **requis** (string) : le prompt ou message pour que l'agent le traite.
- `name` optionnel (string) : nom lisible du hook (par ex., "GitHub"), utilisé comme préfixe dans les résumés de session.
- `agentId` optionnel (string) : router ce hook vers un agent spécifique. Les IDs inconnus se rabattent sur l'agent par défaut. Quand défini, le hook s'exécute en utilisant l'espace de travail et la configuration de l'agent résout.
- `sessionKey` optionnel (string) : la clé utilisée pour identifier la session de l'agent. Par défaut ce champ est rejeté sauf si `hooks.allowRequestSessionKey=true`.
- `wakeMode` optionnel (`now` | `next-heartbeat`) : déclencher un heartbeat immédiat (par défaut `now`) ou attendre la prochaine vérification périodique.
- `deliver` optionnel (boolean) : si `true`, la réponse de l'agent sera envoyée au canal de messagerie. Par défaut `true`. Les réponses qui ne sont que des acquittements heartbeat sont automatiquement ignorées.
- `channel` optionnel (string) : le canal de messagerie pour la livraison. Un parmi : `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Par défaut `last`.
- `to` optionnel (string) : l'identifiant du destinataire pour le canal (par ex., numéro de téléphone pour WhatsApp/Signal, ID de chat pour Telegram, ID de canal pour Discord/Slack/Mattermost (plugin), ID de conversation pour MS Teams). Par défaut le dernier destinataire de la session principale.
- `model` optionnel (string) : surcharge de modèle (par ex., `anthropic/claude-3-5-sonnet` ou un alias). Doit être dans la liste de modèles autorisés si restreint.
- `thinking` optionnel (string) : surcharge du niveau de réflexion (par ex., `low`, `medium`, `high`).
- `timeoutSeconds` optionnel (number) : durée maximale pour l'exécution de l'agent en secondes.

Effet :

- Exécute un tour d'agent **isolé** (sa propre clé de session)
- Publie toujours un résumé dans la session **principale**
- Si `wakeMode=now`, déclenche un heartbeat immédiat

## Politique de clé de session (changement non rétrocompatible)

Les surcharges de `sessionKey` dans la charge utile de `/hooks/agent` sont désactivées par défaut.

- Recommandé : définir un `hooks.defaultSessionKey` fixe et garder les surcharges de requête désactivées.
- Optionnel : autoriser les surcharges de requête uniquement quand nécessaire, et restreindre les préfixes.

Configuration recommandée :

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Configuration de compatibilité (comportement hérité) :

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // fortement recommandé
  },
}
```

### `POST /hooks/<name>` (mappé)

Les noms de hook personnalisés sont résolus via `hooks.mappings` (voir la configuration). Un mapping peut
transformer des charges utiles arbitraires en actions `wake` ou `agent`, avec des templates ou
des transformations de code optionnelles.

Options de mapping (résumé) :

- `hooks.presets: ["gmail"]` activé le mapping Gmail intégré.
- `hooks.mappings` vous permet de définir `match`, `action`, et des templates dans la configuration.
- `hooks.transformsDir` + `transform.module` charge un module JS/TS pour une logique personnalisée.
  - `hooks.transformsDir` (si défini) doit rester dans la racine des transformations sous votre répertoire de configuration OpenClaw (typiquement `~/.openclaw/hooks/transforms`).
  - `transform.module` doit se résoudre dans le répertoire de transformations effectif (les chemins de traversée/échappement sont rejetés).
- Utilisez `match.source` pour garder un point d'ingestion générique (routage piloté par la charge utile).
- Les transformations TS nécessitent un chargeur TS (par ex. `bun` ou `tsx`) ou du `.js` pré-compilé au moment de l'exécution.
- Définissez `deliver: true` + `channel`/`to` sur les mappings pour router les réponses vers une surface de chat
  (`channel` par défaut à `last` et se rabat sur WhatsApp).
- `agentId` route le hook vers un agent spécifique ; les IDs inconnus se rabattent sur l'agent par défaut.
- `hooks.allowedAgentIds` restreint le routage explicite par `agentId`. Omettez-le (ou incluez `*`) pour autoriser n'importe quel agent. Mettez `[]` pour refuser le routage explicite par `agentId`.
- `hooks.defaultSessionKey` définit la session par défaut pour les exécutions d'agent de hook quand aucune clé explicite n'est fournie.
- `hooks.allowRequestSessionKey` contrôle si les charges utiles de `/hooks/agent` peuvent définir `sessionKey` (par défaut : `false`).
- `hooks.allowedSessionKeyPrefixes` restreint optionnellement les valeurs explicites de `sessionKey` depuis les charges utiles de requête et les mappings.
- `allowUnsafeExternalContent: true` désactive l'enveloppe de sécurité du contenu externe pour ce hook
  (dangereux ; uniquement pour les sources internes de confiance).
- `openclaw webhooks gmail setup` écrit la configuration `hooks.gmail` pour `openclaw webhooks gmail run`.
  Voir [Gmail Pub/Sub](/automation/gmail-pubsub) pour le flux complet du watch Gmail.

## Réponses

- `200` pour `/hooks/wake`
- `202` pour `/hooks/agent` (exécution asynchrone démarrée)
- `401` en cas d'échec d'authentification
- `429` après des échecs d'authentification répétés depuis le même client (vérifiez `Retry-After`)
- `400` en cas de charge utile invalide
- `413` en cas de charge utile surdimensionnée

## Exemples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Utiliser un modèle différent

Ajoutez `model` à la charge utile agent (ou au mapping) pour surcharger le modèle pour cette exécution :

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Si vous imposez `agents.defaults.models`, assurez-vous que le modèle de surcharge y est inclus.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Sécurité

- Gardez les points de terminaison de hook derrière le loopback, le tailnet, ou un reverse proxy de confiance.
- Utilisez un jeton de hook dédié ; ne réutilisez pas les jetons d'authentification du Gateway.
- Les échecs d'authentification répétés sont soumis à une limitation de débit par adresse client pour ralentir les tentatives de force brute.
- Si vous utilisez le routage multi-agents, définissez `hooks.allowedAgentIds` pour limiter la sélection explicite par `agentId`.
- Gardez `hooks.allowRequestSessionKey=false` sauf si vous avez besoin de sessions sélectionnées par l'appelant.
- Si vous activez les `sessionKey` de requête, restreignez `hooks.allowedSessionKeyPrefixes` (par exemple, `["hook:"]`).
- Évitez d'inclure des charges utiles brutes sensibles dans les logs webhook.
- Les charges utiles de hook sont traitées comme non fiables et encadrées par des limités de sécurité par défaut.
  Si vous devez désactiver cela pour un hook spécifique, définissez `allowUnsafeExternalContent: true`
  dans le mapping de ce hook (dangereux).
