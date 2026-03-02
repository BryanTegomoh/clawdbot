---
title: Google Chat
description: Connecter OpenClaw à Google Chat via l'API Google Chat.
summary: "Statut de prise en charge de l'application Google Chat, fonctionnalités et configuration"
read_when:
  - Vous travaillez sur les fonctionnalités du canal Google Chat
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/googlechat.md
  workflow: manual
---

# Google Chat (API Chat)

Statut : prêt pour les Messages privés + espaces via les webhooks de l'API Google Chat (HTTP uniquement).

## Configuration rapide (débutant)

1. Créez un projet Google Cloud et activez l'**API Google Chat**.
   - Rendez-vous sur : [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - Activez l'API si elle n'est pas encore activée.
2. Créez un **Compte de service** :
   - Cliquez sur **Create Credentials** > **Service Account**.
   - Nommez-le comme vous voulez (par ex., `openclaw-chat`).
   - Laissez les permissions vides (cliquez sur **Continue**).
   - Laissez les accès des principaux vides (cliquez sur **Done**).
3. Créez et téléchargez la **clé JSON** :
   - Dans la liste des comptes de service, cliquez sur celui que vous venez de créer.
   - Allez dans l'onglet **Keys**.
   - Cliquez sur **Add Key** > **Create new key**.
   - Sélectionnez **JSON** et cliquez sur **Create**.
4. Enregistrez le fichier JSON téléchargé sur votre hôte Gateway (par ex., `~/.openclaw/googlechat-service-account.json`).
5. Créez une application Google Chat dans la [Configuration Google Cloud Console Chat](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) :
   - Remplissez les **informations de l'application** :
     - **App name** : (par ex. `OpenClaw`)
     - **Avatar URL** : (par ex. `https://openclaw.ai/logo.png`)
     - **Description** : (par ex. `Personal AI Assistant`)
   - Activez les **Interactive features**.
   - Sous **Functionality**, cochez **Join spaces and group conversations**.
   - Sous **Connection settings**, sélectionnez **HTTP endpoint URL**.
   - Sous **Triggers**, sélectionnez **Use a common HTTP endpoint URL for all triggers** et définissez-le sur l'URL publique de votre Gateway suivi de `/googlechat`.
     - _Astuce : Exécutez `openclaw status` pour trouver l'URL publique de votre Gateway._
   - Sous **Visibility**, cochez **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**.
   - Saisissez votre adresse e-mail (par ex. `user@example.com`) dans le champ texte.
   - Cliquez sur **Save** en bas.
6. **Activez le statut de l'application** :
   - Après avoir sauvegardé, **rechargez la page**.
   - Cherchez la section **App status** (généralement en haut ou en bas après la sauvegarde).
   - Changez le statut en **Live - available to users**.
   - Cliquez à nouveau sur **Save**.
7. Configurez OpenClaw avec le chemin du compte de service + l'audience du webhook :
   - Env : `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - Ou config : `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Définissez le type d'audience du webhook + la valeur (correspond à la configuration de votre application Chat).
9. Démarrez le Gateway. Google Chat enverra des POST à votre chemin de webhook.

## Ajouter à Google Chat

Une fois le Gateway en cours d'exécution et votre e-mail ajoute à la liste de visibilité :

1. Rendez-vous sur [Google Chat](https://chat.google.com/).
2. Cliquez sur l'icône **+** (plus) à côté de **Direct Messages**.
3. Dans la barre de recherche (où vous ajoutez habituellement des personnes), tapez le **nom de l'application** que vous avez configuré dans la console Google Cloud.
   - **Note** : Le bot _n'apparaîtra pas_ dans la liste de navigation « Marketplace » car c'est une application privée. Vous devez le rechercher par son nom.
4. Sélectionnez votre bot dans les résultats.
5. Cliquez sur **Add** ou **Chat** pour démarrer une conversation 1:1.
6. Envoyez « Hello » pour déclencher l'assistant !

## URL publique (webhook uniquement)

Les webhooks Google Chat nécessitent un point de terminaison HTTPS public. Pour la sécurité, **n'exposez que le chemin `/googlechat`** sur Internet. Gardez le tableau de bord OpenClaw et les autres points de terminaison sensibles sur votre réseau privé.

### Option A : Tailscale Funnel (recommandé)

Utilisez Tailscale Serve pour le tableau de bord privé et Funnel pour le chemin du webhook public. Cela garde `/` privé tout en n'exposant que `/googlechat`.

1. **Vérifiez l'adresse à laquelle votre Gateway est lié :**

   ```bash
   ss -tlnp | grep 18789
   ```

   Notez l'adresse IP (par ex., `127.0.0.1`, `0.0.0.0`, ou votre IP Tailscale comme `100.x.x.x`).

2. **Exposez le tableau de bord au tailnet uniquement (port 8443) :**

   ```bash
   # Si lié à localhost (127.0.0.1 ou 0.0.0.0) :
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Si lié à l'IP Tailscale uniquement (par ex., 100.106.161.80) :
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Exposez uniquement le chemin du webhook publiquement :**

   ```bash
   # Si lié à localhost (127.0.0.1 ou 0.0.0.0) :
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Si lié à l'IP Tailscale uniquement (par ex., 100.106.161.80) :
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Autorisez le nœud pour l'accès Funnel :**
   Si demandé, visitez l'URL d'autorisation affichée dans la sortie pour activer Funnel pour ce nœud dans votre politique tailnet.

5. **Vérifiez la configuration :**

   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Votre URL de webhook publique sera :
`https://<node-name>.<tailnet>.ts.net/googlechat`

Votre tableau de bord privé reste accessible uniquement via le tailnet :
`https://<node-name>.<tailnet>.ts.net:8443/`

Utilisez l'URL publique (sans `:8443`) dans la configuration de l'application Google Chat.

> Note : Cette configuration persiste entre les redémarrages. Pour la supprimer ultérieurement, exécutez `tailscale funnel reset` et `tailscale serve reset`.

### Option B : Proxy inverse (Caddy)

Si vous utilisez un proxy inverse comme Caddy, ne proxy que le chemin spécifique :

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Avec cette configuration, toute requête vers `your-domain.com/` sera ignorée ou retournera une 404, tandis que `your-domain.com/googlechat` est correctement routée vers OpenClaw.

### Option C : Tunnel Cloudflare

Configurez les règles d'entrée de votre tunnel pour ne router que le chemin du webhook :

- **Chemin** : `/googlechat` -> `http://localhost:18789/googlechat`
- **Règle par défaut** : HTTP 404 (Not Found)

## Fonctionnement

1. Google Chat envoie des webhooks POST au Gateway. Chaque requête inclut un en-tête `Authorization: Bearer <token>`.
2. OpenClaw vérifie le token par rapport aux `audienceType` + `audience` configurés :
   - `audienceType: "app-url"` : l'audience est votre URL de webhook HTTPS.
   - `audienceType: "project-number"` : l'audience est le numéro du projet Cloud.
3. Les messages sont routés par espace :
   - Les Messages privés utilisent la clé de session `agent:<agentId>:googlechat:dm:<spaceId>`.
   - Les espaces utilisent la clé de session `agent:<agentId>:googlechat:group:<spaceId>`.
4. L'accès MP est en appairage par défaut. Les expéditeurs inconnus reçoivent un code d'appairage ; approuvez avec :
   - `openclaw pairing approve googlechat <code>`
5. Les espaces de groupe nécessitent un @-mention par défaut. Utilisez `botUser` si la détection de mention nécessite le nom d'utilisateur de l'application.

## Cibles

Utilisez ces identifiants pour la livraison et les listes autorisées :

- Messages privés : `users/<userId>` (recommandé).
- L'e-mail brut `name@example.com` est mutable et n'est utilisé que pour la correspondance directe de liste autorisée lorsque `channels.googlechat.dangerouslyAllowNameMatching: true`.
- Obsolète : `users/<email>` est traité comme un identifiant utilisateur, pas une liste autorisée par e-mail.
- Espaces : `spaces/<spaceId>`.

## Points clés de configuration

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // ou serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optionnel ; aide à la détection de mention
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Notes :

- Les identifiants du compte de service peuvent aussi être passés en ligne avec `serviceAccount` (chaîne JSON).
- `serviceAccountRef` est également pris en charge (SecretRef env/file), y compris les refs par compte sous `channels.googlechat.accounts.<id>.serviceAccountRef`.
- Le chemin de webhook par défaut est `/googlechat` si `webhookPath` n'est pas défini.
- `dangerouslyAllowNameMatching` réactive la correspondance mutable par e-mail pour les listes autorisées (mode de compatibilité d'urgence).
- Les réactions sont disponibles via l'outil `reactions` et `channels action` lorsque `actions.reactions` est activé.
- `typingIndicator` prend en charge `none`, `message` (par défaut) et `reaction` (reaction nécessite un OAuth utilisateur).
- Les pièces jointes sont téléchargées via l'API Chat et stockées dans le pipeline média (taille limitée par `mediaMaxMb`).

Details de référence des secrets : [Gestion des secrets](/gateway/secrets).

## Dépannage

### 405 Method Not Allowed

Si Google Cloud Logs Explorer affiche des erreurs comme :

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Cela signifie que le gestionnaire de webhook n'est pas enregistré. Causes courantes :

1. **Canal non configure** : La section `channels.googlechat` est absente de votre configuration. Vérifiez avec :

   ```bash
   openclaw config get channels.googlechat
   ```

   Si cela retourne « Config path not found », ajoutez la configuration (voir [Points clés de configuration](#points-clés-de-configuration)).

2. **Plugin non active** : Vérifiez le statut du plugin :

   ```bash
   openclaw plugins list | grep googlechat
   ```

   S'il affiche « disabled », ajoutez `plugins.entries.googlechat.enabled: true` à votre configuration.

3. **Gateway non redémarre** : Après avoir ajouté la configuration, redémarrez le Gateway :

   ```bash
   openclaw gateway restart
   ```

Vérifiez que le canal est en cours d'exécution :

```bash
openclaw channels status
# Devrait afficher : Google Chat default: enabled, configured, ...
```

### Autres problèmes

- Vérifiez `openclaw channels status --probe` pour les erreurs d'authentification ou la configuration d'audience manquante.
- Si aucun message n'arrive, confirmez l'URL du webhook de l'application Chat + les abonnements aux événements.
- Si le filtrage par mention bloque les réponses, définissez `botUser` sur le nom de ressource utilisateur de l'application et vérifiez `requireMention`.
- Utilisez `openclaw logs --follow` pendant l'envoi d'un message test pour voir si les requêtes atteignent le Gateway.

Documents associés :

- [Configuration du Gateway](/gateway/configuration)
- [Sécurité](/gateway/security)
- [Réactions](/tools/reactions)
