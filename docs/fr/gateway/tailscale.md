---
title: "Tailscale"
summary: "Tailscale Serve/Funnel intégré pour le tableau de bord du Gateway"
read_when:
  - Exposer l'interface de contrôle du Gateway en dehors de localhost
  - Automatiser l'accès au tableau de bord via tailnet ou public
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/tailscale.md
  workflow: manual
---

# Tailscale (tableau de bord du Gateway)

OpenClaw peut auto-configurer Tailscale **Serve** (tailnet) ou **Funnel** (public) pour
le tableau de bord du Gateway et le port WebSocket. Cela garde le Gateway lie à loopback tandis que
Tailscale fournit le HTTPS, le routage et (pour Serve) les en-têtes d'identité.

## Modes

- `serve` : Serve uniquement sur le tailnet via `tailscale serve`. Le gateway reste sur `127.0.0.1`.
- `funnel` : HTTPS public via `tailscale funnel`. OpenClaw requiert un mot de passe partagé.
- `off` : Par défaut (pas d'automatisation Tailscale).

## Authentification

Définissez `gateway.auth.mode` pour contrôler la négociation :

- `token` (par défaut quand `OPENCLAW_GATEWAY_TOKEN` est défini)
- `password` (secret partagé via `OPENCLAW_GATEWAY_PASSWORD` ou configuration)

Lorsque `tailscale.mode = "serve"` et que `gateway.auth.allowTailscale` est `true`,
l'authentification de l'interface de contrôle/WebSocket peut utiliser les en-têtes d'identité Tailscale
(`tailscale-user-login`) sans fournir de jeton/mot de passe. OpenClaw vérifie
l'identité en résolvant l'adresse `x-forwarded-for` via le démon Tailscale local
(`tailscale whois`) et en la comparant à l'en-tête avant de l'accepter.
OpenClaw ne traite une requête comme Serve que lorsqu'elle arrive de loopback avec
les en-têtes `x-forwarded-for`, `x-forwarded-proto` et `x-forwarded-host` de Tailscale.
Les points de terminaison de l'API HTTP (par exemple `/v1/*`, `/tools/invoke` et `/api/channels/*`)
requièrent toujours une authentification par jeton/mot de passe.
Ce flux sans jeton suppose que l'hôte du gateway est de confiance. Si du code local non fiable
peut s'exécuter sur le même hôte, désactivez `gateway.auth.allowTailscale` et exigez
une authentification par jeton/mot de passe à la place.
Pour exiger des identifiants explicites, définissez `gateway.auth.allowTailscale: false` ou
forcez `gateway.auth.mode: "password"`.

## Exemples de configuration

### Tailnet uniquement (Serve)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Ouvrir : `https://<magicdns>/` (ou votre `gateway.controlUi.basePath` configuré)

### Tailnet uniquement (liaison à l'IP Tailnet)

Utilisez ceci lorsque vous voulez que le Gateway écoute directement sur l'IP Tailnet (sans Serve/Funnel).

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "votre-jeton" },
  },
}
```

Se connecter depuis un autre appareil Tailnet :

- Interface de contrôle : `http://<ip-tailscale>:18789/`
- WebSocket : `ws://<ip-tailscale>:18789`

Note : loopback (`http://127.0.0.1:18789`) ne fonctionnera **pas** dans ce mode.

### Internet public (Funnel + mot de passe partagé)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "remplacez-moi" },
  },
}
```

Préférez `OPENCLAW_GATEWAY_PASSWORD` plutôt que de stocker un mot de passe sur disque.

## Exemples CLI

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## Notes

- Tailscale Serve/Funnel nécessite que le CLI `tailscale` soit installé et connecte.
- `tailscale.mode: "funnel"` refuse de démarrer sauf si le mode d'authentification est `password` pour éviter l'exposition publique.
- Définissez `gateway.tailscale.resetOnExit` si vous voulez qu'OpenClaw annule la configuration
  de `tailscale serve` ou `tailscale funnel` à l'arrêt.
- `gateway.bind: "tailnet"` est une liaison directe au Tailnet (pas de HTTPS, pas de Serve/Funnel).
- `gateway.bind: "auto"` préfère loopback ; utilisez `tailnet` si vous voulez uniquement le Tailnet.
- Serve/Funnel n'exposent que l'**interface de contrôle du Gateway + WS**. Les nœuds se connectent via
  le même point de terminaison WS du Gateway, donc Serve peut fonctionner pour l'accès aux nœuds.

## Contrôle du navigateur (Gateway distant + navigateur local)

Si vous exécutez le Gateway sur une machine mais voulez piloter un navigateur sur une autre machine,
exécutez un **hôte de nœud** sur la machine du navigateur et gardez les deux sur le même tailnet.
Le Gateway transmettra les actions du navigateur au nœud ; aucun serveur de contrôle séparé ou URL Serve n'est nécessaire.

Évitez Funnel pour le contrôle du navigateur ; traitez l'appairage de nœud comme un accès opérateur.

## Prérequis et limitations de Tailscale

- Serve nécessite que le HTTPS soit activé pour votre tailnet ; le CLI vous invite si c'est manquant.
- Serve injecte les en-têtes d'identité Tailscale ; Funnel ne le fait pas.
- Funnel nécessite Tailscale v1.38.3+, MagicDNS, HTTPS activé et un attribut de nœud funnel.
- Funnel ne prend en charge que les ports `443`, `8443` et `10000` via TLS.
- Funnel sur macOS nécessite la variante open source de l'application Tailscale.

## En savoir plus

- Présentation de Tailscale Serve : [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
- Commande `tailscale serve` : [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
- Présentation de Tailscale Funnel : [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
- Commande `tailscale funnel` : [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)
