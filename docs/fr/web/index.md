---
summary: "Surfaces web du Gateway : Interface de contrôle, modes de liaison et sécurité"
read_when:
  - Vous souhaitez accéder au Gateway via Tailscale
  - Vous souhaitez utiliser l'Interface de contrôle dans le navigateur et modifier la configuration
title: "Web"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/web/index.md
  workflow: manual
---

# Web (Gateway)

Le Gateway sert une petite **Interface de contrôle dans le navigateur** (Vite + Lit) sur le même port que le WebSocket du Gateway :

- par défaut : `http://<host>:18789/`
- préfixe optionnel : définir `gateway.controlUi.basePath` (ex. `/openclaw`)

Les fonctionnalités sont décrites dans [Interface de contrôle](/web/control-ui).
Cette page se concentre sur les modes de liaison, la sécurité et les surfaces exposées sur le web.

## Webhooks

Lorsque `hooks.enabled=true`, le Gateway expose également un petit point de terminaison webhook sur le même serveur HTTP.
Voir [Configuration du Gateway](/gateway/configuration) → `hooks` pour l'authentification et les charges utiles.

## Configuration (activée par défaut)

L'Interface de contrôle est **activée par défaut** lorsque les ressources sont présentes (`dist/control-ui`).
Vous pouvez la contrôler via la configuration :

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath optionnel
  },
}
```

## Accès Tailscale

### Serve intégré (recommandé)

Gardez le Gateway en boucle locale et laissez Tailscale Serve le rediriger via un proxy :

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Puis démarrez le gateway :

```bash
openclaw gateway
```

Ouvrir :

- `https://<magicdns>/` (ou votre `gateway.controlUi.basePath` configuré)

### Liaison tailnet + jeton

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

Puis démarrez le gateway (jeton requis pour les liaisons non-loopback) :

```bash
openclaw gateway
```

Ouvrir :

- `http://<tailscale-ip>:18789/` (ou votre `gateway.controlUi.basePath` configuré)

### Internet public (Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // ou OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## Notes de sécurité

- L'authentification du Gateway est requise par défaut (jeton/mot de passe ou en-têtes d'identité Tailscale).
- Les liaisons non-loopback **nécessitent** toujours un jeton/mot de passe partagé (`gateway.auth` ou variable d'environnement).
- L'assistant de configuration initiale génère un jeton de gateway par défaut (même en loopback).
- L'interface envoie `connect.params.auth.token` ou `connect.params.auth.password`.
- Pour les déploiements de l'Interface de contrôle non-loopback, définissez `gateway.controlUi.allowedOrigins`
  explicitement (origines complètes). Sans cela, le démarrage du gateway est refusé par défaut.
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` activé le
  mode de repli sur l'en-tête Host pour l'origine, mais constitue une dégradation dangereuse de la sécurité.
- Avec Serve, les en-têtes d'identité Tailscale peuvent satisfaire l'authentification de l'Interface de contrôle/WebSocket
  lorsque `gateway.auth.allowTailscale` est `true` (aucun jeton/mot de passe requis).
  Les points de terminaison de l'API HTTP nécessitent toujours un jeton/mot de passe. Définissez
  `gateway.auth.allowTailscale: false` pour exiger des identifiants explicites. Voir
  [Tailscale](/gateway/tailscale) et [Sécurité](/gateway/security). Ce
  flux sans jeton suppose que l'hôte du gateway est de confiance.
- `gateway.tailscale.mode: "funnel"` nécessite `gateway.auth.mode: "password"` (mot de passe partagé).

## Construction de l'interface

Le Gateway sert des fichiers statiques depuis `dist/control-ui`. Construisez-les avec :

```bash
pnpm ui:build # installe automatiquement les dépendances UI au premier lancement
```
