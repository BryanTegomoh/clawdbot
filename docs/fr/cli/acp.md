---
summary: "Exécuter le pont ACP pour les intégrations IDE"
read_when:
  - Configuration des intégrations IDE basées sur ACP
  - Débogage du routage de session ACP vers le Gateway
title: "acp"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/acp.md
  workflow: manual
---

# acp

Exécuter le pont ACP (Agent Client Protocol) qui communique avec un Gateway OpenClaw.

Cette commande parle ACP via stdio pour les IDE et transmet les invites au Gateway via WebSocket. Elle maintient les sessions ACP associées aux clés de session du Gateway.

## Utilisation

```bash
openclaw acp

# Gateway distant
openclaw acp --url wss://gateway-host:18789 --token <token>

# Gateway distant (jeton depuis un fichier)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Se rattacher à une clé de session existante
openclaw acp --session agent:main:main

# Se rattacher par libellé (doit déjà exister)
openclaw acp --session-label "support inbox"

# Réinitialiser la clé de session avant la première invite
openclaw acp --session agent:main:main --reset-session
```

## Client ACP (debug)

Utilisez le client ACP intégré pour vérifier le bon fonctionnement du pont sans IDE.
Il lance le pont ACP et vous permet de saisir des invites de manière interactive.

```bash
openclaw acp client

# Pointer le pont lancé vers un Gateway distant
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# Surcharger la commande serveur (défaut : openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

Modèle de permissions (mode debug client) :

- L'approbation automatique est basée sur une liste d'autorisation et ne s'applique qu'aux identifiants d'outils principaux de confiance.
- L'approbation automatique de `read` est limitée au répertoire de travail actuel (`--cwd` lorsque défini).
- Les noms d'outils inconnus/non principaux, les lectures hors portée et les outils dangereux nécessitent toujours une approbation explicite par invite.
- Le `toolCall.kind` fourni par le serveur est traite comme une métadonnée non fiable (pas une source d'autorisation).

## Comment utiliser ceci

Utilisez ACP lorsqu'un IDE (ou un autre client) parle Agent Client Protocol et que vous souhaitez qu'il pilote une session Gateway OpenClaw.

1. Assurez-vous que le Gateway fonctionne (local ou distant).
2. Configurez la cible Gateway (configuration ou options).
3. Configurez votre IDE pour exécuter `openclaw acp` via stdio.

Exemple de configuration (persistée) :

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

Exemple d'exécution directe (sans écriture de configuration) :

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# préféré pour la sécurité des processus locaux
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## Sélection des agents

ACP ne choisit pas directement les agents. Il route par la clé de session Gateway.

Utilisez des clés de session limitées à l'agent pour cibler un agent spécifique :

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

Chaque session ACP correspond à une seule clé de session Gateway. Un agent peut avoir plusieurs sessions ; ACP utilisé par défaut une session isolée `acp:<uuid>` sauf si vous surchargez la clé ou le libellé.

## Configuration de l'éditeur Zed

Ajoutez un agent ACP personnalisé dans `~/.config/zed/settings.json` (ou utilisez l'interface des paramètres de Zed) :

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

Pour cibler un Gateway ou un agent spécifique :

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Dans Zed, ouvrez le panneau Agent et sélectionnez "OpenClaw ACP" pour démarrer un fil de discussion.

## Mappage de sessions

Par défaut, les sessions ACP obtiennent une clé de session Gateway isolée avec un préfixe `acp:`.
Pour réutiliser une session connue, passez une clé de session ou un libellé :

- `--session <key>` : utiliser une clé de session Gateway spécifique.
- `--session-label <label>` : résoudre une session existante par libellé.
- `--reset-session` : créer un nouvel identifiant de session pour cette clé (même clé, nouvelle transcription).

Si votre client ACP supporte les metadonnées, vous pouvez surcharger par session :

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

En savoir plus sur les clés de session sur [/concepts/session](/concepts/session).

## Options

- `--url <url>` : URL WebSocket du Gateway (par défaut gateway.remote.url lorsque configuré).
- `--token <token>` : jeton d'authentification du Gateway.
- `--token-file <path>` : lire le jeton d'authentification du Gateway depuis un fichier.
- `--password <password>` : mot de passe d'authentification du Gateway.
- `--password-file <path>` : lire le mot de passe d'authentification du Gateway depuis un fichier.
- `--session <key>` : clé de session par défaut.
- `--session-label <label>` : libellé de session par défaut à résoudre.
- `--require-existing` : échouer si la clé/le libellé de session n'existe pas.
- `--reset-session` : réinitialiser la clé de session avant la première utilisation.
- `--no-prefix-cwd` : ne pas préfixer les invites avec le répertoire de travail.
- `--verbose, -v` : journalisation détaillée sur stderr.

Note de sécurité :

- `--token` et `--password` peuvent être visibles dans les listes de processus locaux sur certains systèmes.
- Préférez `--token-file`/`--password-file` ou les variables d'environnement (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`).

### Options `acp client`

- `--cwd <dir>` : répertoire de travail pour la session ACP.
- `--server <command>` : commande du serveur ACP (défaut : `openclaw`).
- `--server-args <args...>` : arguments supplémentaires passes au serveur ACP.
- `--server-verbose` : activer la journalisation détaillée sur le serveur ACP.
- `--verbose, -v` : journalisation détaillée du client.
