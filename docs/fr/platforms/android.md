---
summary: "Application Android (nœud) : runbook de connexion + Canvas/Chat/Camera"
read_when:
  - Appairage ou reconnexion du nœud Android
  - Debogage de la decouverte du gateway Android ou de l'authentification
  - Vérification de la parite de l'historique de chat entre les clients
title: "Application Android"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/android.md
  workflow: manual
---

# Application Android (Nœud)

## Aperçu du support

- Rôle : application compagnon de type nœud (Android n'hébergé pas le Gateway).
- Gateway requis : oui (exécutez-le sur macOS, Linux ou Windows via WSL2).
- Installation : [Premiers pas](/start/getting-started) + [Appairage](/gateway/pairing).
- Gateway : [Runbook](/gateway) + [Configuration](/gateway/configuration).
  - Protocoles : [Protocole Gateway](/gateway/protocol) (nœuds + plan de contrôle).

## Contrôle système

Le contrôle système (launchd/systemd) reside sur l'hôte du Gateway. Voir [Gateway](/gateway).

## Runbook de connexion

Application nœud Android ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway**

Android se connecte directement au WebSocket du Gateway (par défaut `ws://<host>:18789`) et utilise l'appairage géré par le Gateway.

### Prerequis

- Vous pouvez exécuter le Gateway sur la machine « maitre ».
- L'appareil/emulateur Android peut atteindre le WebSocket du Gateway :
  - Même réseau local avec mDNS/NSD, **ou**
  - Même tailnet Tailscale utilisant Wide-Area Bonjour / unicast DNS-SD (voir ci-dessous), **ou**
  - Hôte/port du gateway manuel (solution de repli)
- Vous pouvez exécuter le CLI (`openclaw`) sur la machine du gateway (ou via SSH).

### 1) Démarrer le Gateway

```bash
openclaw gateway --port 18789 --verbose
```

Confirmez dans les logs que vous voyez quelque chose comme :

- `listening on ws://0.0.0.0:18789`

Pour les configurations tailnet uniquement (recommandé pour Vienna ⇄ London), liez le gateway a l'IP du tailnet :

- Définissez `gateway.bind: "tailnet"` dans `~/.openclaw/openclaw.json` sur l'hôte du gateway.
- Redemarrez le Gateway / l'application barre de menus macOS.

### 2) Vérifier la decouverte (optionnel)

Depuis la machine du gateway :

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Plus de notes de debogage : [Bonjour](/gateway/bonjour).

#### Decouverte via tailnet (Vienna ⇄ London) avec unicast DNS-SD

La decouverte Android NSD/mDNS ne traverse pas les réseaux. Si votre nœud Android et le gateway sont sur des réseaux differents mais connectés via Tailscale, utilisez plutot Wide-Area Bonjour / unicast DNS-SD :

1. Configurez une zone DNS-SD (exemple `openclaw.internal.`) sur l'hôte du gateway et publiez les enregistrements `_openclaw-gw._tcp`.
2. Configurez le split DNS de Tailscale pour votre domaine choisi pointant vers ce serveur DNS.

Details et exemple de configuration CoreDNS : [Bonjour](/gateway/bonjour).

### 3) Se connecter depuis Android

Dans l'application Android :

- L'application maintient sa connexion au gateway via un **service de premier plan** (notification persistante).
- Ouvrez **Paramètres**.
- Sous **Gateways decouverts**, selectionnez votre gateway et appuyez sur **Connecter**.
- Si mDNS est bloque, utilisez **Avance → Gateway manuel** (hôte + port) et **Connecter (Manuel)**.

Après le premier appairage reussi, Android se reconnecte automatiquement au lancement :

- Point d'accès manuel (si activé), sinon
- Le dernier gateway decouvert (meilleur effort).

### 4) Approuver l'appairage (CLI)

Sur la machine du gateway :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Details de l'appairage : [Appairage Gateway](/gateway/pairing).

### 5) Vérifier que le nœud est connecté

- Via le statut des nœuds :

  ```bash
  openclaw nodes status
  ```

- Via le Gateway :

  ```bash
  openclaw gateway call node.list --params "{}"
  ```

### 6) Chat + historique

La feuille Chat du nœud Android utilisé la **clé de session principale** du gateway (`main`), donc l'historique et les réponses sont partages avec WebChat et les autres clients :

- Historique : `chat.history`
- Envoyer : `chat.send`
- Mises a jour push (meilleur effort) : `chat.subscribe` → `event:"chat"`

### 7) Canvas + camera

#### Hôte Canvas du Gateway (recommandé pour le contenu web)

Si vous souhaitez que le nœud affiche du vrai HTML/CSS/JS que l'agent peut modifier sur le disque, pointez le nœud vers l'hôte Canvas du Gateway.

Note : les nœuds chargent le canvas depuis le serveur HTTP du Gateway (même port que `gateway.port`, par défaut `18789`).

1. Creez `~/.openclaw/workspace/canvas/index.html` sur l'hôte du gateway.

2. Naviguez le nœud vers celui-ci (LAN) :

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

Tailnet (optionnel) : si les deux appareils sont sur Tailscale, utilisez un nom MagicDNS ou une IP tailnet au lieu de `.local`, par exemple `http://<gateway-magicdns>:18789/__openclaw__/canvas/`.

Ce serveur injecte un client de rechargement en direct dans le HTML et recharge lors des modifications de fichiers.
L'hôte A2UI se trouve a `http://<gateway-host>:18789/__openclaw__/a2ui/`.

Commandes Canvas (premier plan uniquement) :

- `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (utilisez `{"url":""}` ou `{"url":"/"}` pour revenir au scaffold par défaut). `canvas.snapshot` retourne `{ format, base64 }` (par défaut `format="jpeg"`).
- A2UI : `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` alias ancien)

Commandes Camera (premier plan uniquement ; soumises aux permissions) :

- `camera.snap` (jpg)
- `camera.clip` (mp4)

Voir [Nœud Camera](/nodes/camera) pour les paramètres et les helpers CLI.
