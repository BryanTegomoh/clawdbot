---
summary: "Interface de contrôle dans le navigateur pour le Gateway (chat, nœuds, configuration)"
read_when:
  - Vous souhaitez opérer le Gateway depuis un navigateur
  - Vous souhaitez un accès Tailnet sans tunnels SSH
title: "Interface de contrôle"
sidebarTitle: "Interface de contrôle"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/web/control-ui.md
  workflow: manual
---

# Interface de contrôle (navigateur)

L'Interface de contrôle est une petite application monopage **Vite + Lit** servie par le Gateway :

- par défaut : `http://<host>:18789/`
- préfixe optionnel : définir `gateway.controlUi.basePath` (ex. `/openclaw`)

Elle communique **directement avec le WebSocket du Gateway** sur le même port.

## Ouverture rapide (local)

Si le Gateway fonctionne sur le même ordinateur, ouvrez :

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (ou [http://localhost:18789/](http://localhost:18789/))

Si la page ne se charge pas, démarrez d'abord le Gateway : `openclaw gateway`.

L'authentification est fournie lors de la poignée de main WebSocket via :

- `connect.params.auth.token`
- `connect.params.auth.password`
  Le panneau des paramètres du Tableau de bord vous permet de stocker un jeton ; les mots de passe ne sont pas conservés.
  L'assistant de configuration initiale génère un jeton de gateway par défaut, collez-le ici lors de la première connexion.

## Appairage d'appareil (première connexion)

Lorsque vous vous connectez à l'Interface de contrôle depuis un nouveau navigateur ou appareil, le Gateway
exige une **approbation d'appairage unique**, même si vous êtes sur le même Tailnet
avec `gateway.auth.allowTailscale: true`. C'est une mesure de sécurité pour empêcher
les accès non autorisés.

**Ce que vous verrez :** "disconnected (1008): pairing required"

**Pour approuver l'appareil :**

```bash
# Lister les demandes en attente
openclaw devices list

# Approuver par ID de demande
openclaw devices approve <requestId>
```

Une fois approuvé, l'appareil est mémorisé et ne nécessitera pas de nouvelle approbation, sauf si
vous le révoquez avec `openclaw devices revoke --device <id> --rôle <rôle>`. Voir
[CLI Appareils](/cli/devices) pour la rotation et la révocation des jetons.

**Notes :**

- Les connexions locales (`127.0.0.1`) sont automatiquement approuvées.
- Les connexions distantes (LAN, Tailnet, etc.) nécessitent une approbation explicite.
- Chaque profil de navigateur génère un identifiant d'appareil unique ; changer de navigateur ou
  effacer les données du navigateur nécessitera un nouvel appairage.

## Fonctionnalités disponibles (aujourd'hui)

- Chat avec le modèle via Gateway WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
- Diffusion des appels d'outils et cartes de sortie d'outils en direct dans le Chat (événements agent)
- Canaux : WhatsApp/Telegram/Discord/Slack + canaux de plugins (Mattermost, etc.) statut + connexion QR + configuration par canal (`channels.status`, `web.login.*`, `config.patch`)
- Instances : liste de présence + rafraîchissement (`system-presence`)
- Sessions : liste + surcharges pensée/verbeux par session (`sessions.list`, `sessions.patch`)
- Tâches cron : lister/ajouter/modifier/exécuter/activer/désactiver + historique d'exécution (`cron.*`)
- Skills : statut, activer/désactiver, installer, mise à jour des clés API (`skills.*`)
- Nœuds : liste + capacités (`node.list`)
- Approbations exec : modifier les listes blanches du gateway ou des nœuds + politique de demande pour `exec host=gateway/node` (`exec.approvals.*`)
- Configuration : voir/modifier `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
- Configuration : appliquer + redémarrer avec validation (`config.apply`) et réveiller la dernière session active
- Les écritures de configuration incluent une protection de hash de base pour éviter l'écrasement d'éditions concurrentes
- Schéma de configuration + rendu de formulaire (`config.schema`, incluant les schémas de plugins et de canaux) ; l'éditeur JSON brut reste disponible
- Débogage : instantanés statut/santé/modèles + journal d'événements + appels RPC manuels (`status`, `health`, `models.list`)
- Journaux : suivi en direct des journaux de fichiers du gateway avec filtré/export (`logs.tail`)
- Mise à jour : exécuter une mise à jour package/git + redémarrage (`update.run`) avec un rapport de redémarrage

Notes du panneau de tâches cron :

- Pour les tâches isolées, la livraison est par défaut en résumé d'annonce. Vous pouvez passer à « aucun » si vous souhaitez des exécutions internes uniquement.
- Les champs canal/cible apparaissent lorsque « annoncer » est sélectionné.
- Le mode webhook utilisé `delivery.mode = "webhook"` avec `delivery.to` défini sur une URL webhook HTTP(S) valide.
- Pour les tâches de session principale, les modes de livraison webhook et « aucun » sont disponibles.
- Les contrôles d'édition avancée incluent supprimer après exécution, effacer la surcharge d'agent, options exact/décalage cron, surcharges de modèle/pensée de l'agent, et bascules de livraison au mieux.
- La validation du formulaire est en ligne avec des erreurs au niveau des champs ; les valeurs invalides désactivent le bouton de sauvegarde jusqu'à correction.
- Définissez `cron.webhookToken` pour envoyer un jeton bearer dédié ; s'il est omis, le webhook est envoyé sans en-tête d'authentification.
- Repli déprécié : les anciennes tâches stockées avec `notify: true` peuvent encore utiliser `cron.webhook` jusqu'à leur migration.

## Comportement du chat

- `chat.send` est **non bloquant** : il accuse réception immédiatement avec `{ runId, status: "started" }` et la réponse est diffusée via les événements `chat`.
- Renvoyer avec la même `idempotencyKey` retourne `{ status: "in_flight" }` pendant l'exécution, et `{ status: "ok" }` après achèvement.
- Les réponses de `chat.history` ont une taille limitée pour la sécurité de l'interface. Lorsque les entrées de transcription sont trop volumineuses, le Gateway peut tronquer les champs texte longs, omettre les blocs de métadonnées lourds, et remplacer les messages surdimensionnés par un espace réservé (`[chat.history omitted: message too large]`).
- `chat.inject` ajouté une note d'assistant à la transcription de session et diffuse un événement `chat` pour les mises à jour de l'interface uniquement (pas d'exécution d'agent, pas de livraison au canal).
- Arrêt :
  - Cliquez sur **Arrêter** (appelle `chat.abort`)
  - Tapez `/stop` (ou des phrases d'arrêt autonomes comme `stop`, `stop action`, `stop run`, `stop openclaw`, `please stop`) pour annuler hors bande
  - `chat.abort` supporte `{ sessionKey }` (sans `runId`) pour annuler toutes les exécutions activés de cette session
- Rétention partielle d'annulation :
  - Lorsqu'une exécution est annulée, le texte partiel de l'assistant peut encore être affiché dans l'interface
  - Le Gateway conserve le texte partiel annulé de l'assistant dans l'historique de transcription lorsqu'une sortie tamponnée existe
  - Les entrées conservées incluent des métadonnées d'annulation pour que les consommateurs de transcription puissent distinguer les partiels annulés de la sortie normale

## Accès Tailnet (recommandé)

### Tailscale Serve intégré (préféré)

Gardez le Gateway en boucle locale et laissez Tailscale Serve le rediriger via un proxy avec HTTPS :

```bash
openclaw gateway --tailscale serve
```

Ouvrir :

- `https://<magicdns>/` (ou votre `gateway.controlUi.basePath` configuré)

Par défaut, les requêtes Serve de l'Interface de contrôle/WebSocket peuvent s'authentifier via les en-têtes d'identité Tailscale
(`tailscale-user-login`) lorsque `gateway.auth.allowTailscale` est `true`. OpenClaw
vérifie l'identité en résolvant l'adresse `x-forwarded-for` avec
`tailscale whois` et en la comparant à l'en-tête, et n'accepte ces en-têtes que lorsque la
requête arrive en loopback avec les en-têtes `x-forwarded-*` de Tailscale. Définissez
`gateway.auth.allowTailscale: false` (ou forcez `gateway.auth.mode: "password"`)
si vous souhaitez exiger un jeton/mot de passe même pour le trafic Serve.
L'authentification Serve sans jeton suppose que l'hôte du gateway est de confiance. Si du
code local non fiable peut s'exécuter sur cet hôte, exigez l'authentification par jeton/mot de passe.

### Liaison tailnet + jeton

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Puis ouvrez :

- `http://<tailscale-ip>:18789/` (ou votre `gateway.controlUi.basePath` configuré)

Collez le jeton dans les paramètres de l'interface (envoyé comme `connect.params.auth.token`).

## HTTP non sécurisé

Si vous ouvrez le Tableau de bord en HTTP simple (`http://<lan-ip>` ou `http://<tailscale-ip>`),
le navigateur fonctionne dans un **contexte non sécurisé** et bloque WebCrypto. Par défaut,
OpenClaw **bloque** les connexions de l'Interface de contrôle sans identité d'appareil.

**Correction recommandée :** utilisez HTTPS (Tailscale Serve) ou ouvrez l'interface localement :

- `https://<magicdns>/` (Serve)
- `http://127.0.0.1:18789/` (sur l'hôte du gateway)

**Comportement de la bascule d'authentification non sécurisée :**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth` ne contourne pas l'identité d'appareil ni les vérifications d'appairage de l'Interface de contrôle.

**Accès d'urgence uniquement :**

```json5
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth` désactive les vérifications d'identité d'appareil de l'Interface de contrôle et constitue une
dégradation sévère de la sécurité. Revenez rapidement en arrière après une utilisation d'urgence.

Voir [Tailscale](/gateway/tailscale) pour le guide de configuration HTTPS.

## Construction de l'interface

Le Gateway sert des fichiers statiques depuis `dist/control-ui`. Construisez-les avec :

```bash
pnpm ui:build # installe automatiquement les dépendances UI au premier lancement
```

Base absolue optionnelle (lorsque vous souhaitez des URL de ressources fixes) :

```bash
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Pour le développement local (serveur de développement séparé) :

```bash
pnpm ui:dev # installe automatiquement les dépendances UI au premier lancement
```

Puis pointez l'interface vers l'URL WS de votre Gateway (ex. `ws://127.0.0.1:18789`).

## Débogage/test : serveur de développement + Gateway distant

L'Interface de contrôle est composée de fichiers statiques ; la cible WebSocket est configurable et peut être
différente de l'origine HTTP. C'est pratique lorsque vous souhaitez le serveur de développement Vite
localement mais que le Gateway fonctionne ailleurs.

1. Démarrez le serveur de développement de l'interface : `pnpm ui:dev`
2. Ouvrez une URL comme :

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Authentification unique optionnelle (si nécessaire) :

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Notes :

- `gatewayUrl` est stocké dans localStorage après le chargement et supprime de l'URL.
- `token` est stocké dans localStorage ; `password` est conservé en mémoire uniquement.
- Lorsque `gatewayUrl` est défini, l'interface ne se rabat pas sur les identifiants de configuration ou d'environnement.
  Fournissez `token` (ou `password`) explicitement. L'absence d'identifiants explicites est une erreur.
- Utilisez `wss://` lorsque le Gateway est derrière TLS (Tailscale Serve, proxy HTTPS, etc.).
- `gatewayUrl` n'est accepté que dans une fenêtre de niveau supérieur (pas intégrée) pour prévenir le détournement de clic.
- Les déploiements de l'Interface de contrôle non-loopback doivent définir `gateway.controlUi.allowedOrigins`
  explicitement (origines complètes). Cela inclut les configurations de développement à distance.
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` activé le
  mode de repli sur l'en-tête Host pour l'origine, mais c'est un mode de sécurité dangereux.

Exemple :

```json5
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

Détails de la configuration d'accès distant : [Accès distant](/gateway/remote).
