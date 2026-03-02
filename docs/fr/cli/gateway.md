---
summary: "CLI Gateway OpenClaw (`openclaw gateway`) : exécuter, interroger et découvrir des Gateways"
read_when:
  - Exécution du Gateway depuis le CLI (dev ou serveurs)
  - Débogage de l'authentification Gateway, modes de liaison et connectivité
  - Découverte de Gateways via Bonjour (LAN + tailnet)
title: "gateway"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/gateway.md
  workflow: manual
---

# CLI Gateway

Le Gateway est le serveur WebSocket d'OpenClaw (canaux, nodes, sessions, hooks).

Les sous-commandes de cette page se trouvent sous `openclaw gateway ...`.

Documentation liée :

- [/gateway/bonjour](/gateway/bonjour)
- [/gateway/discovery](/gateway/discovery)
- [/gateway/configuration](/gateway/configuration)

## Exécuter le Gateway

Exécuter un processus Gateway local :

```bash
openclaw gateway
```

Alias d'avant-plan :

```bash
openclaw gateway run
```

Notes :

- Par défaut, le Gateway refuse de démarrer sauf si `gateway.mode=local` est défini dans `~/.openclaw/openclaw.json`. Utilisez `--allow-unconfigured` pour les exécutions ad-hoc/dev.
- La liaison au-delà du loopback sans authentification est bloquée (garde-fou de sécurité).
- `SIGUSR1` déclenche un redémarrage en cours de processus lorsque autorisé (`commands.restart` est activé par défaut ; définissez `commands.restart: false` pour bloquer le redémarrage manuel, tandis que les outils Gateway/config apply/update restent autorisés).
- Les gestionnaires `SIGINT`/`SIGTERM` arrêtent le processus Gateway, mais ils ne restaurent pas l'état du terminal personnalisé. Si vous encapsulez le CLI avec un TUI ou une entrée en mode brut, restaurez le terminal avant la sortie.

### Options

- `--port <port>` : port WebSocket (la valeur par défaut provient de la configuration/env ; généralement `18789`).
- `--bind <loopback|lan|tailnet|auto|custom>` : mode de liaison de l'écouteur.
- `--auth <token|password>` : surcharge du mode d'authentification.
- `--token <token>` : surcharge du jeton (définit également `OPENCLAW_GATEWAY_TOKEN` pour le processus).
- `--password <password>` : surcharge du mot de passe (définit également `OPENCLAW_GATEWAY_PASSWORD` pour le processus).
- `--tailscale <off|serve|funnel>` : exposer le Gateway via Tailscale.
- `--tailscale-reset-on-exit` : réinitialiser la configuration Tailscale serve/funnel à l'arrêt.
- `--allow-unconfigured` : autoriser le démarrage du Gateway sans `gateway.mode=local` dans la configuration.
- `--dev` : créer une configuration dev + espace de travail si manquant (ignoré BOOTSTRAP.md).
- `--reset` : réinitialiser la configuration dev + identifiants + sessions + espace de travail (nécessite `--dev`).
- `--force` : tuer tout écouteur existant sur le port sélectionné avant le démarrage.
- `--verbose` : logs détaillés.
- `--claude-cli-logs` : afficher uniquement les logs claude-cli dans la console (et activer ses stdout/stderr).
- `--ws-log <auto|full|compact>` : style de log websocket (défaut `auto`).
- `--compact` : alias de `--ws-log compact`.
- `--raw-stream` : enregistrer les événements bruts du flux de modèles en jsonl.
- `--raw-stream-path <path>` : chemin jsonl du flux brut.

## Interroger un Gateway en cours d'exécution

Toutes les commandes d'interrogation utilisent le RPC WebSocket.

Modes de sortie :

- Défaut : lisible par l'homme (colore en TTY).
- `--json` : JSON lisible par machine (pas de style/spinner).
- `--no-color` (ou `NO_COLOR=1`) : désactivé ANSI tout en conservant la mise en page humaine.

Options partagées (lorsque supportées) :

- `--url <url>` : URL WebSocket du Gateway.
- `--token <token>` : jeton du Gateway.
- `--password <password>` : mot de passe du Gateway.
- `--timeout <ms>` : délai d'expiration/budget (varie par commande).
- `--expect-final` : attendre une réponse "finale" (appels d'agent).

Note : lorsque vous définissez `--url`, le CLI ne se replie pas sur les identifiants de la configuration ou de l'environnement.
Passez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

### `gateway health`

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### `gateway status`

`gateway status` affiche le service Gateway (launchd/systemd/schtasks) plus une sonde RPC optionnelle.

```bash
openclaw gateway status
openclaw gateway status --json
```

Options :

- `--url <url>` : surcharger l'URL de la sonde.
- `--token <token>` : authentification par jeton pour la sonde.
- `--password <password>` : authentification par mot de passe pour la sonde.
- `--timeout <ms>` : délai d'expiration de la sonde (défaut `10000`).
- `--no-probe` : ignorer la sonde RPC (vue service uniquement).
- `--deep` : analyser également les services au niveau système.

### `gateway probe`

`gateway probe` est la commande "tout déboguer". Elle sonde toujours :

- votre Gateway distant configuré (si défini), et
- localhost (loopback) **même si le distant est configuré**.

Si plusieurs Gateways sont joignables, la commande les affiche tous. Plusieurs Gateways sont supportés lorsque vous utilisez des profils/ports isolés (par ex., un bot de secours), mais la plupart des installations exécutent un seul Gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### Distant via SSH (parité application Mac)

Le mode "Remote over SSH" de l'application macOS utilise un port-forward local pour que le Gateway distant (qui peut être lié au loopback uniquement) devienne joignable à `ws://127.0.0.1:<port>`.

Équivalent CLI :

```bash
openclaw gateway probe --ssh user@gateway-host
```

Options :

- `--ssh <target>` : `user@host` ou `user@host:port` (le port est par défaut `22`).
- `--ssh-identity <path>` : fichier d'identité.
- `--ssh-auto` : choisir le premier hôte Gateway découvert comme cible SSH (LAN/WAB uniquement).

Configuration (optionnelle, utilisée comme valeurs par défaut) :

- `gateway.remote.sshTarget`
- `gateway.remote.sshIdentity`

### `gateway call <method>`

Utilitaire RPC de bas niveau.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Gérer le service Gateway

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Notes :

- `gateway install` supporte `--port`, `--runtime`, `--token`, `--force`, `--json`.
- Les commandes de cycle de vie acceptent `--json` pour le scripting.

## Découvrir des Gateways (Bonjour)

`gateway discover` recherche les balises Gateway (`_openclaw-gw._tcp`).

- DNS-SD multicast : `local.`
- DNS-SD unicast (Wide-Area Bonjour) : choisissez un domaine (exemple : `openclaw.internal.`) et configurez le DNS fractionné + un serveur DNS ; voir [/gateway/bonjour](/gateway/bonjour)

Seuls les Gateways avec la découverte Bonjour activée (par défaut) annoncent la balise.

Les enregistrements de découverte Wide-Area incluent (TXT) :

- `rôle` (indication du rôle du Gateway)
- `transport` (indication du transport, par ex. `gateway`)
- `gatewayPort` (port WebSocket, généralement `18789`)
- `sshPort` (port SSH ; défaut `22` si absent)
- `tailnetDns` (nom d'hôte MagicDNS, lorsque disponible)
- `gatewayTls` / `gatewayTlsSha256` (TLS activé + empreinte du certificat)
- `cliPath` (indication optionnelle pour les installations distantes)

### `gateway discover`

```bash
openclaw gateway discover
```

Options :

- `--timeout <ms>` : délai d'expiration par commande (browse/resolve) ; défaut `2000`.
- `--json` : sortie lisible par machine (désactive également le style/spinner).

Exemples :

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
