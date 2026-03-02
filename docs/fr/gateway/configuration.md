---
summary: "Aperçu de la configuration : tâches courantes, mise en route rapide et liens vers la référence complète"
read_when:
  - Première configuration d'OpenClaw
  - Recherche de modèles de configuration courants
  - Navigation vers des sections de configuration spécifiques
title: "Configuration"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/configuration.md
  workflow: manual
---

# Configuration

OpenClaw lit une configuration optionnelle en <Tooltip tip="JSON5 prend en charge les commentaires et les virgules finales">**JSON5**</Tooltip> depuis `~/.openclaw/openclaw.json`.

Si le fichier est absent, OpenClaw utilise des valeurs par défaut sûres. Raisons courantes d'ajouter une configuration :

- Connecter des canaux et contrôler qui peut envoyer des messages au bot
- Définir les modèles, outils, isolation en bac à sable ou automatisation (cron, hooks)
- Ajuster les sessions, médias, réseau ou interface

Voir la [référence complète](/gateway/configuration-reference) pour tous les champs disponibles.

<Tip>
**Nouveau dans la configuration ?** Commencez avec `openclaw onboard` pour une configuration interactive, ou consultez le guide [Exemples de configuration](/gateway/configuration-examples) pour des configurations complètes à copier-coller.
</Tip>

## Configuration minimale

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Modifier la configuration

<Tabs>
  <Tab title="Assistant interactif">
    ```bash
    openclaw onboard       # assistant de configuration complet
    openclaw configure     # assistant de configuration
    ```
  </Tab>
  <Tab title="CLI (commandes rapides)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Interface de contrôle">
    Ouvrez [http://127.0.0.1:18789](http://127.0.0.1:18789) et utilisez l'onglet **Config**.
    L'Interface de contrôle affiche un formulaire à partir du schéma de configuration, avec un éditeur **Raw JSON** comme solution de secours.
  </Tab>
  <Tab title="Édition directe">
    Modifiez `~/.openclaw/openclaw.json` directement. Le Gateway surveille le fichier et applique les modifications automatiquement (voir [rechargement à chaud](#config-hot-reload)).
  </Tab>
</Tabs>

## Validation stricte

<Warning>
OpenClaw n'accepte que les configurations correspondant entièrement au schéma. Les clés inconnues, types malformés ou valeurs invalides empêchent le Gateway de **démarrer**. La seule exception au niveau racine est `$schema` (chaîne), pour que les éditeurs puissent attacher des métadonnées JSON Schema.
</Warning>

Lorsque la validation échoue :

- Le Gateway ne démarre pas
- Seules les commandes de diagnostic fonctionnent (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Exécutez `openclaw doctor` pour voir les problèmes exacts
- Exécutez `openclaw doctor --fix` (ou `--yes`) pour appliquer les réparations

## Tâches courantes

<AccordionGroup>
  <Accordion title="Configurer un canal (WhatsApp, Telegram, Discord, etc.)">
    Chaque canal a sa propre section de configuration sous `channels.<fournisseur>`. Consultez la page dédiée au canal pour les étapes de configuration :

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Tous les canaux partagent le même modèle de politique DM :

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // uniquement pour allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Choisir et configurer les modèles">
    Définissez le modèle principal et les replis optionnels :

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` définit le catalogue de modèles et sert de liste autorisée pour `/model`.
    - Les références de modèle utilisent le format `fournisseur/modèle` (par exemple `anthropic/claude-opus-4-6`).
    - `agents.defaults.imageMaxDimensionPx` contrôle la réduction d'échelle des images dans les transcriptions/outils (défaut `1200`) ; des valeurs plus basses réduisent généralement l'utilisation de tokens de vision sur les exécutions lourdes en captures d'écran.
    - Voir [CLI des modèles](/concepts/models) pour changer de modèle en chat et [Basculement de modèle](/concepts/model-failover) pour la rotation d'authentification et le comportement de repli.
    - Pour les fournisseurs personnalisés/auto-hébergés, voir [Fournisseurs personnalisés](/gateway/configuration-reference#custom-providers-and-base-urls) dans la référence.

  </Accordion>

  <Accordion title="Contrôler qui peut envoyer des messages au bot">
    L'accès DM est contrôlé par canal via `dmPolicy` :

    - `"pairing"` (par défaut) : les expéditeurs inconnus reçoivent un code d'appairage unique à approuver
    - `"allowlist"` : uniquement les expéditeurs dans `allowFrom` (ou le magasin d'appairage autorisé)
    - `"open"` : autoriser tous les DM entrants (nécessite `allowFrom: ["*"]`)
    - `"disabled"` : ignorer tous les DM

    Pour les groupes, utilisez `groupPolicy` + `groupAllowFrom` ou les listes autorisées spécifiques au canal.

    Voir la [référence complète](/gateway/configuration-reference#dm-and-group-access) pour les détails par canal.

  </Accordion>

  <Accordion title="Configurer le filtrage par mention en groupe">
    Les messages de groupe nécessitent par défaut une **mention**. Configurez les modèles par agent :

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Mentions de métadonnées** : @-mentions natifs (mention par appui WhatsApp, @bot Telegram, etc.)
    - **Modèles textuels** : patterns regex dans `mentionPatterns`
    - Voir la [référence complète](/gateway/configuration-reference#group-chat-mention-gating) pour les remplacements par canal et le mode auto-chat.

  </Accordion>

  <Accordion title="Configurer les sessions et réinitialisations">
    Les sessions contrôlent la continuité et l'isolation des conversations :

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // recommandé pour multi-utilisateur
        threadBindings: {
          enabled: true,
          ttlHours: 24,
        },
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope` : `main` (partagé) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - `threadBindings` : valeurs par défaut globales pour le routage de session lie aux threads (Discord prend en charge `/focus`, `/unfocus`, `/agents` et `/session ttl`).
    - Voir [Gestion des sessions](/concepts/session) pour la portée, les liens d'identité et la politique d'envoi.
    - Voir la [référence complète](/gateway/configuration-reference#session) pour tous les champs.

  </Accordion>

  <Accordion title="Activer l'isolation en bac à sable">
    Exécutez les sessions agent dans des conteneurs Docker isolés :

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    Construisez l'image d'abord : `scripts/sandbox-setup.sh`

    Voir [Isolation en bac à sable](/gateway/sandboxing) pour le guide complet et la [référence complète](/gateway/configuration-reference#sandbox) pour toutes les options.

  </Accordion>

  <Accordion title="Configurer le heartbeat (vérifications périodiques)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every` : chaîne de durée (`30m`, `2h`). Définir `0m` pour désactiver.
    - `target` : `last` | `whatsapp` | `telegram` | `discord` | `none`
    - `directPolicy` : `allow` (par défaut) ou `block` pour les cibles heartbeat de type DM
    - Voir [Heartbeat](/gateway/heartbeat) pour le guide complet.

  </Accordion>

  <Accordion title="Configurer les tâches cron">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
        runLog: {
          maxBytes: "2mb",
          keepLines: 2000,
        },
      },
    }
    ```

    - `sessionRetention` : purge des sessions d'exécution isolées terminées de `sessions.json` (défaut `24h` ; définir `false` pour désactiver).
    - `runLog` : purge de `cron/runs/<jobId>.jsonl` par taille et lignes conservées.
    - Voir [Tâches cron](/automation/cron-jobs) pour l'aperçu des fonctionnalités et les exemples CLI.

  </Accordion>

  <Accordion title="Configurer les webhooks (hooks)">
    Activez les points d'accès webhook HTTP sur le Gateway :

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    Voir la [référence complète](/gateway/configuration-reference#hooks) pour toutes les options de mapping et l'intégration Gmail.

  </Accordion>

  <Accordion title="Configurer le routage multi-agent">
    Exécutez plusieurs agents isolés avec des espaces de travail et sessions séparés :

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    Voir [Multi-Agent](/concepts/multi-agent) et la [référence complète](/gateway/configuration-reference#multi-agent-routing) pour les règles de liaison et les profils d'accès par agent.

  </Accordion>

  <Accordion title="Diviser la configuration en plusieurs fichiers ($include)">
    Utilisez `$include` pour organiser les configurations volumineuses :

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **Fichier unique** : remplace l'objet contenant
    - **Tableau de fichiers** : fusion profonde dans l'ordre (le dernier l'emporte)
    - **Clés voisines** : fusionnées après les inclusions (remplacent les valeurs incluses)
    - **Inclusions imbriquées** : prises en charge jusqu'à 10 niveaux de profondeur
    - **Chemins relatifs** : résolus relativement au fichier incluant
    - **Gestion des erreurs** : messages clairs pour les fichiers manquants, erreurs de parsing et inclusions circulaires

  </Accordion>
</AccordionGroup>

## Rechargement à chaud de la configuration

Le Gateway surveille `~/.openclaw/openclaw.json` et applique les modifications automatiquement, sans redémarrage manuel nécessaire pour la plupart des paramètres.

### Modes de rechargement

| Mode                         | Comportement                                                                                    |
| ---------------------------- | ----------------------------------------------------------------------------------------------- |
| **`hybrid`** (par défaut)    | Applique à chaud les modifications sûres instantanément. Redémarre automatiquement pour les critiques. |
| **`hot`**                    | Applique à chaud les modifications sûres uniquement. Journalise un avertissement quand un redémarrage est nécessaire. |
| **`restart`**                | Redémarre le Gateway à chaque modification de configuration, sûre ou non.                       |
| **`off`**                    | Désactive la surveillance du fichier. Les modifications prennent effet au prochain redémarrage manuel. |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Ce qui s'applique à chaud vs ce qui nécessite un redémarrage

La plupart des champs s'appliquent à chaud sans interruption. En mode `hybrid`, les modifications nécessitant un redémarrage sont gérées automatiquement.

| Catégorie           | Champs                                                               | Redémarrage nécessaire ? |
| ------------------- | -------------------------------------------------------------------- | ------------------------ |
| Canaux              | `channels.*`, `web` (WhatsApp) — tous les canaux intégrés et extensions | Non                      |
| Agent & modèles     | `agent`, `agents`, `models`, `routing`                               | Non                      |
| Automatisation      | `hooks`, `cron`, `agent.heartbeat`                                   | Non                      |
| Sessions & messages | `session`, `messages`                                                | Non                      |
| Outils & médias     | `tools`, `browser`, `skills`, `audio`, `talk`                        | Non                      |
| UI & divers         | `ui`, `logging`, `identity`, `bindings`                              | Non                      |
| Serveur Gateway     | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                 | **Oui**                  |
| Infrastructure      | `discovery`, `canvasHost`, `plugins`                                 | **Oui**                  |

<Note>
`gateway.reload` et `gateway.remote` sont des exceptions : les modifier ne déclenche **pas** de redémarrage.
</Note>

## RPC de configuration (mises à jour programmatiques)

<Note>
Les RPC d'écriture du plan de contrôle (`config.apply`, `config.patch`, `update.run`) sont limités en débit à **3 requêtes par 60 secondes** par `deviceId+clientIp`. En cas de limitation, le RPC renvoie `UNAVAILABLE` avec `retryAfterMs`.
</Note>

<AccordionGroup>
  <Accordion title="config.apply (remplacement complet)">
    Valide + écrit la configuration complète et redémarre le Gateway en une seule étape.

    <Warning>
    `config.apply` remplace la **configuration entière**. Utilisez `config.patch` pour les mises à jour partielles, ou `openclaw config set` pour les clés individuelles.
    </Warning>

    Paramètres :

    - `raw` (chaîne) — charge utile JSON5 pour la configuration entière
    - `baseHash` (optionnel) — hash de configuration de `config.get` (requis quand une configuration existe)
    - `sessionKey` (optionnel) — clé de session pour le ping de réveil post-redémarrage
    - `note` (optionnel) — note pour la sentinelle de redémarrage
    - `restartDelayMs` (optionnel) — délai avant le redémarrage (défaut 2000)

    Les demandes de redémarrage sont fusionnées lorsqu'une est déjà en attente/en cours, et un temps de refroidissement de 30 secondes s'applique entre les cycles de redémarrage.

    ```bash
    openclaw gateway call config.get --params '{}'  # capturer payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```

  </Accordion>

  <Accordion title="config.patch (mise à jour partielle)">
    Fusionne une mise à jour partielle dans la configuration existante (sémantique JSON merge patch) :

    - Les objets fusionnent récursivement
    - `null` supprime une clé
    - Les tableaux remplacent

    Paramètres :

    - `raw` (chaîne) — JSON5 avec seulement les clés à modifier
    - `baseHash` (requis) — hash de configuration de `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — identiques à `config.apply`

    Le comportement de redémarrage est identique à `config.apply` : redémarrages en attente fusionnés plus un temps de refroidissement de 30 secondes entre les cycles.

    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```

  </Accordion>
</AccordionGroup>

## Variables d'environnement

OpenClaw lit les variables d'environnement du processus parent plus :

- `.env` du répertoire de travail actuel (si présent)
- `~/.openclaw/.env` (repli global)

Aucun de ces fichiers ne remplace les variables d'environnement existantes. Vous pouvez aussi définir des variables d'environnement en ligne dans la configuration :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Import d'environnement shell (optionnel)">
  Si activé et que les clés attendues ne sont pas définies, OpenClaw lance votre shell de connexion et importe uniquement les clés manquantes :

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Équivalent variable d'environnement : `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Substitution de variables d'environnement dans les valeurs de configuration">
  Référencez les variables d'environnement dans toute valeur de chaîne de configuration avec `${VAR_NAME}` :

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Règles :

- Seuls les noms en majuscules sont reconnus : `[A-Z_][A-Z0-9_]*`
- Les variables manquantes/vides génèrent une erreur au chargement
- Échappez avec `$${VAR}` pour un résultat littéral
- Fonctionne dans les fichiers `$include`
- Substitution en ligne : `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

<Accordion title="Secret refs (env, file, exec)">
  Pour les champs supportant les objets SecretRef, vous pouvez utiliser :

```json5
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

Les details SecretRef (y compris `secrets.providers` pour `env`/`file`/`exec`) sont dans [Gestion des secrets](/gateway/secrets).
</Accordion>

Voir [Environnement](/help/environment) pour la précédence et les sources complètes.

## Référence complète

Pour la référence champ par champ complète, voir **[Référence de configuration](/gateway/configuration-reference)**.

---

_Liens connexes : [Exemples de configuration](/gateway/configuration-examples) · [Référence de configuration](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_
