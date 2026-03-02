---
summary: "Push Gmail Pub/Sub câblé aux webhooks OpenClaw via gogcli"
read_when:
  - Câblage de déclencheurs de boîte Gmail vers OpenClaw
  - Configuration du push Pub/Sub pour le réveil de l'agent
title: "Gmail PubSub"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/gmail-pubsub.md
  workflow: manual
---

# Gmail Pub/Sub -> OpenClaw

Objectif : Gmail watch -> push Pub/Sub -> `gog gmail watch serve` -> webhook OpenClaw.

## Prérequis

- `gcloud` installé et connecte ([guide d'installation](https://docs.cloud.google.com/sdk/docs/install-sdk)).
- `gog` (gogcli) installé et autorise pour le compte Gmail ([gogcli.sh](https://gogcli.sh/)).
- Hooks OpenClaw activés (voir [Webhooks](/automation/webhook)).
- `tailscale` connecté ([tailscale.com](https://tailscale.com/)). La configuration prise en charge utilise Tailscale Funnel pour le point de terminaison HTTPS public.
  D'autres services de tunnel peuvent fonctionner, mais sont en DIY/non supportés et nécessitent un câblage manuel.
  Pour l'instant, Tailscale est ce que nous supportons.

Exemple de configuration de hook (activer le mapping de preset Gmail) :

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

Pour livrer le résumé Gmail vers une surface de chat, surchargez le preset avec un mapping
qui définit `deliver` + optionnellement `channel`/`to` :

```json5
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

Si vous voulez un canal fixe, définissez `channel` + `to`. Sinon `channel: "last"`
utilise la dernière route de livraison (repli vers WhatsApp).

Pour forcer un modèle moins cher pour les exécutions Gmail, définissez `model` dans le mapping
(`fournisseur/modèle` ou alias). Si vous imposez `agents.defaults.models`, incluez-le dans la liste.

Pour définir un modèle et un niveau de réflexion par défaut spécifiquement pour les hooks Gmail, ajoutez
`hooks.gmail.model` / `hooks.gmail.thinking` dans votre configuration :

```json5
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

Notes :

- Les `model`/`thinking` par hook dans le mapping surchargent toujours ces valeurs par défaut.
- Ordre de repli : `hooks.gmail.model` -> `agents.defaults.model.fallbacks` -> principal (auth/rate-limit/timeouts).
- Si `agents.defaults.models` est défini, le modèle Gmail doit être dans la liste autorisée.
- Le contenu du hook Gmail est encadré par des limités de sécurité de contenu externe par défaut.
  Pour désactiver (dangereux), définissez `hooks.gmail.allowUnsafeExternalContent: true`.

Pour personnaliser davantage le traitement de la charge utile, ajoutez `hooks.mappings` ou un module de transformation JS/TS
sous `~/.openclaw/hooks/transforms` (voir [Webhooks](/automation/webhook)).

## Assistant (recommandé)

Utilisez l'assistant OpenClaw pour tout câbler ensemble (installe les dépendances sur macOS via brew) :

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Valeurs par défaut :

- Utilise Tailscale Funnel pour le point de terminaison push public.
- Écrit la configuration `hooks.gmail` pour `openclaw webhooks gmail run`.
- Active le preset de hook Gmail (`hooks.presets: ["gmail"]`).

Note sur le chemin : quand `tailscale.mode` est activé, OpenClaw définit automatiquement
`hooks.gmail.serve.path` à `/` et garde le chemin public à
`hooks.gmail.tailscale.path` (par défaut `/gmail-pubsub`) car Tailscale
supprime le préfixe set-path avant le proxying.
Si vous avez besoin que le backend reçoive le chemin préfixé, définissez
`hooks.gmail.tailscale.target` (ou `--tailscale-target`) avec une URL complète comme
`http://127.0.0.1:8788/gmail-pubsub` et faites correspondre `hooks.gmail.serve.path`.

Vous voulez un point de terminaison personnalisé ? Utilisez `--push-endpoint <url>` ou `--tailscale off`.

Note de plateforme : sur macOS l'assistant installe `gcloud`, `gogcli`, et `tailscale`
via Homebrew ; sur Linux installez-les manuellement d'abord.

Démarrage automatique du Gateway (recommandé) :

- Quand `hooks.enabled=true` et `hooks.gmail.account` est défini, le Gateway démarre
  `gog gmail watch serve` au démarrage et renouvelle automatiquement le watch.
- Définissez `OPENCLAW_SKIP_GMAIL_WATCHER=1` pour désactiver (utile si vous exécutez le démon vous-même).
- N'exécutez pas le démon manuel en même temps, sinon vous obtiendrez
  `listen tcp 127.0.0.1:8788: bind: address already in use`.

Démon manuel (démarre `gog gmail watch serve` + renouvellement automatique) :

```bash
openclaw webhooks gmail run
```

## Configuration unique

1. Sélectionnez le projet GCP **qui possède le client OAuth** utilisé par `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Note : Gmail watch nécessite que le sujet Pub/Sub soit dans le même projet que le client OAuth.

2. Activer les APIs :

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3. Créer un sujet :

```bash
gcloud pubsub topics create gog-gmail-watch
```

4. Autoriser le push Gmail à publier :

```bash
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## Démarrer le watch

```bash
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Notez le `history_id` de la sortie (pour le débogage).

## Exécuter le gestionnaire push

Exemple local (authentification par jeton partagé) :

```bash
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Notes :

- `--token` protège le point de terminaison push (`x-gog-token` ou `?token=`).
- `--hook-url` pointe vers OpenClaw `/hooks/gmail` (mappé ; exécution isolée + résumé vers le principal).
- `--include-body` et `--max-bytes` contrôlent l'extrait du corps envoyé à OpenClaw.

Recommandé : `openclaw webhooks gmail run` encapsule le même flux et renouvelle automatiquement le watch.

## Exposer le gestionnaire (avancé, non supporte)

Si vous avez besoin d'un tunnel non-Tailscale, câblez-le manuellement et utilisez l'URL publique dans
l'abonnement push (non supporte, sans garde-fous) :

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Utilisez l'URL générée comme point de terminaison push :

```bash
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Production : utilisez un point de terminaison HTTPS stable et configurez Pub/Sub OIDC JWT, puis exécutez :

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## Test

Envoyez un message à la boîte de réception surveillée :

```bash
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Vérifiez l'état du watch et l'historique :

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## Dépannage

- `Invalid topicName` : projet incompatible (le sujet n'est pas dans le projet du client OAuth).
- `User not authorized` : `rôles/pubsub.publisher` manquant sur le sujet.
- Messages vides : le push Gmail ne fournit que `historyId` ; récupérez via `gog gmail history`.

## Nettoyage

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```
