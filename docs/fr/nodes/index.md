---
summary: "Nœuds : appairage, capacités, permissions et assistants CLI pour canvas/caméra/écran/système"
read_when:
  - Appairage de nœuds iOS/Android à un gateway
  - Utilisation du canvas/caméra du nœud pour le contexte de l'agent
  - Ajout de nouvelles commandes de nœud ou assistants CLI
title: "Nœuds"
sidebarTitle: "Nœuds"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/index.md
  workflow: manual
---

# Nœuds

Un **nœud** est un appareil compagnon (macOS/iOS/Android/sans interface) qui se connecte au **WebSocket** du Gateway (même port que les opérateurs) avec `rôle: "node"` et expose une surface de commande (ex. `canvas.*`, `camera.*`, `system.*`) via `node.invoke`. Détails du protocole : [Protocole du Gateway](/gateway/protocol).

Transport historique : [Protocole Bridge](/gateway/bridge-protocol) (TCP JSONL ; déprécié/supprimé pour les nœuds actuels).

macOS peut également fonctionner en **mode nœud** : l'application de la barre de menus se connecte au serveur WS du Gateway et expose ses commandes canvas/caméra locales comme un nœud (ainsi `openclaw nodes …` fonctionne contre ce Mac).

Notes :

- Les nœuds sont des **périphériques**, pas des gateways. Ils n'exécutent pas le service gateway.
- Les messages Telegram/WhatsApp/etc. arrivent sur le **gateway**, pas sur les nœuds.
- Guide de dépannage : [/nodes/troubleshooting](/nodes/troubleshooting)

## Appairage + statut

**Les nœuds WS utilisent l'appairage d'appareil.** Les nœuds présentent une identité d'appareil lors du `connect` ; le Gateway
crée une demande d'appairage d'appareil pour `rôle: node`. Approuvez via le CLI d'appareils (ou l'interface).

CLI rapide :

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Notes :

- `nodes status` marque un nœud comme **appairé** lorsque son rôle d'appairage d'appareil inclut `node`.
- `node.pair.*` (CLI : `openclaw nodes pending/approve/reject`) est un magasin d'appairage de nœuds distinct appartenant au gateway ; il ne **contrôle pas** la poignée de main WS `connect`.

## Hôte de nœud distant (system.run)

Utilisez un **hôte de nœud** lorsque votre Gateway fonctionne sur une machine et que vous souhaitez que les commandes
s'exécutent sur une autre. Le modèle communique toujours avec le **gateway** ; le gateway
transfère les appels `exec` vers l'**hôte de nœud** lorsque `host=node` est sélectionné.

### Où s'exécute quoi

- **Hôte du Gateway** : reçoit les messages, exécute le modèle, route les appels d'outils.
- **Hôte du nœud** : exécute `system.run`/`system.which` sur la machine du nœud.
- **Approbations** : appliquées sur l'hôte du nœud via `~/.openclaw/exec-approvals.json`.

### Démarrer un hôte de nœud (premier plan)

Sur la machine du nœud :

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### Gateway distant via tunnel SSH (liaison loopback)

Si le Gateway est lié à loopback (`gateway.bind=loopback`, par défaut en mode local),
les hôtes de nœud distants ne peuvent pas se connecter directement. Créez un tunnel SSH et pointez
l'hôte du nœud vers l'extrémité locale du tunnel.

Exemple (hôte de nœud -> hôte du gateway) :

```bash
# Terminal A (garder en cours d'exécution) : transférer le port local 18790 -> gateway 127.0.0.1:18789
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Terminal B : exporter le jeton du gateway et se connecter via le tunnel
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

Notes :

- Le jeton est `gateway.auth.token` de la configuration du gateway (`~/.openclaw/openclaw.json` sur l'hôte du gateway).
- `openclaw node run` lit `OPENCLAW_GATEWAY_TOKEN` pour l'authentification.

### Démarrer un hôte de nœud (service)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### Appairer + nommer

Sur l'hôte du gateway :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes list
```

Options de nommage :

- `--display-name` sur `openclaw node run` / `openclaw node install` (persiste dans `~/.openclaw/node.json` sur le nœud).
- `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (surcharge côté gateway).

### Ajouter les commandes à la liste blanche

Les approbations exec sont **par hôte de nœud**. Ajoutez des entrées à la liste blanche depuis le gateway :

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Les approbations se trouvent sur l'hôte du nœud à `~/.openclaw/exec-approvals.json`.

### Diriger exec vers le nœud

Configurez les valeurs par défaut (configuration du gateway) :

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

Ou par session :

```
/exec host=node security=allowlist node=<id-or-name>
```

Une fois défini, tout appel `exec` avec `host=node` s'exécute sur l'hôte du nœud (soumis à la
liste blanche/approbations du nœud).

Références :

- [CLI hôte de nœud](/cli/node)
- [Outil exec](/tools/exec)
- [Approbations exec](/tools/exec-approvals)

## Invocation de commandes

Bas niveau (RPC brut) :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Des assistants de plus haut niveau existent pour les flux de travail courants « donner à l'agent une pièce jointe MEDIA ».

## Captures d'écran (instantanés canvas)

Si le nœud affiche le Canvas (WebView), `canvas.snapshot` retourne `{ format, base64 }`.

Assistant CLI (écrit dans un fichier temporaire et affiche `MEDIA:<path>`) :

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### Contrôles du canvas

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notes :

- `canvas présent` accepte des URL ou des chemins de fichiers locaux (`--target`), plus des options `--x/--y/--width/--height` pour le positionnement.
- `canvas eval` accepte du JS en ligne (`--js`) ou un argument positionnel.

### A2UI (Canvas)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Notes :

- Seul A2UI v0.8 JSONL est supporté (v0.9/createSurface est rejeté).

## Photos + vidéos (caméra du nœud)

Photos (`jpg`) :

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # par défaut : les deux orientations (2 lignes MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Clips vidéo (`mp4`) :

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notes :

- Le nœud doit être au **premier plan** pour `canvas.*` et `camera.*` (les appels en arrière-plan retournent `NODE_BACKGROUND_UNAVAILABLE`).
- La durée du clip est limitée (actuellement `<= 60s`) pour éviter les charges utiles base64 surdimensionnées.
- Android demandera les permissions `CAMERA`/`RECORD_AUDIO` lorsque possible ; les permissions refusées échouent avec `*_PERMISSION_REQUIRED`.

## Enregistrements d'écran (nœuds)

Les nœuds exposent `screen.record` (mp4). Exemple :

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notes :

- `screen.record` nécessite que l'application du nœud soit au premier plan.
- Android affichera l'invite système de capture d'écran avant l'enregistrement.
- Les enregistrements d'écran sont limités à `<= 60s`.
- `--no-audio` désactive la capture du microphone (supporté sur iOS/Android ; macOS utilise l'audio de capture système).
- Utilisez `--screen <index>` pour sélectionner un écran lorsque plusieurs écrans sont disponibles.

## Localisation (nœuds)

Les nœuds exposent `location.get` lorsque la localisation est activée dans les paramètres.

Assistant CLI :

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Notes :

- La localisation est **désactivée par défaut**.
- « Toujours » nécessite une permission système ; la récupération en arrière-plan est au mieux.
- La réponse inclut lat/lon, précision (mètres) et horodatage.

## SMS (nœuds Android)

Les nœuds Android peuvent exposer `sms.send` lorsque l'utilisateur accorde la permission **SMS** et que l'appareil supporte la téléphonie.

Invocation bas niveau :

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notes :

- L'invite de permission doit être acceptée sur l'appareil Android avant que la capacité soit annoncée.
- Les appareils Wi-Fi uniquement sans téléphonie n'annonceront pas `sms.send`.

## Commandes système (hôte de nœud / nœud Mac)

Le nœud macOS expose `system.run`, `system.notify` et `system.execApprovals.get/set`.
L'hôte de nœud sans interface expose `system.run`, `system.which` et `system.execApprovals.get/set`.

Exemples :

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notes :

- `system.run` retourne stdout/stderr/code de sortie dans la charge utile.
- `system.notify` respecte l'état de la permission de notification sur l'application macOS.
- `system.run` supporte `--cwd`, `--env KEY=VAL`, `--command-timeout` et `--needs-screen-recording`.
- Pour les wrappers shell (`bash|sh|zsh ... -c/-lc`), les valeurs `--env` de portée requête sont réduites à une liste blanche explicite (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
- Pour les décisions « toujours autoriser » en mode liste blanche, les wrappers de dispatch connus (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) conservent les chemins d'exécutables internes plutôt que les chemins de wrappers. Si le déroulement n'est pas sûr, aucune entrée de liste blanche n'est conservée automatiquement.
- Sur les hôtes de nœud Windows en mode liste blanche, les exécutions de wrapper shell via `cmd.exe /c` nécessitent une approbation (l'entrée de liste blanche seule n'autorisé pas automatiquement la forme wrapper).
- `system.notify` supporte `--priority <passive|activé|timeSensitive>` et `--delivery <system|overlay|auto>`.
- Les hôtes de nœud ignorent les surcharges de `PATH` et suppriment les clés de démarrage/shell dangereuses (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`). Si vous avez besoin d'entrées PATH supplémentaires, configurez l'environnement du service hôte de nœud (ou installez les outils dans des emplacements standard) au lieu de passer `PATH` via `--env`.
- Sur le nœud macOS en mode nœud, `system.run` est contrôlé par les approbations exec dans l'application macOS (Paramètres → Approbations exec).
  Demander/liste blanche/complet se comportent comme l'hôte de nœud sans interface ; les invites refusées retournent `SYSTEM_RUN_DENIED`.
- Sur l'hôte de nœud sans interface, `system.run` est contrôlé par les approbations exec (`~/.openclaw/exec-approvals.json`).

## Liaison exec du nœud

Lorsque plusieurs nœuds sont disponibles, vous pouvez lier exec à un nœud spécifique.
Cela définit le nœud par défaut pour `exec host=node` (et peut être surchargé par agent).

Valeur par défaut globale :

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Surcharge par agent :

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Supprimer pour autoriser n'importe quel nœud :

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## Carte des permissions

Les nœuds peuvent inclure une carte `permissions` dans `node.list` / `node.describe`, indexée par nom de permission (ex. `screenRecording`, `accessibility`) avec des valeurs booléennes (`true` = accordée).

## Hôte de nœud sans interface (multiplateforme)

OpenClaw peut exécuter un **hôte de nœud sans interface** (pas d'interface graphique) qui se connecte au WebSocket du Gateway
et expose `system.run` / `system.which`. C'est utile sous Linux/Windows
ou pour exécuter un nœud minimal à côté d'un serveur.

Démarrer :

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notes :

- L'appairage est toujours requis (le Gateway affichera une invite d'approbation de nœud).
- L'hôte de nœud stocke son identifiant de nœud, jeton, nom d'affichage et informations de connexion au gateway dans `~/.openclaw/node.json`.
- Les approbations exec sont appliquées localement via `~/.openclaw/exec-approvals.json`
  (voir [Approbations exec](/tools/exec-approvals)).
- Sur macOS, l'hôte de nœud sans interface exécute `system.run` localement par défaut. Définissez
  `OPENCLAW_NODE_EXEC_HOST=app` pour router `system.run` via l'hôte exec de l'application compagnon ; ajoutez
  `OPENCLAW_NODE_EXEC_FALLBACK=0` pour exiger l'hôte de l'application et échouer en mode fermé s'il n'est pas disponible.
- Ajoutez `--tls` / `--tls-fingerprint` lorsque le WS du Gateway utilisé TLS.

## Mode nœud Mac

- L'application de la barre de menus macOS se connecte au serveur WS du Gateway en tant que nœud (ainsi `openclaw nodes …` fonctionne contre ce Mac).
- En mode distant, l'application ouvre un tunnel SSH pour le port du Gateway et se connecte à `localhost`.
