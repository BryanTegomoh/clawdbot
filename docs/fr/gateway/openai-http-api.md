---
summary: "Exposer un endpoint HTTP /v1/chat/completions compatible OpenAI depuis le Gateway"
read_when:
  - Intégration d'outils qui attendent OpenAI Chat Completions
title: "OpenAI Chat Completions"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/openai-http-api.md
  workflow: manual
---

# OpenAI Chat Completions (HTTP)

Le Gateway d'OpenClaw peut servir un petit endpoint Chat Completions compatible OpenAI.

Cet endpoint est **désactivé par défaut**. Activez-le d'abord dans la configuration.

- `POST /v1/chat/completions`
- Même port que le Gateway (multiplex WS + HTTP) : `http://<gateway-host>:<port>/v1/chat/completions`

En coulisses, les requêtes sont exécutées comme une exécution agent Gateway normale (même chemin de code que `openclaw agent`), donc le routage/permissions/configuration correspondent à votre Gateway.

## Authentification

Utilisé la configuration d'authentification du Gateway. Envoyez un jeton bearer :

- `Authorization: Bearer <token>`

Notes :

- Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).
- Si `gateway.auth.rateLimit` est configuré et que trop d'échecs d'authentification surviennent, l'endpoint renvoie `429` avec `Retry-After`.

## Choix d'un agent

Pas d'en-têtes personnalisés requis : encodez l'identifiant agent dans le champ OpenAI `model` :

- `model: "openclaw:<agentId>"` (exemple : `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

Ou ciblez un agent OpenClaw spécifique par en-tête :

- `x-openclaw-agent-id: <agentId>` (par défaut : `main`)

Avancé :

- `x-openclaw-session-key: <sessionKey>` pour contrôler totalement le routage de session.

## Activer l'endpoint

Définissez `gateway.http.endpoints.chatCompletions.enabled` sur `true` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## Désactiver l'endpoint

Définissez `gateway.http.endpoints.chatCompletions.enabled` sur `false` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## Comportement de session

Par défaut, l'endpoint est **sans état par requête** (une nouvelle clé de session est générée à chaque appel).

Si la requête inclut une chaîne OpenAI `user`, le Gateway dérive une clé de session stable à partir de celle-ci, de sorte que les appels répétés puissent partager une session agent.

## Streaming (SSE)

Définissez `stream: true` pour recevoir des Server-Sent Events (SSE) :

- `Content-Type: text/event-stream`
- Chaque ligne d'événement est `data: <json>`
- Le flux se termine par `data: [DONE]`

## Exemples

Non-streaming :

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Streaming :

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```
