---
summary: "Prise en charge de Signal via signal-cli (JSON-RPC + SSE), chemins de configuration et modèle de numéro"
read_when:
  - Configuration de la prise en charge de Signal
  - Debogage de l'envoi/reception Signal
title: "Signal"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/signal.md
  workflow: manual
---

# Signal (signal-cli)

Statut : intégration CLI externe. Le Gateway communique avec `signal-cli` via HTTP JSON-RPC + SSE.

## Prerequis

- OpenClaw installe sur votre serveur (flux Linux ci-dessous teste sur Ubuntu 24).
- `signal-cli` disponible sur l'hôte ou le gateway s'exécute.
- Un numéro de téléphone pouvant recevoir un SMS de vérification (pour le chemin d'enregistrement par SMS).
- Accès navigateur pour le captcha Signal (`signalcaptchas.org`) lors de l'enregistrement.

## Configuration rapide (debutant)

1. Utilisez un **numéro Signal séparé** pour le bot (recommandé).
2. Installez `signal-cli` (Java requis si vous utilisez la version JVM).
3. Choisissez un chemin de configuration :
   - **Chemin A (lien QR) :** `signal-cli link -n "OpenClaw"` et scannez avec Signal.
   - **Chemin B (enregistrement SMS) :** enregistrez un numéro dédié avec vérification captcha + SMS.
4. Configurez OpenClaw et redemarrez le gateway.
5. Envoyez un premier message privé et approuvez l'appairage (`openclaw pairing approve signal <CODE>`).

Configuration minimale :

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Référence des champs :

| Champ       | Description                                         |
| ----------- | --------------------------------------------------- |
| `account`   | Numéro de téléphone du bot au format E.164 (`+15551234567`) |
| `cliPath`   | Chemin vers `signal-cli` (`signal-cli` si dans le `PATH`) |
| `dmPolicy`  | Politique d'accès aux messages privés (`pairing` recommandé) |
| `allowFrom` | Numéros de téléphone ou valeurs `uuid:<id>` autorisés pour les messages privés |

## Description

- Canal Signal via `signal-cli` (pas de libsignal embarque).
- Routage déterministe : les réponses reviennent toujours vers Signal.
- Les messages privés partagent la session principale de l'agent ; les groupes sont isoles (`agent:<agentId>:signal:group:<groupId>`).

## Écritures de configuration

Par défaut, Signal est autorisé a écrire des mises a jour de configuration declenchees par `/config set|unset` (nécessite `commands.config: true`).

Désactiver avec :

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Le modèle de numéro (important)

- Le gateway se connecte a un **appareil Signal** (le compte `signal-cli`).
- Si vous exécutez le bot sur **votre compte Signal personnel**, il ignorera vos propres messages (protection anti-boucle).
- Pour « j'envoie un message au bot et il répond », utilisez un **numéro de bot séparé**.

## Chemin de configuration A : lier un compte Signal existant (QR)

1. Installez `signal-cli` (version JVM ou native).
2. Liez un compte de bot :
   - `signal-cli link -n "OpenClaw"` puis scannez le QR dans Signal.
3. Configurez Signal et démarrez le gateway.

Exemple :

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Support multi-comptes : utilisez `channels.signal.accounts` avec une configuration par compte et un `name` optionnel. Voir [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modèle partage.

## Chemin de configuration B : enregistrer un numéro de bot dédié (SMS, Linux)

Utilisez ceci lorsque vous souhaitez un numéro de bot dédié au lieu de lier un compte d'application Signal existant.

1. Obtenez un numéro pouvant recevoir des SMS (ou vérification vocale pour les lignes fixes).
   - Utilisez un numéro de bot dédié pour eviter les conflits de compte/session.
2. Installez `signal-cli` sur l'hôte du gateway :

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Si vous utilisez la version JVM (`signal-cli-${VERSION}.tar.gz`), installez d'abord JRE 25+.
Gardez `signal-cli` a jour ; les notes en amont indiquent que les anciennes versions peuvent cesser de fonctionner lorsque les API du serveur Signal changent.

3. Enregistrez et verifiez le numéro :

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Si un captcha est requis :

1. Ouvrez `https://signalcaptchas.org/registration/generate.html`.
2. Completez le captcha, copiez le lien cible `signalcaptcha://...` depuis « Open Signal ».
3. Exécutez depuis la même adresse IP externe que la session du navigateur lorsque c'est possible.
4. Exécutez l'enregistrement a nouveau immédiatement (les jetons captcha expirent rapidement) :

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. Configurez OpenClaw, redemarrez le gateway, verifiez le canal :

```bash
# Si vous executez le gateway en tant que service systemd utilisateur :
systemctl --user restart openclaw-gateway

# Puis verifiez :
openclaw doctor
openclaw channels status --probe
```

5. Appairez votre expéditeur de messages privés :
   - Envoyez n'importe quel message au numéro du bot.
   - Approuvez le code sur le serveur : `openclaw pairing approve signal <PAIRING_CODE>`.
   - Sauvegardez le numéro du bot comme contact sur votre téléphone pour eviter « Contact inconnu ».

Important : enregistrer un compte de numéro de téléphone avec `signal-cli` peut desauthentifier la session principale de l'application Signal pour ce numéro. Préférez un numéro de bot dédié, ou utilisez le mode lien QR si vous devez garder la configuration de votre application téléphone existante.

Références en amont :

- README de `signal-cli` : `https://github.com/AsamK/signal-cli`
- Flux captcha : `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Flux de liaison : `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Mode daemon externe (httpUrl)

Si vous souhaitez gérer `signal-cli` vous-même (demarrages a froid JVM lents, initialisation de conteneur ou CPU partages), exécutez le daemon séparément et pointez OpenClaw dessus :

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Cela saute le lancement automatique et l'attente de démarrage a l'interieur d'OpenClaw. Pour les demarrages lents avec le lancement automatique, définissez `channels.signal.startupTimeoutMs`.

## Contrôle d'accès (messages privés + groupes)

Messages privés :

- Par défaut : `channels.signal.dmPolicy = "pairing"`.
- Les expéditeurs inconnus reçoivent un code d'appairage ; les messages sont ignorés jusqu'a approbation (les codes expirent après 1 heure).
- Approuver via :
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- L'appairage est l'echange de jetons par défaut pour les messages privés Signal. Details : [Appairage](/channels/pairing)
- Les expéditeurs UUID uniquement (depuis `sourceUuid`) sont stockes sous la forme `uuid:<id>` dans `channels.signal.allowFrom`.

Groupes :

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` contrôle qui peut déclencher dans les groupes lorsque `allowlist` est défini.
- Note d'exécution : si `channels.signal` est complètement absent, l'exécution se rabat sur `groupPolicy="allowlist"` pour les vérifications de groupe (même si `channels.defaults.groupPolicy` est défini).

## Fonctionnement (comportement)

- `signal-cli` s'exécute en tant que daemon ; le gateway lit les événements via SSE.
- Les messages entrants sont normalisés dans l'enveloppe de canal partagee.
- Les réponses sont toujours acheminees vers le même numéro ou groupe.

## Médias + limités

- Le texte sortant est decoupe a `channels.signal.textChunkLimit` (par défaut 4000).
- Découpage optionnel par lignes : définissez `channels.signal.chunkMode="newline"` pour decouper sur les lignes vides (limités de paragraphe) avant le découpage par longueur.
- Pieces jointes prises en charge (base64 récupéré depuis `signal-cli`).
- Limité média par défaut : `channels.signal.mediaMaxMb` (par défaut 8).
- Utilisez `channels.signal.ignoreAttachments` pour ignorer le téléchargement des médias.
- L'historique de contexte du groupe utilisé `channels.signal.historyLimit` (ou `channels.signal.accounts.*.historyLimit`), avec repli sur `messages.groupChat.historyLimit`. Définissez `0` pour désactiver (par défaut 50).

## Saisie + accuses de reception

- **Indicateurs de saisie** : OpenClaw envoie des signaux de saisie via `signal-cli sendTyping` et les rafraichit pendant qu'une réponse est en cours.
- **Accuses de reception** : lorsque `channels.signal.sendReadReceipts` est true, OpenClaw transmet les accuses de reception pour les messages privés autorisés.
- signal-cli n'expose pas les accuses de reception pour les groupes.

## Réactions (outil de message)

- Utilisez `message action=react` avec `channel=signal`.
- Cibles : E.164 ou UUID de l'expéditeur (utilisez `uuid:<id>` depuis la sortie d'appairage ; l'UUID nu fonctionne aussi).
- `messageId` est l'horodatage Signal du message auquel vous reagissez.
- Les reactions de groupe nécessitent `targetAuthor` ou `targetAuthorUuid`.

Exemples :

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Configuration :

- `channels.signal.actions.reactions` : activer/désactiver les actions de reaction (par défaut true).
- `channels.signal.reactionLevel` : `off | ack | minimal | extensive`.
  - `off`/`ack` désactivé les reactions de l'agent (l'outil de message `react` donnera une erreur).
  - `minimal`/`extensive` activé les reactions de l'agent et définit le niveau de guidage.
- Remplacements par compte : `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Cibles de livraison (CLI/cron)

- Messages privés : `signal:+15551234567` (ou E.164 simple).
- Messages privés UUID : `uuid:<id>` (ou UUID nu).
- Groupes : `signal:group:<groupId>`.
- Noms d'utilisateur : `username:<name>` (si pris en charge par votre compte Signal).

## Dépannage

Exécutez cette echelle en premier :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis confirmez l'état d'appairage des messages privés si nécessaire :

```bash
openclaw pairing list signal
```

Pannes courantes :

- Daemon accessible mais pas de réponses : verifiez les paramètres du compte/daemon (`httpUrl`, `account`) et le mode de reception.
- Messages privés ignorés : l'expéditeur est en attente d'approbation d'appairage.
- Messages de groupe ignorés : le filtrage des expéditeurs/mentions de groupe bloque la livraison.
- Erreurs de validation de configuration après les modifications : exécutez `openclaw doctor --fix`.
- Signal absent des diagnostics : confirmez `channels.signal.enabled: true`.

Vérifications supplementaires :

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Pour le flux de triage : [/channels/troubleshooting](/channels/troubleshooting).

## Notes de sécurité

- `signal-cli` stocke les clés de compte localement (généralement `~/.local/share/signal-cli/data/`).
- Sauvegardez l'état du compte Signal avant la migration ou la reconstruction du serveur.
- Gardez `channels.signal.dmPolicy: "pairing"` sauf si vous souhaitez explicitement un accès plus large aux messages privés.
- La vérification SMS n'est nécessaire que pour les flux d'enregistrement ou de récupération, mais perdre le contrôle du numéro/compte peut compliquer la re-inscription.

## Référence de configuration (Signal)

Configuration complète : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.signal.enabled` : activer/désactiver le démarrage du canal.
- `channels.signal.account` : E.164 pour le compte du bot.
- `channels.signal.cliPath` : chemin vers `signal-cli`.
- `channels.signal.httpUrl` : URL complète du daemon (remplace host/port).
- `channels.signal.httpHost`, `channels.signal.httpPort` : liaison du daemon (par défaut 127.0.0.1:8080).
- `channels.signal.autoStart` : lancement automatique du daemon (par défaut true si `httpUrl` non défini).
- `channels.signal.startupTimeoutMs` : timeout d'attente de démarrage en ms (limité 120000).
- `channels.signal.receiveMode` : `on-start | manual`.
- `channels.signal.ignoreAttachments` : ignorer les telechargements de pieces jointes.
- `channels.signal.ignoreStories` : ignorer les stories du daemon.
- `channels.signal.sendReadReceipts` : transmettre les accuses de reception.
- `channels.signal.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
- `channels.signal.allowFrom` : liste autorisée de messages privés (E.164 ou `uuid:<id>`). `open` nécessite `"*"`. Signal n'a pas de noms d'utilisateur ; utilisez les identifiants téléphone/UUID.
- `channels.signal.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
- `channels.signal.groupAllowFrom` : liste autorisée d'expéditeurs de groupe.
- `channels.signal.historyLimit` : nombre maximum de messages de groupe a inclure comme contexte (0 désactive).
- `channels.signal.dmHistoryLimit` : limité d'historique des messages privés en tours utilisateur. Remplacements par utilisateur : `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit` : taille de découpage sortant (caractères).
- `channels.signal.chunkMode` : `length` (par défaut) ou `newline` pour decouper sur les lignes vides (limités de paragraphe) avant le découpage par longueur.
- `channels.signal.mediaMaxMb` : limité de médias entrants/sortants (Mo).

Options globales associées :

- `agents.list[].groupChat.mentionPatterns` (Signal ne prend pas en charge les mentions natives).
- `messages.groupChat.mentionPatterns` (repli global).
- `messages.responsePrefix`.
