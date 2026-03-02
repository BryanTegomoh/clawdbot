---
summary: "Adaptateurs RPC pour les CLI externes (signal-cli, legacy imsg) et schémas du Gateway"
read_when:
  - Ajout ou modification d'intégrations CLI externes
  - Débogage des adaptateurs RPC (signal-cli, imsg)
title: "Adaptateurs RPC"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/rpc.md
  workflow: manual
---

# Adaptateurs RPC

OpenClaw intègre des CLI externes via JSON-RPC. Deux schémas sont utilisés aujourd'hui.

## Schéma A : Démon HTTP (signal-cli)

- `signal-cli` s'exécute comme un démon avec JSON-RPC sur HTTP.
- Le flux d'événements est en SSE (`/api/v1/events`).
- Sonde de santé : `/api/v1/check`.
- OpenClaw gère le cycle de vie lorsque `channels.signal.autoStart=true`.

Voir [Signal](/channels/signal) pour la configuration et les points de terminaison.

## Schéma B : Processus enfant stdio (legacy : imsg)

> **Note :** Pour les nouvelles configurations iMessage, utilisez [BlueBubbles](/channels/bluebubbles) à la place.

- OpenClaw lance `imsg rpc` comme processus enfant (intégration iMessage legacy).
- JSON-RPC est délimité par lignes sur stdin/stdout (un objet JSON par ligne).
- Pas de port TCP, pas de démon requis.

Méthodes principales utilisées :

- `watch.subscribe` → notifications (`method: "message"`)
- `watch.unsubscribe`
- `send`
- `chats.list` (sonde/diagnostics)

Voir [iMessage](/channels/imessage) pour la configuration legacy et l'adressage (`chat_id` préféré).

## Directives pour les adaptateurs

- Le Gateway gère le processus (démarrage/arrêt lié au cycle de vie du fournisseur).
- Gardez les clients RPC résilients : timeouts, redémarrage en cas de sortie.
- Préférez les identifiants stables (ex. `chat_id`) aux chaînes d'affichage.
