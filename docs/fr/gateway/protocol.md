---
summary: "Protocole WebSocket du Gateway : handshake, trames, versionnage"
read_when:
  - Implémentation ou mise à jour des clients WS du gateway
  - Débogage des incohérences de protocole ou des échecs de connexion
  - Régénération du schéma/modèles de protocole
title: "Protocole Gateway"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/protocol.md
  workflow: manual
---

# Protocole Gateway (WebSocket)

Le protocole WS du Gateway est le **plan de contrôle unique + transport de nœuds** pour OpenClaw. Tous les clients (CLI, interface web, application macOS, nœuds iOS/Android, nœuds headless) se connectent via WebSocket et déclarent leur **rôle** + **scope** au moment du handshake.

## Transport

- WebSocket, trames texte avec payloads JSON.
- La première trame **doit** être une requête `connect`.

## Handshake (connect)

Gateway vers Client (challenge pré-connexion) :

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Client vers Gateway :

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Gateway vers Client :

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Lorsqu'un jeton de périphérique est émis, `hello-ok` inclut aussi :

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### Exemple nœud

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## Tramage

- **Requête** : `{type:"req", id, method, params}`
- **Réponse** : `{type:"res", id, ok, payload|error}`
- **Événement** : `{type:"event", event, payload, seq?, stateVersion?}`

Les méthodes avec effets de bord nécessitent des **clés d'idempotence** (voir le schéma).

## Rôles + scopes

### Rôles

- `operator` = client du plan de contrôle (CLI/interface/automation).
- `node` = hôte de capacités (camera/screen/canvas/system.run).

### Scopes (operator)

Scopes courants :

- `operator.read`
- `operator.write`
- `operator.admin`
- `operator.approvals`
- `operator.pairing`

### Caps/commands/permissions (node)

Les nœuds déclarent leurs revendications de capacités au moment de la connexion :

- `caps` : catégories de capacités de haut niveau.
- `commands` : liste autorisée de commandes pour l'invocation.
- `permissions` : bascules granulaires (par ex. `screen.record`, `camera.capture`).

Le Gateway traite celles-ci comme des **revendications** et applique des listes autorisées côté serveur.

## Présence

- `system-presence` retourne des entrées indexées par identité de périphérique.
- Les entrées de présence incluent `deviceId`, `rôles` et `scopes` pour que les interfaces puissent afficher une seule ligne par périphérique même lorsqu'il se connecte à la fois comme **operator** et comme **node**.

### Méthodes d'aide nœud

- Les nœuds peuvent appeler `skills.bins` pour récupérer la liste actuelle des exécutables de skills pour les vérifications d'auto-autorisation.

### Méthodes d'aide operator

- Les operators peuvent appeler `tools.catalog` (`operator.read`) pour récupérer le catalogue d'outils runtime d'un agent. La réponse inclut les outils groupés et les métadonnées de provenance :
  - `source` : `core` ou `plugin`
  - `pluginId` : propriétaire du plugin quand `source="plugin"`
  - `optional` : si un outil de plugin est optionnel

## Approbations exec

- Lorsqu'une requête exec nécessite une approbation, le gateway diffuse `exec.approval.requested`.
- Les clients operator résolvent en appelant `exec.approval.resolve` (nécessite le scope `operator.approvals`).

## Versionnage

- `PROTOCOL_VERSION` se trouve dans `src/gateway/protocol/schema.ts`.
- Les clients envoient `minProtocol` + `maxProtocol` ; le serveur rejette les incohérences.
- Les schémas + modèles sont générés à partir des définitions TypeBox :
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`
  - `pnpm protocol:check`

## Auth

- Si `OPENCLAW_GATEWAY_TOKEN` (ou `--token`) est défini, `connect.params.auth.token` doit correspondre ou la socket est fermée.
- Après l'appairage, le Gateway émet un **jeton de périphérique** limité au rôle + scopes de la connexion. Il est retourné dans `hello-ok.auth.deviceToken` et doit être persisté par le client pour les connexions futures.
- Les jetons de périphérique peuvent être renouvelés/révoqués via `device.token.rotate` et `device.token.revoke` (nécessite le scope `operator.pairing`).

## Identité de périphérique + appairage

- Les nœuds doivent inclure une identité de périphérique stable (`device.id`) dérivée d'une empreinte de clé.
- Les Gateways émettent des jetons par périphérique + rôle.
- Les approbations d'appairage sont requises pour les nouveaux IDs de périphérique sauf si l'auto-approbation locale est activée.
- Les connexions **locales** incluent le loopback et l'adresse tailnet propre de l'hôte gateway (pour que les binds tailnet sur le même hôte puissent toujours être auto-approuvés).
- Tous les clients WS doivent inclure l'identité `device` lors du `connect` (operator + node). L'Interface de contrôle peut l'omettre **uniquement** lorsque `gateway.controlUi.dangerouslyDisableDeviceAuth` est activé pour un accès de secours.
- Toutes les connexions doivent signer le nonce `connect.challenge` fourni par le serveur.
- Le payload de signature préféré est `v3`, qui lie `platform` et `deviceFamily` en plus des champs device/client/rôle/scopes/token/nonce.
- Les signatures `v2` anciennes restent acceptées pour la compatibilité, mais l'épinglage des métadonnées de périphérique appaire contrôle toujours la politique de commande lors de la reconnexion.

## TLS + pinning

- TLS est supporté pour les connexions WS.
- Les clients peuvent optionnellement épingler l'empreinte du certificat gateway (voir la configuration `gateway.tls` plus `gateway.remote.tlsFingerprint` ou le CLI `--tls-fingerprint`).

## Portée

Ce protocole expose l'**API complète du gateway** (statut, canaux, modèles, chat, agent, sessions, nœuds, approbations, etc.). La surface exacte est définie par les schémas TypeBox dans `src/gateway/protocol/schema.ts`.
