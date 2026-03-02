---
summary: "Capture caméra (nœud iOS + application macOS) pour utilisation par l'agent : photos (jpg) et courts clips vidéo (mp4)"
read_when:
  - Ajout ou modification de la capture caméra sur les nœuds iOS ou macOS
  - Extension des flux de travail de fichiers temporaires MEDIA accessibles par l'agent
title: "Capture caméra"
sidebarTitle: "Capture caméra"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/camera.md
  workflow: manual
---

# Capture caméra (agent)

OpenClaw prend en charge la **capture caméra** pour les flux de travail d'agent :

- **Nœud iOS** (appairé via Gateway) : capturer une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio optionnel) via `node.invoke`.
- **Nœud Android** (appairé via Gateway) : capturer une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio optionnel) via `node.invoke`.
- **Application macOS** (nœud via Gateway) : capturer une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio optionnel) via `node.invoke`.

Tout accès caméra est contrôlé par des **paramètres gérés par l'utilisateur**.

## Nœud iOS

### Paramètre utilisateur (activé par défaut)

- Onglet Paramètres iOS → **Caméra** → **Autoriser la caméra** (`camera.enabled`)
  - Par défaut : **activé** (la clé manquante est traitée comme activée).
  - Lorsque désactivé : les commandes `camera.*` retournent `CAMERA_DISABLED`.

### Commandes (via Gateway `node.invoke`)

- `camera.list`
  - Charge utile de réponse :
    - `devices` : tableau de `{ id, name, position, deviceType }`

- `camera.snap`
  - Paramètres :
    - `facing` : `front|back` (par défaut : `front`)
    - `maxWidth` : nombre (optionnel ; par défaut `1600` sur le nœud iOS)
    - `quality` : `0..1` (optionnel ; par défaut `0.9`)
    - `format` : actuellement `jpg`
    - `delayMs` : nombre (optionnel ; par défaut `0`)
    - `deviceId` : chaîne (optionnel ; depuis `camera.list`)
  - Charge utile de réponse :
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Protection de charge utile : les photos sont recompressées pour maintenir la charge base64 sous 5 Mo.

- `camera.clip`
  - Paramètres :
    - `facing` : `front|back` (par défaut : `front`)
    - `durationMs` : nombre (par défaut `3000`, limité à un maximum de `60000`)
    - `includeAudio` : booléen (par défaut `true`)
    - `format` : actuellement `mp4`
    - `deviceId` : chaîne (optionnel ; depuis `camera.list`)
  - Charge utile de réponse :
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

### Exigence de premier plan

Comme pour `canvas.*`, le nœud iOS n'autorisé les commandes `camera.*` qu'au **premier plan**. Les invocations en arrière-plan retournent `NODE_BACKGROUND_UNAVAILABLE`.

### Assistant CLI (fichiers temporaires + MEDIA)

Le moyen le plus simple d'obtenir des pièces jointes est via l'assistant CLI, qui écrit les médias décodés dans un fichier temporaire et affiche `MEDIA:<path>`.

Exemples :

```bash
openclaw nodes camera snap --node <id>               # par défaut : avant + arrière (2 lignes MEDIA)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Notes :

- `nodes camera snap` capture par défaut les **deux** orientations pour donner les deux vues à l'agent.
- Les fichiers de sortie sont temporaires (dans le répertoire temporaire du système) sauf si vous créez votre propre wrapper.

## Nœud Android

### Paramètre utilisateur Android (activé par défaut)

- Feuille de paramètres Android → **Caméra** → **Autoriser la caméra** (`camera.enabled`)
  - Par défaut : **activé** (la clé manquante est traitée comme activée).
  - Lorsque désactivé : les commandes `camera.*` retournent `CAMERA_DISABLED`.

### Permissions

- Android nécessite des permissions d'exécution :
  - `CAMERA` pour `camera.snap` et `camera.clip`.
  - `RECORD_AUDIO` pour `camera.clip` lorsque `includeAudio=true`.

Si des permissions sont manquantes, l'application demandera lorsque possible ; si refusées, les requêtes `camera.*` échouent avec une
erreur `*_PERMISSION_REQUIRED`.

### Exigence de premier plan Android

Comme pour `canvas.*`, le nœud Android n'autorisé les commandes `camera.*` qu'au **premier plan**. Les invocations en arrière-plan retournent `NODE_BACKGROUND_UNAVAILABLE`.

### Protection de charge utile

Les photos sont recompressées pour maintenir la charge base64 sous 5 Mo.

## Application macOS

### Paramètre utilisateur (désactivé par défaut)

L'application compagnon macOS expose une case à cocher :

- **Paramètres → Général → Autoriser la caméra** (`openclaw.cameraEnabled`)
  - Par défaut : **désactivé**
  - Lorsque désactivé : les requêtes caméra retournent « Camera disabled by user ».

### Assistant CLI (invocation du nœud)

Utilisez le CLI principal `openclaw` pour invoquer les commandes caméra sur le nœud macOS.

Exemples :

```bash
openclaw nodes camera list --node <id>            # lister les identifiants de caméra
openclaw nodes camera snap --node <id>            # affiche MEDIA:<path>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # affiche MEDIA:<path>
openclaw nodes camera clip --node <id> --duration-ms 3000      # affiche MEDIA:<path> (ancien drapeau)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Notes :

- `openclaw nodes camera snap` utilisé par défaut `maxWidth=1600` sauf surcharge.
- Sur macOS, `camera.snap` attend `delayMs` (par défaut 2000ms) après la stabilisation du préchauffage/de l'exposition avant la capture.
- Les charges utiles de photos sont recompressées pour maintenir le base64 sous 5 Mo.

## Sécurité + limités pratiques

- L'accès à la caméra et au microphone déclenche les invites habituelles de permission du système (et nécessite des chaînes d'utilisation dans Info.plist).
- Les clips vidéo sont limités (actuellement `<= 60s`) pour éviter les charges utiles de nœud surdimensionnées (surcharge base64 + limités de message).

## Vidéo d'écran macOS (niveau OS)

Pour la vidéo d'_écran_ (pas de caméra), utilisez le compagnon macOS :

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # affiche MEDIA:<path>
```

Notes :

- Nécessite la permission **Enregistrement d'écran** macOS (TCC).
