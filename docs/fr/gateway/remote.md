---
summary: "Accès distant via tunnels SSH (Gateway WS) et tailnets"
read_when:
  - Exécution ou dépannage de configurations de gateway distant
title: "Accès distant"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/remote.md
  workflow: manual
---

# Accès distant (SSH, tunnels et tailnets)

Ce dépôt prend en charge le mode « distant via SSH » en maintenant un seul Gateway (le maître) en fonctionnement sur un hôte dédié (bureau/serveur) et en y connectant les clients.

- Pour les **opérateurs (vous / l'application macOS)** : le tunnel SSH est la solution universelle de repli.
- Pour les **nœuds (iOS/Android et futurs appareils)** : connectez-vous au **WebSocket** du Gateway (LAN/tailnet ou tunnel SSH selon les besoins).

## L'idée principale

- Le WebSocket du Gateway se lie au **loopback** sur votre port configuré (par défaut 18789).
- Pour une utilisation distante, vous redirigez ce port loopback via SSH (ou utilisez un tailnet/VPN pour réduire le tunneling).

## Configurations VPN/tailnet courantes (ou réside l'agent)

Considérez l'**hôte du Gateway** comme « l'endroit ou réside l'agent ». Il possède les sessions, les profils d'authentification, les canaux et l'état.
Votre portable/bureau (et les nœuds) se connectent a cet hôte.

### 1) Gateway permanent dans votre tailnet (VPS ou serveur domestique)

Exécutez le Gateway sur un hôte persistant et accédez-y via **Tailscale** ou SSH.

- **Meilleure UX :** gardez `gateway.bind: "loopback"` et utilisez **Tailscale Serve** pour l'interface de contrôle.
- **Repli :** gardez loopback + tunnel SSH depuis toute machine nécessitant un accès.
- **Exemples :** [exe.dev](/install/exe-dev) (VM facile) ou [Hetzner](/install/hetzner) (VPS production).

C'est idéal quand votre portable se met souvent en veille mais que vous voulez que l'agent soit toujours actif.

### 2) Le bureau domestique exécute le Gateway, le portable est la télécommande

Le portable **n'exécute pas** l'agent. Il se connecte a distance :

- Utilisez le mode **Remote over SSH** de l'application macOS (Réglages -> Général -> "OpenClaw runs").
- L'application ouvre et gère le tunnel, donc WebChat + les vérifications de sante « fonctionnent directement ».

Runbook : [accès distant macOS](/platforms/mac/remote).

### 3) Le portable exécute le Gateway, accès distant depuis d'autres machines

Gardez le Gateway local mais exposez-le en toute sécurité :

- Tunnel SSH vers le portable depuis d'autres machines, ou
- Tailscale Serve l'interface de contrôle et gardez le Gateway en loopback uniquement.

Guide : [Tailscale](/gateway/tailscale) et [Vue d'ensemble web](/web).

## Flux de commandes (qu'est-ce qui s'exécute ou)

Un seul service gateway possède l'état + les canaux. Les nœuds sont des périphériques.

Exemple de flux (Telegram -> nœud) :

- Le message Telegram arrive au **Gateway**.
- Le Gateway exécute l'**agent** et decide s'il faut appeler un outil de nœud.
- Le Gateway appelle le **nœud** via le WebSocket du Gateway (`node.*` RPC).
- Le nœud retourne le résultat ; le Gateway répond vers Telegram.

Notes :

- **Les nœuds n'exécutent pas le service gateway.** Un seul gateway devrait fonctionner par hôte sauf si vous exécutez intentionnellement des profils isoles (voir [Plusieurs gateways](/gateway/multiple-gateways)).
- Le « mode nœud » de l'application macOS est simplement un client nœud via le WebSocket du Gateway.

## Tunnel SSH (CLI + outils)

Créez un tunnel local vers le WS du Gateway distant :

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Avec le tunnel actif :

- `openclaw health` et `openclaw status --deep` atteignent désormais le gateway distant via `ws://127.0.0.1:18789`.
- `openclaw gateway {status,health,send,agent,call}` peut aussi cibler l'URL redirigée via `--url` si nécessaire.

Note : remplacez `18789` par votre `gateway.port` configuré (ou `--port`/`OPENCLAW_GATEWAY_PORT`).
Note : quand vous passez `--url`, le CLI ne se rabat pas sur les identifiants de configuration ou d'environnement.
Incluez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

## Valeurs par défaut distantes du CLI

Vous pouvez persister une cible distante pour que les commandes CLI l'utilisent par défaut :

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

Quand le gateway est en loopback uniquement, gardez l'URL a `ws://127.0.0.1:18789` et ouvrez d'abord le tunnel SSH.

## Précédence des identifiants

La résolution des identifiants pour les appels/sondes du Gateway suit désormais un contrat unique partage :

- Les identifiants explicites (`--token`, `--password` ou `gatewayToken` de l'outil) ont toujours la priorité.
- Valeurs par défaut en mode local :
  - token : `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` -> `gateway.remote.token`
  - password : `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.auth.password` -> `gateway.remote.password`
- Valeurs par défaut en mode distant :
  - token : `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token`
  - password : `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.remote.password` -> `gateway.auth.password`
- Les vérifications de token de sonde/statut distant sont strictes par défaut : elles utilisent uniquement `gateway.remote.token` (pas de repli sur le token local) en mode distant.
- Les variables d'environnement anciennes `CLAWDBOT_GATEWAY_*` ne sont utilisées que par les chemins d'appel de compatibilité ; la résolution sonde/statut/authentification utilise uniquement `OPENCLAW_GATEWAY_*`.

## Interface de chat via SSH

WebChat n'utilisé plus un port HTTP séparé. L'interface de chat SwiftUI se connecte directement au WebSocket du Gateway.

- Redirigez `18789` via SSH (voir ci-dessus), puis connectez les clients a `ws://127.0.0.1:18789`.
- Sur macOS, préférez le mode « Remote over SSH » de l'application, qui gère le tunnel automatiquement.

## Application macOS « Remote over SSH »

L'application de barre de menu macOS peut piloter la même configuration de bout en bout (vérifications de statut distant, WebChat et redirection Voice Wake).

Runbook : [accès distant macOS](/platforms/mac/remote).

## Règles de sécurité (distant/VPN)

En bref : **gardez le Gateway en loopback uniquement** sauf si vous êtes sur d'avoir besoin d'un bind.

- **Loopback + SSH/Tailscale Serve** est la valeur par défaut la plus sûre (pas d'exposition publique).
- Les **binds non-loopback** (`lan`/`tailnet`/`custom`, ou `auto` quand loopback n'est pas disponible) doivent utiliser des tokens/mots de passe d'authentification.
- `gateway.remote.token` / `.password` sont des sources d'identifiants client. Ils **ne configurent pas** l'authentification serveur par eux-mêmes.
- Les chemins d'appel locaux peuvent utiliser `gateway.remote.*` comme repli quand `gateway.auth.*` n'est pas défini.
- `gateway.remote.tlsFingerprint` épingle le certificat TLS distant lors de l'utilisation de `wss://`.
- **Tailscale Serve** peut authentifier le trafic de l'interface de contrôle/WebSocket via des en-têtes d'identité quand `gateway.auth.allowTailscale: true` ; les points de terminaison de l'API HTTP nécessitent toujours une authentification par token/mot de passe. Ce flux sans token suppose que l'hôte du gateway est de confiance. Mettez-le a `false` si vous voulez des tokens/mots de passe partout.
- Traitez le contrôle du navigateur comme un accès opérateur : tailnet uniquement + appairage délibéré des nœuds.

Approfondissement : [Sécurité](/gateway/security).
