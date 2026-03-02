---
summary: "Plugins/extensions OpenClaw : découverte, configuration et sécurité"
read_when:
  - Ajout ou modification de plugins/extensions
  - Documentation des règles d'installation ou de chargement des plugins
title: "Plugins"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/plugin.md
  workflow: manual
---

# Plugins (Extensions)

## Démarrage rapide (nouveau avec les plugins ?)

Un plugin est simplement un **petit module de code** qui étend OpenClaw avec des fonctionnalités supplémentaires (commandes, outils et RPC Gateway).

La plupart du temps, vous utiliserez des plugins quand vous voulez une fonctionnalité qui n'est pas encore intégrée dans le noyau d'OpenClaw (ou que vous voulez garder les fonctionnalités optionnelles hors de votre installation principale).

Chemin rapide :

1. Voir ce qui est déjà chargé :

```bash
openclaw plugins list
```

2. Installer un plugin officiel (exemple : Voice Call) :

```bash
openclaw plugins install @openclaw/voice-call
```

Les spécifications npm sont **registre uniquement** (nom de package + version/tag optionnel). Les spécifications Git/URL/fichier sont rejetées.

3. Redémarrez le Gateway, puis configurez sous `plugins.entries.<id>.config`.

Voir [Voice Call](/plugins/voice-call) pour un exemple concret de plugin.
Vous cherchez des listes tierces ? Voir [Plugins communautaires](/plugins/community).

## Plugins disponibles (officiels)

- Microsoft Teams est en mode plugin uniquement depuis la version 2026.1.15 ; installez `@openclaw/msteams` si vous utilisez Teams.
- Memory (Core) — plugin de recherche mémoire fourni (activé par défaut via `plugins.slots.memory`)
- Memory (LanceDB) — plugin de mémoire long terme fourni (rappel/capture automatique ; définissez `plugins.slots.memory = "memory-lancedb"`)
- [Voice Call](/plugins/voice-call) — `@openclaw/voice-call`
- [Zalo Personal](/plugins/zalouser) — `@openclaw/zalouser`
- [Matrix](/channels/matrix) — `@openclaw/matrix`
- [Nostr](/channels/nostr) — `@openclaw/nostr`
- [Zalo](/channels/zalo) — `@openclaw/zalo`
- [Microsoft Teams](/channels/msteams) — `@openclaw/msteams`
- Google Antigravity OAuth (auth fournisseur) — fourni sous `google-antigravity-auth` (désactivé par défaut)
- Gemini CLI OAuth (auth fournisseur) — fourni sous `google-gemini-cli-auth` (désactivé par défaut)
- Qwen OAuth (auth fournisseur) — fourni sous `qwen-portal-auth` (désactivé par défaut)
- Copilot Proxy (auth fournisseur) — pont local VS Code Copilot Proxy ; distinct du login par appareil `github-copilot` intégré (fourni, désactivé par défaut)

Les plugins OpenClaw sont des **modules TypeScript** chargés à l'exécution via jiti. **La validation de la configuration n'exécute pas le code du plugin** ; elle utilise le manifeste du plugin et le JSON Schema à la place. Voir [Manifeste du plugin](/plugins/manifest).

Les plugins peuvent enregistrer :

- Des méthodes RPC Gateway
- Des handlers HTTP Gateway
- Des outils pour l'agent
- Des commandes CLI
- Des services en arrière-plan
- Une validation de configuration optionnelle
- Des **Skills** (en listant des répertoires `skills` dans le manifeste du plugin)
- Des **commandes auto-reply** (s'exécutent sans invoquer l'agent IA)

Les plugins s'exécutent **dans le même processus** que le Gateway, donc traitez-les comme du code de confiance.
Guide d'authoring d'outils : [Outils d'agent plugin](/plugins/agent-tools).

## Helpers de runtime

Les plugins peuvent accéder à des helpers core sélectionnés via `api.runtime`. Pour le TTS téléphonie :

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes :

- Utilisé la configuration core `messages.tts` (OpenAI ou ElevenLabs).
- Retourne un buffer audio PCM + taux d'échantillonnage. Les plugins doivent rééchantillonner/encoder pour les fournisseurs.
- Edge TTS n'est pas supporté pour la téléphonie.

## Découverte & préséance

OpenClaw scanne, dans l'ordre :

1. Chemins de configuration

- `plugins.load.paths` (fichier ou répertoire)

2. Extensions de l'espace de travail

- `<workspace>/.openclaw/extensions/*.ts`
- `<workspace>/.openclaw/extensions/*/index.ts`

3. Extensions globales

- `~/.openclaw/extensions/*.ts`
- `~/.openclaw/extensions/*/index.ts`

4. Extensions fournies (livrées avec OpenClaw, **désactivées par défaut**)

- `<openclaw>/extensions/*`

Les plugins fournis doivent être activés explicitement via `plugins.entries.<id>.enabled` ou `openclaw plugins enable <id>`. Les plugins installés sont activés par défaut, mais peuvent être désactivés de la même manière.

Notes de renforcement :

- Si `plugins.allow` est vide et que des plugins non fournis sont découvrables, OpenClaw journalise un avertissement au démarrage avec les ids et sources des plugins.
- Les chemins candidats sont vérifiés en sécurité avant l'admission à la découverte. OpenClaw bloque les candidats quand :
  - l'entrée de l'extension se résout en dehors de la racine du plugin (y compris les échappements par symlink/traversée de chemin),
  - la racine/chemin source du plugin est accessible en écriture par tous,
  - la propriété du chemin est suspecte pour les plugins non fournis (le propriétaire POSIX n'est ni l'uid courant ni root).
- Les plugins non fournis chargés sans provenance d'installation/chemin-de-chargement émettent un avertissement pour que vous puissiez épingler la confiance (`plugins.allow`) ou le suivi d'installation (`plugins.installs`).

Chaque plugin doit inclure un fichier `openclaw.plugin.json` dans sa racine. Si un chemin pointe vers un fichier, la racine du plugin est le répertoire du fichier et doit contenir le manifeste.

Si plusieurs plugins se résolvent au même id, la première correspondance dans l'ordre ci-dessus l'emporte et les copies de moindre préséance sont ignorées.

### Packs de packages

Un répertoire de plugin peut inclure un `package.json` avec `openclaw.extensions` :

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Chaque entrée devient un plugin. Si le pack liste plusieurs extensions, l'id du plugin devient `name/<fileBase>`.

Si votre plugin importe des dépendances npm, installez-les dans ce répertoire pour que `node_modules` soit disponible (`npm install` / `pnpm install`).

Garde-fou de sécurité : chaque entrée `openclaw.extensions` doit rester à l'intérieur du répertoire du plugin après résolution des symlinks. Les entrées qui échappent au répertoire du package sont rejetées.

Note de sécurité : `openclaw plugins install` installe les dépendances du plugin avec `npm install --ignore-scripts` (pas de scripts de cycle de vie). Gardez les arbres de dépendances des plugins en « JS/TS pur » et évitez les packages qui nécessitent des builds `postinstall`.

### Métadonnées de catalogue de canaux

Les plugins de canaux peuvent annoncer des métadonnées d'onboarding via `openclaw.channel` et des indices d'installation via `openclaw.install`. Cela garde le catalogue core libre de données.

Exemple :

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Self-hosted chat via Nextcloud Talk webhook bots.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw peut aussi fusionner des **catalogues de canaux externes** (par exemple, un export de registre MPM). Déposez un fichier JSON à l'un des emplacements :

- `~/.openclaw/mpm/plugins.json`
- `~/.openclaw/mpm/catalog.json`
- `~/.openclaw/plugins/catalog.json`

Ou pointez `OPENCLAW_PLUGIN_CATALOG_PATHS` (ou `OPENCLAW_MPM_CATALOG_PATHS`) vers un ou plusieurs fichiers JSON (délimités par virgule/point-virgule/`PATH`). Chaque fichier doit contenir `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

## IDs de plugins

IDs de plugins par défaut :

- Packs de packages : `name` dans `package.json`
- Fichier autonome : nom de base du fichier (`~/.../voice-call.ts` → `voice-call`)

Si un plugin exporte `id`, OpenClaw l'utilisé mais avertit quand il ne correspond pas à l'id configuré.

## Configuration

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

Champs :

- `enabled` : bascule principale (défaut : true)
- `allow` : liste d'autorisation (optionnelle)
- `deny` : liste de refus (optionnelle ; le refus l'emporte)
- `load.paths` : fichiers/répertoires de plugins supplémentaires
- `entries.<id>` : bascules par plugin + configuration

Les changements de configuration **nécessitent un redémarrage du gateway**.

Règles de validation (strictes) :

- Les ids de plugins inconnus dans `entries`, `allow`, `deny` ou `slots` sont des **erreurs**.
- Les clés `channels.<id>` inconnues sont des **erreurs** sauf si un manifeste de plugin déclare l'id du canal.
- La configuration du plugin est validée en utilisant le JSON Schema intégré dans `openclaw.plugin.json` (`configSchema`).
- Si un plugin est désactivé, sa configuration est préservée et un **avertissement** est émis.

## Slots de plugins (catégories exclusives)

Certaines catégories de plugins sont **exclusives** (une seule activé à la fois). Utilisez `plugins.slots` pour sélectionner quel plugin possède le slot :

```json5
{
  plugins: {
    slots: {
      memory: "memory-core", // ou "none" pour désactiver les plugins mémoire
    },
  },
}
```

Si plusieurs plugins déclarent `kind: "memory"`, seul celui sélectionné se charge. Les autres sont désactivés avec des diagnostics.

## Interface de contrôle (schema + labels)

L'interface de contrôle utilisé `config.schema` (JSON Schema + `uiHints`) pour rendre de meilleurs formulaires.

OpenClaw enrichit `uiHints` à l'exécution en fonction des plugins découverts :

- Ajouté des labels par plugin pour `plugins.entries.<id>` / `.enabled` / `.config`
- Fusionne les indices optionnels de champs de configuration fournis par le plugin sous :
  `plugins.entries.<id>.config.<field>`

Si vous voulez que les champs de configuration de votre plugin affichent de bons labels/placeholders (et marquent les secrets comme sensibles), fournissez `uiHints` avec votre JSON Schema dans le manifeste du plugin.

Exemple :

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "API Key", "sensitive": true },
    "region": { "label": "Region", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copier un fichier/répertoire local dans ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # chemin relatif ok
openclaw plugins install ./plugin.tgz           # installer depuis un tarball local
openclaw plugins install ./plugin.zip           # installer depuis un zip local
openclaw plugins install -l ./extensions/voice-call # lien (pas de copie) pour le dev
openclaw plugins install @openclaw/voice-call # installer depuis npm
openclaw plugins install @openclaw/voice-call --pin # stocker le name@version résolu exact
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` ne fonctionne que pour les installations npm suivies sous `plugins.installs`.
Si les métadonnées d'intégrité stockées changent entre les mises à jour, OpenClaw avertit et demande confirmation (utilisez le global `--yes` pour contourner les invites).

Les plugins peuvent aussi enregistrer leurs propres commandes de niveau supérieur (exemple : `openclaw voicecall`).

## API Plugin (aperçu)

Les plugins exportent soit :

- Une fonction : `(api) => { ... }`
- Un objet : `{ id, name, configSchema, register(api) { ... } }`

## Hooks de plugins

Les plugins peuvent enregistrer des hooks à l'exécution. Cela permet à un plugin d'embarquer de l'automatisation pilotée par événements sans installation séparée de pack de hooks.

### Exemple

```ts
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // Logique du hook ici.
    },
    {
      name: "my-plugin.command-new",
      description: "Runs when /new is invoked",
    },
  );
}
```

Notes :

- Enregistrez les hooks explicitement via `api.registerHook(...)`.
- Les règles d'éligibilité des hooks s'appliquent toujours (exigences OS/bins/env/config).
- Les hooks gérés par les plugins apparaissent dans `openclaw hooks list` avec `plugin:<id>`.
- Vous ne pouvez pas activer/désactiver les hooks gérés par les plugins via `openclaw hooks` ; activez/désactivez le plugin à la place.

## Plugins fournisseur (auth de modèle)

Les plugins peuvent enregistrer des flux **d'auth de fournisseur de modèle** pour que les utilisateurs puissent exécuter OAuth ou la configuration de clé API dans OpenClaw (pas de scripts externes nécessaires).

Enregistrez un fournisseur via `api.registerProvider(...)`. Chaque fournisseur expose une ou plusieurs méthodes d'auth (OAuth, clé API, code d'appareil, etc.). Ces méthodes alimentent :

- `openclaw models auth login --provider <id> [--method <id>]`

Exemple :

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Exécuter le flux OAuth et retourner les profils d'auth.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notes :

- `run` reçoit un `ProviderAuthContext` avec des helpers `prompter`, `runtime`, `openUrl` et `oauth.createVpsAwareHandlers`.
- Retournez `configPatch` quand vous avez besoin d'ajouter des modèles par défaut ou de la configuration fournisseur.
- Retournez `defaultModel` pour que `--set-default` puisse mettre à jour les défauts de l'agent.

### Enregistrer un canal de messagerie

Les plugins peuvent enregistrer des **plugins de canal** qui se comportent comme les canaux intégrés (WhatsApp, Telegram, etc.). La configuration du canal se trouve sous `channels.<id>` et est validée par votre code de plugin de canal.

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notes :

- Mettez la configuration sous `channels.<id>` (pas `plugins.entries`).
- `meta.label` est utilisé pour les labels dans les listes CLI/UI.
- `meta.aliases` ajouté des ids alternatifs pour la normalisation et les entrées CLI.
- `meta.preferOver` liste les ids de canaux à ignorer pour l'auto-activation quand les deux sont configurés.
- `meta.detailLabel` et `meta.systemImage` permettent aux interfaces d'afficher des labels/icônes de canaux plus riches.

### Hooks d'onboarding de canal

Les plugins de canal peuvent définir des hooks d'onboarding optionnels sur `plugin.onboarding` :

- `configuré(ctx)` est le flux de configuration de base.
- `configureInteractive(ctx)` peut prendre entièrement en charge la configuration interactive pour les états configurés et non configurés.
- `configureWhenConfigured(ctx)` peut remplacer le comportement uniquement pour les canaux déjà configurés.

Préséance des hooks dans l'assistant :

1. `configureInteractive` (si présent)
2. `configureWhenConfigured` (uniquement quand le statut du canal est déjà configuré)
3. repli sur `configuré`

Détails du contexte :

- `configureInteractive` et `configureWhenConfigured` reçoivent :
  - `configured` (`true` ou `false`)
  - `label` (nom du canal visible par l'utilisateur utilisé par les invites)
  - plus les champs partagés config/runtime/prompter/options
- Retourner `"skip"` laisse la sélection et le suivi de compte inchangés.
- Retourner `{ cfg, accountId? }` applique les mises à jour de configuration et enregistre la sélection de compte.

### Écrire un nouveau canal de messagerie (étape par étape)

Utilisez ceci quand vous voulez une **nouvelle surface de chat** (un « canal de messagerie »), pas un fournisseur de modèle. La documentation des fournisseurs de modèle se trouve sous `/providers/*`.

1. Choisir un id + forme de configuration

- Toute la configuration de canal se trouve sous `channels.<id>`.
- Préférez `channels.<id>.accounts.<accountId>` pour les configurations multi-comptes.

2. Définir les métadonnées du canal

- `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` contrôlent les listes CLI/UI.
- `meta.docsPath` doit pointer vers une page de documentation comme `/channels/<id>`.
- `meta.preferOver` permet à un plugin de remplacer un autre canal (l'auto-activation le préfère).
- `meta.detailLabel` et `meta.systemImage` sont utilisés par les interfaces pour le texte de détail/icônes.

3. Implémenter les adaptateurs requis

- `config.listAccountIds` + `config.resolveAccount`
- `capabilities` (types de chat, médias, threads, etc.)
- `outbound.deliveryMode` + `outbound.sendText` (pour l'envoi basique)

4. Ajouter les adaptateurs optionnels selon les besoins

- `setup` (assistant), `security` (politique DM), `status` (santé/diagnostics)
- `gateway` (démarrer/arrêter/login), `mentions`, `threading`, `streaming`
- `actions` (actions sur messages), `commands` (comportement de commandes natif)

5. Enregistrer le canal dans votre plugin

- `api.registerChannel({ plugin })`

Exemple de configuration minimale :

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true },
      },
    },
  },
}
```

Plugin de canal minimal (sortant uniquement) :

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // livrer `text` à votre canal ici
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Chargez le plugin (répertoire d'extensions ou `plugins.load.paths`), redémarrez le gateway, puis configurez `channels.<id>` dans votre configuration.

### Outils pour l'agent

Voir le guide dédié : [Outils d'agent plugin](/plugins/agent-tools).

### Enregistrer une méthode RPC Gateway

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

### Enregistrer des commandes CLI

```ts
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### Enregistrer des commandes auto-reply

Les plugins peuvent enregistrer des commandes slash personnalisées qui s'exécutent **sans invoquer l'agent IA**. C'est utile pour les commandes de bascule, les vérifications de statut ou les actions rapides qui n'ont pas besoin de traitement LLM.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

Contexte du handler de commande :

- `senderId` : L'ID de l'expéditeur (si disponible)
- `channel` : Le canal où la commande a été envoyée
- `isAuthorizedSender` : Si l'expéditeur est un utilisateur autorisé
- `args` : Arguments passés après la commande (si `acceptsArgs: true`)
- `commandBody` : Le texte complet de la commande
- `config` : La configuration OpenClaw courante

Options de commande :

- `name` : Nom de la commande (sans le `/` initial)
- `description` : Texte d'aide affiché dans les listes de commandes
- `acceptsArgs` : Si la commande accepte des arguments (défaut : false). Si false et que des arguments sont fournis, la commande ne correspondra pas et le message passera aux autres handlers
- `requireAuth` : Si un expéditeur autorisé est requis (défaut : true)
- `handler` : Fonction qui retourne `{ text: string }` (peut être async)

Exemple avec autorisation et arguments :

```ts
api.registerCommand({
  name: "setmode",
  description: "Set plugin mode",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Notes :

- Les commandes de plugins sont traitées **avant** les commandes intégrées et l'agent IA
- Les commandes sont enregistrées globalement et fonctionnent sur tous les canaux
- Les noms de commandes sont insensibles à la casse (`/MyStatus` correspond à `/mystatus`)
- Les noms de commandes doivent commencer par une lettre et ne contenir que des lettres, chiffrés, tirets et underscores
- Les noms de commandes réservés (comme `help`, `status`, `reset`, etc.) ne peuvent pas être remplacés par des plugins
- L'enregistrement de commandes en double entre plugins échouera avec une erreur de diagnostic

### Enregistrer des services en arrière-plan

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

## Conventions de nommage

- Méthodes Gateway : `pluginId.action` (exemple : `voicecall.status`)
- Outils : `snake_case` (exemple : `voice_call`)
- Commandes CLI : kebab ou camel, mais évitez les conflits avec les commandes core

## Skills

Les plugins peuvent embarquer un skill dans le dépôt (`skills/<name>/SKILL.md`).
Activez-le avec `plugins.entries.<id>.enabled` (ou d'autres portes de configuration) et assurez-vous qu'il est présent dans vos emplacements de Skills workspace/managed.

## Distribution (npm)

Packaging recommandé :

- Package principal : `openclaw` (ce dépôt)
- Plugins : packages npm séparés sous `@openclaw/*` (exemple : `@openclaw/voice-call`)

Contrat de publication :

- Le `package.json` du plugin doit inclure `openclaw.extensions` avec un ou plusieurs fichiers d'entrée.
- Les fichiers d'entrée peuvent être `.js` ou `.ts` (jiti charge le TS à l'exécution).
- `openclaw plugins install <npm-spec>` utilisé `npm pack`, extrait dans `~/.openclaw/extensions/<id>/`, et l'activé dans la configuration.
- Stabilité des clés de configuration : les packages scopés sont normalisés vers l'id **non scopé** pour `plugins.entries.*`.

## Exemple de plugin : Voice Call

Ce dépôt inclut un plugin d'appel vocal (Twilio ou fallback log) :

- Source : `extensions/voice-call`
- Skill : `skills/voice-call`
- CLI : `openclaw voicecall start|status`
- Outil : `voice_call`
- RPC : `voicecall.start`, `voicecall.status`
- Configuration (twilio) : `provider: "twilio"` + `twilio.accountSid/authToken/from` (optionnel `statusCallbackUrl`, `twimlUrl`)
- Configuration (dev) : `provider: "log"` (pas de réseau)

Voir [Voice Call](/plugins/voice-call) et `extensions/voice-call/README.md` pour l'installation et l'utilisation.

## Notes de sécurité

Les plugins s'exécutent dans le même processus que le Gateway. Traitez-les comme du code de confiance :

- N'installez que des plugins en lesquels vous avez confiance.
- Préférez les listes d'autorisation `plugins.allow`.
- Redémarrez le Gateway après les changements.

## Tester les plugins

Les plugins peuvent (et devraient) embarquer des tests :

- Les plugins dans le dépôt peuvent garder des tests Vitest sous `src/**` (exemple : `src/plugins/voice-call.plugin.test.ts`).
- Les plugins publiés séparément devraient exécuter leur propre CI (lint/build/test) et valider que `openclaw.extensions` pointe vers le point d'entrée compilé (`dist/index.js`).
