---
summary: "Exposer un endpoint HTTP /v1/responses compatible OpenResponses depuis le Gateway"
read_when:
  - Intégration de clients qui parlent l'API OpenResponses
  - Vous souhaitez des entrées basées sur des items, des appels d'outils client ou des événements SSE
title: "API OpenResponses"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/openresponses-http-api.md
  workflow: manual
---

# API OpenResponses (HTTP)

Le Gateway d'OpenClaw peut servir un endpoint `POST /v1/responses` compatible OpenResponses.

Cet endpoint est **désactivé par défaut**. Activez-le d'abord dans la configuration.

- `POST /v1/responses`
- Même port que le Gateway (multiplex WS + HTTP) : `http://<gateway-host>:<port>/v1/responses`

En coulisses, les requêtes sont exécutées comme une exécution agent Gateway normale (même chemin de code que `openclaw agent`), donc le routage/permissions/configuration correspondent à votre Gateway.

## Authentification

Utilisé la configuration d'authentification du Gateway. Envoyez un jeton bearer :

- `Authorization: Bearer <token>`

Notes :

- Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).
- Si `gateway.auth.rateLimit` est configuré et que trop d'échecs d'authentification surviennent, l'endpoint renvoie `429` avec `Retry-After`.

## Choix d'un agent

Pas d'en-têtes personnalisés requis : encodez l'identifiant agent dans le champ OpenResponses `model` :

- `model: "openclaw:<agentId>"` (exemple : `"openclaw:main"`, `"openclaw:beta"`)
- `model: "agent:<agentId>"` (alias)

Ou ciblez un agent OpenClaw spécifique par en-tête :

- `x-openclaw-agent-id: <agentId>` (par défaut : `main`)

Avancé :

- `x-openclaw-session-key: <sessionKey>` pour contrôler totalement le routage de session.

## Activer l'endpoint

Définissez `gateway.http.endpoints.responses.enabled` sur `true` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## Désactiver l'endpoint

Définissez `gateway.http.endpoints.responses.enabled` sur `false` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## Comportement de session

Par défaut, l'endpoint est **sans état par requête** (une nouvelle clé de session est générée à chaque appel).

Si la requête inclut une chaîne OpenResponses `user`, le Gateway dérive une clé de session stable à partir de celle-ci, de sorte que les appels répétés puissent partager une session agent.

## Forme de requête (supportée)

La requête suit l'API OpenResponses avec une entrée basée sur des items. Support actuel :

- `input` : chaîne ou tableau d'objets item.
- `instructions` : fusionné dans le prompt système.
- `tools` : définitions d'outils client (outils fonction).
- `tool_choice` : filtrer ou exiger des outils client.
- `stream` : activé le streaming SSE.
- `max_output_tokens` : limité de sortie au mieux (dépend du fournisseur).
- `user` : routage de session stable.

Accepté mais **actuellement ignoré** :

- `max_tool_calls`
- `reasoning`
- `metadata`
- `store`
- `previous_response_id`
- `truncation`

## Items (entrée)

### `message`

Rôles : `system`, `developer`, `user`, `assistant`.

- `system` et `developer` sont ajoutés au prompt système.
- L'item `user` ou `function_call_output` le plus récent devient le « message courant ».
- Les messages user/assistant antérieurs sont inclus comme historique pour le contexte.

### `function_call_output` (outils par tour)

Envoyez les résultats d'outils au modèle :

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### `reasoning` et `item_reference`

Acceptés pour la compatibilité de schéma mais ignorés lors de la construction du prompt.

## Outils (outils fonction côté client)

Fournissez des outils avec `tools: [{ type: "function", function: { name, description?, parameters? } }]`.

Si l'agent décide d'appeler un outil, la réponse renvoie un item de sortie `function_call`.
Vous envoyez ensuite une requête de suivi avec `function_call_output` pour continuer le tour.

## Images (`input_image`)

Supporte les sources base64 ou URL :

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Types MIME autorisés (actuels) : `image/jpeg`, `image/png`, `image/gif`, `image/webp`.
Taille maximale (actuelle) : 10 Mo.

## Fichiers (`input_file`)

Supporte les sources base64 ou URL :

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Types MIME autorisés (actuels) : `text/plain`, `text/markdown`, `text/html`, `text/csv`,
`application/json`, `application/pdf`.

Taille maximale (actuelle) : 5 Mo.

Comportement actuel :

- Le contenu du fichier est décodé et ajoute au **prompt système**, pas au message utilisateur, il reste donc éphémère (non persisté dans l'historique de session).
- Les PDFs sont analysés pour extraire le texte. Si peu de texte est trouvé, les premières pages sont rastérisées en images et passées au modèle.

L'analyse PDF utilisé le build legacy Node-friendly `pdfjs-dist` (sans worker). Le build moderne PDF.js attend des workers/globals DOM du navigateur, donc il n'est pas utilisé dans le Gateway.

Valeurs par défaut pour le fetch URL :

- `files.allowUrl` : `true`
- `images.allowUrl` : `true`
- `maxUrlParts` : `8` (total des parties `input_file` + `input_image` basées sur URL par requête)
- Les requêtes sont protégées (résolution DNS, blocage d'IP privée, limités de redirection, timeouts).
- Les listes autorisées de noms d'hôte optionnelles sont supportées par type d'entrée (`files.urlAllowlist`, `images.urlAllowlist`).
  - Hôte exact : `"cdn.example.com"`
  - Sous-domaines wildcard : `"*.assets.example.com"` (ne correspond pas à l'apex)

## Limités fichiers + images (configuration)

Les valeurs par défaut peuvent être ajustées sous `gateway.http.endpoints.responses` :

```json5
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: ["image/jpeg", "image/png", "image/gif", "image/webp"],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Valeurs par défaut lorsque omises :

- `maxBodyBytes` : 20 Mo
- `maxUrlParts` : 8
- `files.maxBytes` : 5 Mo
- `files.maxChars` : 200k
- `files.maxRedirects` : 3
- `files.timeoutMs` : 10s
- `files.pdf.maxPages` : 4
- `files.pdf.maxPixels` : 4 000 000
- `files.pdf.minTextChars` : 200
- `images.maxBytes` : 10 Mo
- `images.maxRedirects` : 3
- `images.timeoutMs` : 10s

Note de sécurité :

- Les listes autorisées d'URL sont appliquées avant le fetch et sur les sauts de redirection.
- Autoriser un nom d'hôte ne contourne pas le blocage d'IP privée/interne.
- Pour les gateways exposés à Internet, appliquez des contrôles d'egress réseau en plus des gardes au niveau applicatif.
  Voir [Sécurité](/gateway/security).

## Streaming (SSE)

Définissez `stream: true` pour recevoir des Server-Sent Events (SSE) :

- `Content-Type: text/event-stream`
- Chaque ligne d'événement est `event: <type>` et `data: <json>`
- Le flux se termine par `data: [DONE]`

Types d'événements actuellement émis :

- `response.created`
- `response.in_progress`
- `response.output_item.added`
- `response.content_part.added`
- `response.output_text.delta`
- `response.output_text.done`
- `response.content_part.done`
- `response.output_item.done`
- `response.completed`
- `response.failed` (en cas d'erreur)

## Usage

`usage` est rempli lorsque le fournisseur sous-jacent rapporte le nombre de jetons.

## Erreurs

Les erreurs utilisent un objet JSON comme :

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Cas courants :

- `401` authentification manquante/invalide
- `400` corps de requête invalide
- `405` mauvaise méthode

## Exemples

Non-streaming :

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Streaming :

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```
