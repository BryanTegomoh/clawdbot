---
summary: "Aperçu de l'appairage : approuver qui peut vous envoyer des messages privés et quels nœuds peuvent rejoindre"
read_when:
  - Configuration du contrôle d'accès aux messages privés
  - Appairage d'un nouveau nœud iOS/Android
  - Revue de la posture de sécurité d'OpenClaw
title: "Appairage"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/pairing.md
  workflow: manual
---

# Appairage

L'« appairage » est l'étape d'**approbation explicite par le propriétaire** d'OpenClaw.
Il est utilisé a deux endroits :

1. **Appairage des messages privés** (qui est autorisé a parler au bot)
2. **Appairage des nœuds** (quels appareils/nœuds sont autorisés a rejoindre le réseau du gateway)

Contexte de sécurité : [Sécurité](/gateway/security)

## 1) Appairage des messages privés (accès aux discussions entrantes)

Lorsqu'un canal est configuré avec la politique de messages privés `pairing`, les expéditeurs inconnus reçoivent un code court et leur message n'est **pas traite** tant que vous ne l'avez pas approuve.

Les politiques de messages privés par défaut sont documentees dans : [Sécurité](/gateway/security)

Codes d'appairage :

- 8 caractères, majuscules, sans caractères ambigus (`0O1I`).
- **Expirent après 1 heure**. Le bot n'envoie le message d'appairage que lorsqu'une nouvelle demande est créée (environ une fois par heure par expéditeur).
- Les demandes d'appairage en attente sont limitées a **3 par canal** par défaut ; les demandes supplementaires sont ignorées jusqu'a ce qu'une expire ou soit approuvee.

### Approuver un expéditeur

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canaux pris en charge : `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`, `feishu`.

### Ou l'état est stocke

Stocke sous `~/.openclaw/credentials/` :

- Demandes en attente : `<channel>-pairing.json`
- Magasin de liste autorisée approuvee :
  - Compte par défaut : `<channel>-allowFrom.json`
  - Compte non-défaut : `<channel>-<accountId>-allowFrom.json`

Comportement de portee par compte :

- Les comptes non-défaut lisent/ecrivent uniquement leur fichier de liste autorisée a portee limitée.
- Le compte par défaut utilisé le fichier de liste autorisée a portee canal (non-scope).

Traitez ces fichiers comme sensibles (ils controlent l'accès a votre assistant).

## 2) Appairage des nœuds (iOS/Android/macOS/nœuds headless)

Les nœuds se connectent au Gateway en tant qu'**appareils** avec `rôle: node`. Le Gateway
créé une demande d'appairage d'appareil qui doit être approuvee.

### Appairage via Telegram (recommandé pour iOS)

Si vous utilisez le plugin `device-pair`, vous pouvez effectuer le premier appairage d'appareil entièrement depuis Telegram :

1. Dans Telegram, envoyez a votre bot : `/pair`
2. Le bot répond avec deux messages : un message d'instructions et un **code de configuration** séparé (facile a copier/coller dans Telegram).
3. Sur votre téléphone, ouvrez l'application iOS OpenClaw -> Reglages -> Gateway.
4. Collez le code de configuration et connectez-vous.
5. De retour dans Telegram : `/pair approve`

Le code de configuration est un payload JSON encode en base64 qui contient :

- `url` : l'URL WebSocket du Gateway (`ws://...` ou `wss://...`)
- `token` : un jeton d'appairage a duree limitée

Traitez le code de configuration comme un mot de passe tant qu'il est valide.

### Approuver un nœud

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Stockage de l'état d'appairage des nœuds

Stocke sous `~/.openclaw/devices/` :

- `pending.json` (courte duree ; les demandes en attente expirent)
- `paired.json` (appareils appairés + jetons)

### Notes

- L'API historique `node.pair.*` (CLI : `openclaw nodes pending/approve`) est un
  magasin d'appairage distinct appartenant au gateway. Les nœuds WS nécessitent toujours l'appairage d'appareil.

## Documentation associée

- Modèle de sécurité + injection de prompt : [Sécurité](/gateway/security)
- Mise a jour en toute sécurité (exécuter doctor) : [Mise a jour](/install/updating)
- Configurations des canaux :
  - Telegram : [Telegram](/channels/telegram)
  - WhatsApp : [WhatsApp](/channels/whatsapp)
  - Signal : [Signal](/channels/signal)
  - BlueBubbles (iMessage) : [BlueBubbles](/channels/bluebubbles)
  - iMessage (ancien) : [iMessage](/channels/imessage)
  - Discord : [Discord](/channels/discord)
  - Slack : [Slack](/channels/slack)
