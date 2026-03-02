---
title: "API d'invocation des outils"
summary: "Invoquer un outil unique directement via le point de terminaison HTTP du Gateway"
read_when:
  - Appeler des outils sans exécuter un tour complet d'agent
  - Construire des automatisations nécessitant l'application de la politique d'outils
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/tools-invoke-http-api.md
  workflow: manual
---

# Invocation des outils (HTTP)

Le Gateway d'OpenClaw expose un point de terminaison HTTP simple pour invoquer un outil unique directement. Il est toujours activé, mais protégé par l'authentification du Gateway et la politique d'outils.

- `POST /tools/invoke`
- Même port que le Gateway (multiplexage WS + HTTP) : `http://<hôte-gateway>:<port>/tools/invoke`

La taille maximale de la charge utile par défaut est de 2 Mo.

## Authentification

Utilisé la configuration d'authentification du Gateway. Envoyez un jeton porteur :

- `Authorization: Bearer <jeton>`

Notes :

- Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
- Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).
- Si `gateway.auth.rateLimit` est configuré et que trop d'échecs d'authentification se produisent, le point de terminaison renvoie `429` avec `Retry-After`.

## Corps de la requête

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Champs :

- `tool` (chaîne, requis) : nom de l'outil à invoquer.
- `action` (chaîne, optionnel) : mappé dans les arguments si le schéma de l'outil prend en charge `action` et que la charge utile des arguments l'a omis.
- `args` (objet, optionnel) : arguments spécifiques à l'outil.
- `sessionKey` (chaîne, optionnel) : clé de session cible. Si omis ou `"main"`, le Gateway utilise la clé de session principale configurée (respecte `session.mainKey` et l'agent par défaut, ou `global` dans la portée globale).
- `dryRun` (booléen, optionnel) : réservé pour un usage futur ; actuellement ignoré.

## Comportement de politique et routage

La disponibilité des outils est filtrée à travers la même chaîne de politique utilisée par les agents du Gateway :

- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- politiques de groupe (si la clé de session correspond à un groupe ou un canal)
- politique de sous-agent (lors de l'invocation avec une clé de session de sous-agent)

Si un outil n'est pas autorisé par la politique, le point de terminaison renvoie **404**.

Le Gateway HTTP applique également une liste de refus stricte par défaut (même si la politique de session autorise l'outil) :

- `sessions_spawn`
- `sessions_send`
- `gateway`
- `whatsapp_login`

Vous pouvez personnaliser cette liste de refus via `gateway.tools` :

```json5
{
  gateway: {
    tools: {
      // Outils supplémentaires à bloquer via HTTP /tools/invoke
      deny: ["browser"],
      // Retirer des outils de la liste de refus par défaut
      allow: ["gateway"],
    },
  },
}
```

Pour aider les politiques de groupe à résoudre le contexte, vous pouvez optionnellement définir :

- `x-openclaw-message-channel: <canal>` (exemple : `slack`, `telegram`)
- `x-openclaw-account-id: <accountId>` (lorsque plusieurs comptes existent)

## Réponses

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }` (requête invalide ou erreur d'entrée de l'outil)
- `401` → non autorise
- `429` → limitation de débit d'authentification (`Retry-After` défini)
- `404` → outil non disponible (introuvable ou non dans la liste d'autorisation)
- `405` → méthode non autorisée
- `500` → `{ ok: false, error: { type, message } }` (erreur inattendue d'exécution de l'outil ; message assaini)

## Exemple

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer VOTRE_JETON' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
