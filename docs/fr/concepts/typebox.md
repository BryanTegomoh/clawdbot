---
summary: "Les schemas TypeBox comme référence unique pour le protocole du Gateway"
read_when:
  - Mise a jour des schemas de protocole ou de la génération de code
title: "TypeBox"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/typebox.md
  workflow: manual
---

# TypeBox comme référence unique du protocole

Dernière mise à jour : 2026-01-10

TypeBox est une bibliothèque de schemas TypeScript-first. Nous l'utilisons pour définir le **protocole WebSocket du Gateway** (handshake, requête/réponse, événements serveur). Ces schemas pilotent la **validation à l'exécution**, l'**export JSON Schema** et la **génération de code Swift** pour l'application macOS. Une seule source de référence ; tout le reste est généré.

Si vous souhaitez le contexte de protocole de plus haut niveau, commencez par [Architecture du Gateway](/concepts/architecture).

## Modèle mental (30 secondes)

Chaque message WS du Gateway est l'une de trois trames :

- **Request** : `{ type: "req", id, method, params }`
- **Response** : `{ type: "res", id, ok, payload | error }`
- **Event** : `{ type: "event", event, payload, seq?, stateVersion? }`

La première trame **doit** être une requête `connect`. Après cela, les clients peuvent appeler des méthodes (par ex. `health`, `send`, `chat.send`) et s'abonner a des événements (par ex. `presence`, `tick`, `agent`).

Flux de connexion (minimal) :

```
Client                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Méthodes + événements courants :

| Catégorie | Exemples                                                  | Notes                              |
| --------- | --------------------------------------------------------- | ---------------------------------- |
| Noyau     | `connect`, `health`, `status`                             | `connect` doit être en premier     |
| Messagerie| `send`, `poll`, `agent`, `agent.wait`                     | les effets de bord nécessitent `idempotencyKey` |
| Chat      | `chat.history`, `chat.send`, `chat.abort`, `chat.inject`  | WebChat les utilisé                |
| Sessions  | `sessions.list`, `sessions.patch`, `sessions.delete`      | administration des sessions        |
| Nœuds    | `node.list`, `node.invoke`, `node.pair.*`                 | Gateway WS + actions de nœuds     |
| Événements| `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | push serveur                       |

La liste faisant autorité se trouve dans `src/gateway/server.ts` (`METHODS`, `EVENTS`).

## Ou se trouvent les schemas

- Source : `src/gateway/protocol/schema.ts`
- Validateurs à l'exécution (AJV) : `src/gateway/protocol/index.ts`
- Handshake serveur + dispatch des méthodes : `src/gateway/server.ts`
- Client nœud : `src/gateway/client.ts`
- JSON Schema généré : `dist/protocol.schema.json`
- Modèles Swift générés : `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## Pipeline actuel

- `pnpm protocol:gen`
  - écrit le JSON Schema (draft-07) dans `dist/protocol.schema.json`
- `pnpm protocol:gen:swift`
  - généré les modèles Swift du Gateway
- `pnpm protocol:check`
  - exécute les deux générateurs et vérifie que la sortie est commitée

## Comment les schemas sont utilisés à l'exécution

- **Côté serveur** : chaque trame entrante est validée avec AJV. Le handshake n'accepte qu'une requête `connect` dont les paramètres correspondent a `ConnectParams`.
- **Côté client** : le client JS valide les trames d'événements et de réponses avant de les utiliser.
- **Surface des méthodes** : le Gateway annonce les `methods` et `events` supportes dans `hello-ok`.

## Exemples de trames

Connexion (premier message) :

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Réponse hello-ok :

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Requête + réponse :

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Événement :

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## Client minimal (Node.js)

Flux utile le plus petit : connect + health.

```ts
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

## Exemple détaillé : ajouter une méthode de bout en bout

Exemple : ajouter une nouvelle requête `system.echo` qui retourne `{ ok: true, text }`.

1. **Schema (source de référence)**

Ajouter dans `src/gateway/protocol/schema.ts` :

```ts
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Ajouter les deux a `ProtocolSchemas` et exporter les types :

```ts
  SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```ts
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2. **Validation**

Dans `src/gateway/protocol/index.ts`, exporter un validateur AJV :

```ts
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3. **Comportement serveur**

Ajouter un handler dans `src/gateway/server-methods/system.ts` :

```ts
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

L'enregistrer dans `src/gateway/server-methods.ts` (qui fusionne déjà `systemHandlers`), puis ajouter `"system.echo"` a `METHODS` dans `src/gateway/server.ts`.

4. **Regénérér**

```bash
pnpm protocol:check
```

5. **Tests + docs**

Ajouter un test serveur dans `src/gateway/server.*.test.ts` et noter la méthode dans la documentation.

## Comportement de la génération de code Swift

Le générateur Swift émet :

- Un enum `GatewayFrame` avec les cas `req`, `res`, `event` et `unknown`
- Des structs/enums de payload fortement types
- Les valeurs `ErrorCode` et `GATEWAY_PROTOCOL_VERSION`

Les types de trames inconnus sont préservés en tant que payloads bruts pour la compatibilité ascendante.

## Versionnement + compatibilité

- `PROTOCOL_VERSION` se trouve dans `src/gateway/protocol/schema.ts`.
- Les clients envoient `minProtocol` + `maxProtocol` ; le serveur rejette les incompatibilités.
- Les modèles Swift conservent les types de trames inconnus pour eviter de casser les anciens clients.

## Motifs et conventions des schemas

- La plupart des objets utilisent `additionalProperties: false` pour des payloads stricts.
- `NonEmptyString` est la valeur par défaut pour les identifiants et les noms de méthodes/événements.
- Le `GatewayFrame` de niveau supérieur utilisé un **discriminateur** sur `type`.
- Les méthodes avec effets de bord nécessitent généralement un `idempotencyKey` dans les paramètres (exemple : `send`, `poll`, `agent`, `chat.send`).

## JSON Schema en direct

Le JSON Schema généré est dans le depot a `dist/protocol.schema.json`. Le fichier brut publie est généralement disponible a :

- [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## Quand vous modifiez les schemas

1. Mettez à jour les schemas TypeBox.
2. Exécutez `pnpm protocol:check`.
3. Commitez le schema regénéré + les modèles Swift.
