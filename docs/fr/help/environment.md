---
summary: "Où OpenClaw charge les variables d'environnement et l'ordre de priorité"
read_when:
  - Vous avez besoin de savoir quelles variables d'environnement sont chargées, et dans quel ordre
  - Vous déboguez des clés API manquantes dans le Gateway
  - Vous documentez l'authentification des fournisseurs ou les environnements de déploiement
title: "Variables d'environnement"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: help/environment.md
  workflow: manual
---

# Variables d'environnement

OpenClaw récupère les variables d'environnement depuis plusieurs sources. La règle est de **ne jamais écraser les valeurs existantes**.

## Priorité (la plus haute → la plus basse)

1. **Environnement du processus** (ce que le processus Gateway possède déjà depuis le shell/démon parent).
2. **`.env` dans le répertoire de travail courant** (comportement par défaut de dotenv ; n'écrase pas).
3. **`.env` global** situé à `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env` ; n'écrase pas).
4. **Bloc `env` de la configuration** dans `~/.openclaw/openclaw.json` (appliqué uniquement si absent).
5. **Import optionnel du shell de connexion** (`env.shellEnv.enabled` ou `OPENCLAW_LOAD_SHELL_ENV=1`), appliqué uniquement pour les clés attendues manquantes.

Si le fichier de configuration est entièrement absent, l'étape 4 est ignorée ; l'import du shell s'exécute quand même s'il est activé.

## Bloc `env` de la configuration

Deux façons équivalentes de définir des variables d'environnement en ligne (les deux ne sont pas écrasantes) :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Import de l'environnement du shell

`env.shellEnv` exécute votre shell de connexion et importe uniquement les clés attendues **manquantes** :

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Équivalents en variables d'environnement :

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## Substitution de variables d'environnement dans la configuration

Vous pouvez référencer des variables d'environnement directement dans les valeurs de chaîne de la configuration en utilisant la syntaxe `${VAR_NAME}` :

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

Voir [Configuration : Substitution de variables d'environnement](/gateway/configuration#env-var-substitution-in-config) pour tous les détails.

## Secret refs vs chaînes `${ENV}`

OpenClaw prend en charge deux patterns basés sur les variables d'environnement :

- Substitution de chaîne `${VAR}` dans les valeurs de configuration.
- Objets SecretRef (`{ source: "env", provider: "default", id: "VAR" }`) pour les champs qui supportent les références de secrets.

Les deux sont résolus depuis l'environnement du processus au moment de l'activation. Les détails sur les SecretRef sont documentés dans [Gestion des secrets](/gateway/secrets).

## Variables d'environnement liées aux chemins

| Variable               | Objectif                                                                                                                                                                          |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_HOME`        | Remplace le répertoire personnel utilisé pour toute la résolution de chemins internes (`~/.openclaw/`, répertoires d'agents, sessions, identifiants). Utile lorsqu'OpenClaw est exécuté en tant qu'utilisateur de service dédié. |
| `OPENCLAW_STATE_DIR`   | Remplace le répertoire d'état (par défaut `~/.openclaw`).                                                                                                                            |
| `OPENCLAW_CONFIG_PATH` | Remplace le chemin du fichier de configuration (par défaut `~/.openclaw/openclaw.json`).                                                                                                             |

## Journalisation

| Variable             | Objectif                                                                                                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_LOG_LEVEL` | Remplace le niveau de journalisation pour le fichier et la console (ex. `debug`, `trace`). A priorité sur `logging.level` et `logging.consoleLevel` dans la configuration. Les valeurs invalides sont ignorées avec un avertissement. |

### `OPENCLAW_HOME`

Lorsqu'il est défini, `OPENCLAW_HOME` remplace le répertoire personnel du système (`$HOME` / `os.homedir()`) pour toute la résolution de chemins internes. Cela permet une isolation complète du système de fichiers pour les comptes de service sans interface graphique.

**Priorité :** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

**Exemple** (LaunchDaemon macOS) :

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` peut également être défini avec un chemin tilde (ex. `~/svc`), qui est résolu en utilisant `$HOME` avant utilisation.

## Liens connexes

- [Configuration du Gateway](/gateway/configuration)
- [FAQ : variables d'environnement et chargement .env](/help/faq#env-vars-and-env-loading)
- [Vue d'ensemble des modèles](/concepts/models)
