---
summary: "Protocole Bridge (nœuds legacy) : TCP JSONL, appairage, RPC limité"
read_when:
  - Développement ou débogage de clients nœud (iOS/Android/macOS en mode nœud)
  - Investigation des échecs d'appairage ou d'authentification bridge
  - Audit de la surface nœud exposée par le gateway
title: "Protocole Bridge"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/bridge-protocol.md
  workflow: manual
---

# Protocole Bridge (transport nœud legacy)

Le protocole Bridge est un transport nœud **legacy** (TCP JSONL). Les nouveaux clients nœud doivent utiliser le protocole WebSocket unifié du Gateway à la place.

Si vous développez un client opérateur ou nœud, utilisez le [protocole Gateway](/gateway/protocol).

**Note :** Les versions actuelles d'OpenClaw ne livrent plus le listener TCP bridge ; ce document est conservé comme référence historique.
Les clés de configuration legacy `bridge.*` ne font plus partie du schéma de configuration.

## Pourquoi nous avons les deux

- **Frontière de sécurité** : le bridge expose une petite liste autorisée au lieu de la surface API complète du gateway.
- **Appairage + identité de nœud** : l'admission des nœuds est gérée par le gateway et liée à un jeton par nœud.
- **UX de découverte** : les nœuds peuvent découvrir les gateways via Bonjour sur le LAN, ou se connecter directement via un tailnet.
- **WS loopback** : le plan de contrôle WS complet reste local sauf s'il est acheminé via un tunnel SSH.

## Transport

- TCP, un objet JSON par ligne (JSONL).
- TLS optionnel (lorsque `bridge.tls.enabled` est true).
- Le port par défaut du listener legacy était `18790` (les versions actuelles ne démarrent pas de bridge TCP).

Lorsque TLS est activé, les enregistrements TXT de découverte incluent `bridgeTls=1` plus `bridgeTlsSha256` comme indice non secret. Notez que les enregistrements TXT Bonjour/mDNS ne sont pas authentifiés ; les clients ne doivent pas traiter l'empreinte annoncée comme un épinglage faisant autorité sans intention explicite de l'utilisateur ou autre vérification hors bande.

## Handshake + appairage

1. Le client envoie `hello` avec les métadonnées du nœud + jeton (si déjà appairé).
2. Si non appairé, le gateway répond `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3. Le client envoie `pair-request`.
4. Le gateway attend l'approbation, puis envoie `pair-ok` et `hello-ok`.

`hello-ok` renvoie `serverName` et peut inclure `canvasHostUrl`.

## Trames

Client → Gateway :

- `req` / `res` : RPC gateway limité (chat, sessions, config, health, voicewake, skills.bins)
- `event` : signaux du nœud (transcription vocale, requête agent, abonnement chat, cycle de vie exec)

Gateway → Client :

- `invoke` / `invoke-res` : commandes nœud (`canvas.*`, `camera.*`, `screen.record`, `location.get`, `sms.send`)
- `event` : mises à jour de chat pour les sessions abonnées
- `ping` / `pong` : keepalive

L'application de la liste autorisée legacy se trouvait dans `src/gateway/server-bridge.ts` (supprimé).

## Événements du cycle de vie exec

Les nœuds peuvent émettre des événements `exec.finished` ou `exec.denied` pour faire remonter l'activité system.run. Ceux-ci sont mappés vers des événements système dans le gateway. (Les nœuds legacy peuvent encore émettre `exec.started`.)

Champs de la charge utile (tous optionnels sauf indication contraire) :

- `sessionKey` (obligatoire) : session agent pour recevoir l'événement système.
- `runId` : identifiant exec unique pour le regroupement.
- `command` : chaîne de commande brute ou formatée.
- `exitCode`, `timedOut`, `success`, `output` : détails de complétion (terminé uniquement).
- `reason` : raison du refus (refusé uniquement).

## Utilisation Tailnet

- Liez le bridge à une IP tailnet : `bridge.bind: "tailnet"` dans `~/.openclaw/openclaw.json`.
- Les clients se connectent via le nom MagicDNS ou l'IP tailnet.
- Bonjour ne **traverse pas** les réseaux ; utilisez hôte/port manuels ou DNS-SD étendu si nécessaire.

## Versionnage

Le Bridge est actuellement en **v1 implicite** (pas de négociation min/max). La rétrocompatibilité est attendue ; ajoutez un champ de version du protocole bridge avant toute modification incompatible.
