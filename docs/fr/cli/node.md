---
summary: "Référence CLI pour `openclaw node` (node host headless)"
read_when:
  - Exécution du node host headless
  - Appairage d'un node non macOS pour system.run
title: "node"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/node.md
  workflow: manual
---

# `openclaw node`

Exécuter un **node host headless** qui se connecte au WebSocket du Gateway et expose `system.run` / `system.which` sur cette machine.

## Pourquoi utiliser un node host ?

Utilisez un node host lorsque vous souhaitez que les agents **exécutent des commandes sur d'autres machines** de votre réseau sans installer une application macOS complète.

Cas d'utilisation courants :

- Exécuter des commandes sur des machines Linux/Windows distantes (serveurs de build, machines de laboratoire, NAS).
- Garder l'exécution en **bac à sable** sur le Gateway, mais déléguer les exécutions approuvées à d'autres hôtes.
- Fournir une cible d'exécution légère et headless pour l'automatisation ou les nodes CI.

L'exécution est toujours gardée par les **approbations d'exécution** et les listes d'autorisation par agent sur le node host, donc vous pouvez garder l'accès aux commandes limité et explicite.

## Proxy navigateur (zéro configuration)

Les node hosts annoncent automatiquement un proxy navigateur si `browser.enabled` n'est pas désactivé sur le node. Cela permet à l'agent d'utiliser l'automatisation du navigateur sur ce node sans configuration supplémentaire.

Désactivez-le sur le node si nécessaire :

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## Exécuter (avant-plan)

```bash
openclaw node run --host <gateway-host> --port 18789
```

Options :

- `--host <host>` : hôte WebSocket du Gateway (défaut : `127.0.0.1`)
- `--port <port>` : port WebSocket du Gateway (défaut : `18789`)
- `--tls` : utiliser TLS pour la connexion au Gateway
- `--tls-fingerprint <sha256>` : empreinte attendue du certificat TLS (sha256)
- `--node-id <id>` : surcharger l'identifiant du node (efface le jeton d'appairage)
- `--display-name <name>` : surcharger le nom d'affichage du node

## Service (arrière-plan)

Installer un node host headless comme service utilisateur.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Options :

- `--host <host>` : hôte WebSocket du Gateway (défaut : `127.0.0.1`)
- `--port <port>` : port WebSocket du Gateway (défaut : `18789`)
- `--tls` : utiliser TLS pour la connexion au Gateway
- `--tls-fingerprint <sha256>` : empreinte attendue du certificat TLS (sha256)
- `--node-id <id>` : surcharger l'identifiant du node (efface le jeton d'appairage)
- `--display-name <name>` : surcharger le nom d'affichage du node
- `--runtime <runtime>` : runtime du service (`node` ou `bun`)
- `--force` : réinstaller/écraser si déjà installé

Gérer le service :

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Utilisez `openclaw node run` pour un node host en avant-plan (sans service).

Les commandes de service acceptent `--json` pour une sortie lisible par machine.

## Appairage

La première connexion crée une demande d'appairage de node en attente sur le Gateway.
Approuvez-la via :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Le node host stocke son identifiant de node, son jeton, son nom d'affichage et les informations de connexion au Gateway dans `~/.openclaw/node.json`.

## Approbations d'exécution

`system.run` est contrôlé par les approbations d'exécution locales :

- `~/.openclaw/exec-approvals.json`
- [Exec approvals](/tools/exec-approvals)
- `openclaw approvals --node <id|name|ip>` (modifier depuis le Gateway)
