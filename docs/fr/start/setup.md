---
read_when:
  - Installation sur une nouvelle machine
  - Vous voulez la dernière version sans casser votre configuration personnelle
summary: Configuration avancée et flux de travail de développement pour OpenClaw
title: Configuration
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/setup.md
  workflow: manual
---

# Configuration

<Note>
Si vous configurez pour la première fois, commencez par [Premiers pas](/start/getting-started).
Pour les détails de l'assistant, consultez [Assistant de configuration initiale](/start/wizard).
</Note>

Dernière mise à jour : 2026-01-01

## TL;DR

- **La personnalisation vit en dehors du dépôt :** `~/.openclaw/workspace` (espace de travail) + `~/.openclaw/openclaw.json` (configuration).
- **Flux stable :** installez l'application macOS ; laissez-la exécuter le Gateway intégré.
- **Flux de pointe :** exécutez le Gateway vous-même via `pnpm gateway:watch`, puis laissez l'application macOS se connecter en mode Local.

## Prérequis (depuis les sources)

- Node `>=22`
- `pnpm`
- Docker (optionnel ; uniquement pour la configuration conteneurisée/e2e — voir [Docker](/install/docker))

## Stratégie de personnalisation (pour que les mises à jour ne cassent rien)

Si vous voulez « 100 % personnalisé pour moi » _et_ des mises à jour faciles, gardez vos personnalisations dans :

- **Configuration :** `~/.openclaw/openclaw.json` (JSON/JSON5)
- **Espace de travail :** `~/.openclaw/workspace` (skills, prompts, mémoires ; faites-en un dépôt git privé)

Initialisation :

```bash
openclaw setup
```

Depuis ce dépôt, utilisez l'entrée CLI locale :

```bash
openclaw setup
```

Si vous n'avez pas encore d'installation globale, exécutez via `pnpm openclaw setup`.

## Exécuter le Gateway depuis ce dépôt

Après `pnpm build`, vous pouvez exécuter le CLI packagé directement :

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## Flux stable (application macOS d'abord)

1. Installez et lancez **OpenClaw.app** (barre de menus).
2. Complétez la checklist de configuration initiale/permissions (invites TCC).
3. Assurez-vous que le Gateway est **Local** et en cours d'exécution (l'application le gère).
4. Liez les surfaces (exemple : WhatsApp) :

```bash
openclaw channels login
```

5. Vérification :

```bash
openclaw health
```

Si la configuration initiale n'est pas disponible dans votre build :

- Exécutez `openclaw setup`, puis `openclaw channels login`, puis démarrez le Gateway manuellement (`openclaw gateway`).

## Flux de pointe (Gateway dans un terminal)

Objectif : travailler sur le Gateway TypeScript, obtenir le rechargement à chaud, garder l'interface macOS connectée.

### 0) (Optionnel) Exécuter aussi l'application macOS depuis les sources

Si vous voulez aussi l'application macOS à la pointe :

```bash
./scripts/restart-mac.sh
```

### 1) Démarrer le Gateway de développement

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` exécute le gateway en mode surveillance et recharge lors des changements TypeScript.

### 2) Pointer l'application macOS vers votre Gateway en cours d'exécution

Dans **OpenClaw.app** :

- Mode de connexion : **Local**
  L'application se connectera au gateway en cours d'exécution sur le port configuré.

### 3) Vérifier

- L'état du Gateway dans l'application devrait afficher **"Using existing gateway …"**
- Ou via CLI :

```bash
openclaw health
```

### Pièges courants

- **Mauvais port :** le WS du Gateway est par défaut `ws://127.0.0.1:18789` ; gardez l'application et le CLI sur le même port.
- **Emplacement de l'état :**
  - Identifiants : `~/.openclaw/credentials/`
  - Sessions : `~/.openclaw/agents/<agentId>/sessions/`
  - Logs : `/tmp/openclaw/`

## Carte de stockage des identifiants

Utilisez ceci pour déboguer l'authentification ou décider quoi sauvegarder :

- **WhatsApp** : `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- **Jeton bot Telegram** : configuration/env ou `channels.telegram.tokenFile`
- **Jeton bot Discord** : configuration/env (fichier de jeton non encore supporté)
- **Jetons Slack** : configuration/env (`channels.slack.*`)
- **Listes d'appairage autorisées** :
  - `~/.openclaw/credentials/<channel>-allowFrom.json` (compte par défaut)
  - `~/.openclaw/credentials/<channel>-<accountId>-allowFrom.json` (comptes non par défaut)
- **Profils d'authentification de modèle** : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **Payload de secrets stocké dans un fichier (optionnel)** : `~/.openclaw/secrets.json`
- **Import OAuth ancien** : `~/.openclaw/credentials/oauth.json`
  Plus de détails : [Sécurité](/gateway/security#credential-storage-map).

## Mise à jour (sans casser votre configuration)

- Gardez `~/.openclaw/workspace` et `~/.openclaw/` comme « vos affaires » ; ne mettez pas de prompts/configurations personnels dans le dépôt `openclaw`.
- Mise à jour des sources : `git pull` + `pnpm install` (quand le lockfile change) + continuez à utiliser `pnpm gateway:watch`.

## Linux (service utilisateur systemd)

Les installations Linux utilisent un service **utilisateur** systemd. Par défaut, systemd arrête
les services utilisateur à la déconnexion/inactivité, ce qui tue le Gateway. La configuration initiale tente d'activer
le lingering pour vous (peut demander sudo). Si c'est toujours désactivé, exécutez :

```bash
sudo loginctl enable-linger $USER
```

Pour les serveurs toujours allumés ou multi-utilisateurs, envisagez un service **système** au lieu d'un
service utilisateur (pas de lingering nécessaire). Voir [Guide opérationnel du Gateway](/gateway) pour les notes systemd.

## Documentation associée

- [Guide opérationnel du Gateway](/gateway) (options, supervision, ports)
- [Configuration du Gateway](/gateway/configuration) (schéma de configuration + exemples)
- [Discord](/channels/discord) et [Telegram](/channels/telegram) (tags de réponse + paramètres replyToMode)
- [Configuration d'assistant OpenClaw](/start/openclaw)
- [Application macOS](/platforms/macos) (cycle de vie du gateway)
