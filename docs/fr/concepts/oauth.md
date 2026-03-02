---
summary: "OAuth dans OpenClaw : échange de tokens, stockage et modèles multi-comptes"
read_when:
  - Vous voulez comprendre le OAuth d'OpenClaw de bout en bout
  - Vous rencontrez des problèmes d'invalidation de token / déconnexion
  - Vous voulez le setup-token ou les flux d'authentification OAuth
  - Vous voulez plusieurs comptes ou le routage de profils
title: "OAuth"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/oauth.md
  workflow: manual
---

# OAuth

OpenClaw supporte l'authentification par abonnement via OAuth pour les fournisseurs qui l'offrent (notamment **OpenAI Codex (OAuth ChatGPT)**). Pour les abonnements Anthropic, utilisez le flux **setup-token**. Cette page explique :

- comment l'**échange de tokens** OAuth fonctionne (PKCE)
- ou les tokens sont **stockés** (et pourquoi)
- comment gérer **plusieurs comptes** (profils + surcharges par session)

OpenClaw supporte également les **plugins de fournisseurs** qui livrent leurs propres flux OAuth ou de clé API. Exécutez-les via :

```bash
openclaw models auth login --provider <id>
```

Pour l'authentification basee sur SecretRef (fournisseurs `env`/`file`/`exec`), voir [Gestion des secrets](/gateway/secrets).

## Le puits de tokens (pourquoi il existe)

Les fournisseurs OAuth émettent couramment un **nouveau refresh token** lors des flux de connexion/rafraîchissement. Certains fournisseurs (ou clients OAuth) peuvent invalider les anciens refresh tokens quand un nouveau est émis pour le même utilisateur/application.

Symptôme pratique :

- vous vous connectez via OpenClaw _et_ via Claude Code / Codex CLI, et l'un d'eux est aléatoirement « déconnecté » plus tard

Pour réduire cela, OpenClaw traite `auth-profiles.json` comme un **puits de tokens** :

- le runtime lit les identifiants depuis **un seul endroit**
- nous pouvons garder plusieurs profils et les router de manière déterministe

## Stockage (ou résident les tokens)

Les secrets sont stockés **par agent** :

- Profils d'authentification (OAuth + clés API + refs optionnelles au niveau valeur) : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- Fichier de compatibilité ancien : `~/.openclaw/agents/<agentId>/agent/auth.json`
  (les entrées `api_key` statiques sont nettoyees lorsqu'elles sont decouvertes)

Ancien fichier d'import uniquement (encore supporté, mais pas le magasin principal) :

- `~/.openclaw/credentials/oauth.json` (importé dans `auth-profiles.json` à la première utilisation)

Tous les éléments ci-dessus respectent également `$OPENCLAW_STATE_DIR` (surcharge du répertoire d'état). Référence complète : [/gateway/configuration](/gateway/configuration#auth-storage-oauth--api-keys)

Pour les refs de secrets statiques et le comportement d'activation de l'instantane d'exécution, voir [Gestion des secrets](/gateway/secrets).

## Setup-token Anthropic (authentification par abonnement)

Exécutez `claude setup-token` sur n'importe quelle machine, puis collez-le dans OpenClaw :

```bash
openclaw models auth setup-token --provider anthropic
```

Si vous avez généré le token ailleurs, collez-le manuellement :

```bash
openclaw models auth paste-token --provider anthropic
```

Vérifiez :

```bash
openclaw models status
```

## Échange OAuth (comment la connexion fonctionne)

Les flux de connexion interactifs d'OpenClaw sont implémentés dans `@mariozechner/pi-ai` et connectés aux assistants/commandes.

### Setup-token Anthropic (Claude Pro/Max)

Forme du flux :

1. exécuter `claude setup-token`
2. coller le token dans OpenClaw
3. stocker comme profil d'authentification par token (pas de rafraîchissement)

Le chemin de l'assistant est `openclaw onboard` -> choix d'authentification `setup-token` (Anthropic).

### OpenAI Codex (OAuth ChatGPT)

Forme du flux (PKCE) :

1. générer un vérificateur/challenge PKCE + `state` aléatoire
2. ouvrir `https://auth.openai.com/oauth/authorize?...`
3. essayer de capturer le callback sur `http://127.0.0.1:1455/auth/callback`
4. si le callback ne peut pas se lier (ou vous êtes distant/headless), coller l'URL/code de redirection
5. échanger à `https://auth.openai.com/oauth/token`
6. extraire `accountId` du token d'accès et stocker `{ access, refresh, expires, accountId }`

Le chemin de l'assistant est `openclaw onboard` -> choix d'authentification `openai-codex`.

## Rafraîchissement + expiration

Les profils stockent un horodatage `expires`.

A l'exécution :

- si `expires` est dans le futur, utiliser le token d'accès stocké
- si expiré, rafraîchir (sous un verrou de fichier) et écraser les identifiants stockés

Le flux de rafraîchissement est automatique ; vous n'avez généralement pas besoin de gérer les tokens manuellement.

## Comptes multiples (profils) + routage

Deux modèles :

### 1) Préféré : agents séparés

Si vous voulez que « personnel » et « travail » n'interagissent jamais, utilisez des agents isolés (sessions + identifiants + espace de travail séparés) :

```bash
openclaw agents add work
openclaw agents add personal
```

Puis configurez l'authentification par agent (assistant) et routez les chats vers le bon agent.

### 2) Avancé : profils multiples dans un agent

`auth-profiles.json` supporte plusieurs identifiants de profil pour le même fournisseur.

Choisir quel profil est utilisé :

- globalement via l'ordonnancement de configuration (`auth.order`)
- par session via `/model ...@<profileId>`

Exemple (surcharge de session) :

- `/model Opus@anthropic:work`

Comment voir quels identifiants de profil existent :

- `openclaw channels list --json` (affiche `auth[]`)

Documentation associée :

- [/concepts/model-failover](/concepts/model-failover) (rotation + règles de temps de repos)
- [/tools/slash-commands](/tools/slash-commands) (surface de commande)
