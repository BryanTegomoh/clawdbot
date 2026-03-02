---
summary: "Hooks : automatisation événementielle pour commandes et événements de cycle de vie"
read_when:
  - Vous voulez une automatisation événementielle pour /new, /reset, /stop et les événements du cycle de vie de l'agent
  - Vous voulez créer, installer ou déboguer des hooks
title: "Hooks"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/hooks.md
  workflow: manual
---

# Hooks

Les hooks fournissent un système événementiel extensible pour automatiser des actions en réponse aux commandes et événements de l'agent. Les hooks sont automatiquement découverts depuis des répertoires et peuvent être gérés via des commandes CLI, de manière similaire au fonctionnement des Skills dans OpenClaw.

## Pour commencer

Les hooks sont de petits scripts qui s'exécutent quand quelque chose se produit. Il en existe deux types :

- **Hooks** (cette page) : s'exécutent à l'intérieur du Gateway quand des événements de l'agent se déclenchent, comme `/new`, `/reset`, `/stop`, ou des événements de cycle de vie.
- **Webhooks** : webhooks HTTP externes qui permettent à d'autres systèmes de déclencher du travail dans OpenClaw. Voir [Webhook Hooks](/automation/webhook) ou utilisez `openclaw webhooks` pour les commandes d'aide Gmail.

Les hooks peuvent aussi être regroupés dans des plugins ; voir [Plugins](/tools/plugin#plugin-hooks).

Utilisations courantes :

- Sauvegarder un instantané mémoire quand vous réinitialisez une session
- Maintenir une piste d'audit des commandes pour le dépannage ou la conformité
- Déclencher une automatisation de suivi quand une session démarre ou se termine
- Écrire des fichiers dans l'espace de travail de l'agent ou appeler des APIs externes quand des événements se déclenchent

Si vous savez écrire une petite fonction TypeScript, vous savez écrire un hook. Les hooks sont découverts automatiquement, et vous les activez ou désactivez via le CLI.

## Vue d'ensemble

Le système de hooks vous permet de :

- Sauvegarder le contexte de session en mémoire quand `/new` est émis
- Journaliser toutes les commandes pour l'audit
- Déclencher des automatisations personnalisées sur les événements du cycle de vie de l'agent
- Étendre le comportement d'OpenClaw sans modifier le code source

## Premiers pas

### Hooks intégrés

OpenClaw est livré avec quatre hooks intégrés qui sont automatiquement découverts :

- **session-memory** : sauvegarde le contexte de session dans votre espace de travail d'agent (par défaut `~/.openclaw/workspace/memory/`) quand vous émettez `/new`
- **bootstrap-extra-files** : injecte des fichiers bootstrap supplémentaires de l'espace de travail depuis des patterns glob/chemin configurés pendant `agent:bootstrap`
- **command-logger** : journalise tous les événements de commande dans `~/.openclaw/logs/commands.log`
- **boot-md** : exécute `BOOT.md` quand le Gateway démarre (nécessite les hooks internes activés)

Lister les hooks disponibles :

```bash
openclaw hooks list
```

Activer un hook :

```bash
openclaw hooks enable session-memory
```

Vérifier le statut d'un hook :

```bash
openclaw hooks check
```

Obtenir des informations détaillées :

```bash
openclaw hooks info session-memory
```

### Configuration initiale

Pendant la configuration initiale (`openclaw onboard`), vous serez invité à activer les hooks recommandés. L'assistant découvre automatiquement les hooks éligibles et les présente pour sélection.

## Découverte des hooks

Les hooks sont automatiquement découverts depuis trois répertoires (par ordre de priorité) :

1. **Hooks d'espace de travail** : `<workspace>/hooks/` (par agent, priorité la plus haute)
2. **Hooks gérés** : `~/.openclaw/hooks/` (installés par l'utilisateur, partagés entre espaces de travail)
3. **Hooks intégrés** : `<openclaw>/dist/hooks/bundled/` (livrés avec OpenClaw)

Les répertoires de hooks gérés peuvent être soit un **hook unique** soit un **pack de hooks** (répertoire package).

Chaque hook est un répertoire contenant :

```
my-hook/
├── HOOK.md          # Métadonnées + documentation
└── handler.ts       # Implémentation du gestionnaire
```

## Packs de hooks (npm/archives)

Les packs de hooks sont des packages npm standard qui exportent un ou plusieurs hooks via `openclaw.hooks` dans
`package.json`. Installez-les avec :

```bash
openclaw hooks install <path-or-spec>
```

Les spécifications npm sont uniquement via le registre (nom de package + version/tag optionnel). Les spécifications Git/URL/fichier sont rejetées.

Exemple `package.json` :

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Chaque entrée pointe vers un répertoire de hook contenant `HOOK.md` et `handler.ts` (ou `index.ts`).
Les packs de hooks peuvent embarquer des dépendances ; elles seront installées sous `~/.openclaw/hooks/<id>`.
Chaque entrée `openclaw.hooks` doit rester à l'intérieur du répertoire du package après
résolution des liens symboliques ; les entrées qui s'en échappent sont rejetées.

Note de sécurité : `openclaw hooks install` installe les dépendances avec `npm install --ignore-scripts`
(pas de scripts de cycle de vie). Gardez les arbres de dépendances des packs de hooks en "pur JS/TS" et évitez les packages qui dépendent
de builds `postinstall`.

## Structure d'un hook

### Format HOOK.md

Le fichier `HOOK.md` contient des métadonnées en frontmatter YAML plus de la documentation Markdown :

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### Champs de métadonnées

L'objet `metadata.openclaw` supporte :

- **`emoji`** : emoji d'affichage pour le CLI (par ex., `"💾"`)
- **`events`** : tableau d'événements à écouter (par ex., `["command:new", "command:reset"]`)
- **`export`** : export nommé à utiliser (par défaut `"default"`)
- **`homepage`** : URL de documentation
- **`requires`** : exigences optionnelles
  - **`bins`** : binaires requis dans le PATH (par ex., `["git", "node"]`)
  - **`anyBins`** : au moins un de ces binaires doit être présent
  - **`env`** : variables d'environnement requises
  - **`config`** : chemins de configuration requis (par ex., `["workspace.dir"]`)
  - **`os`** : plateformes requises (par ex., `["darwin", "linux"]`)
- **`always`** : contourner les vérifications d'éligibilité (booléen)
- **`install`** : méthodes d'installation (pour les hooks intégrés : `[{"id":"bundled","kind":"bundled"}]`)

### Implémentation du gestionnaire

Le fichier `handler.ts` exporte une fonction `HookHandler` :

```typescript
const myHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### Contexte d'événement

Chaque événement inclut :

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // par ex., 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // Identifiant de session
  timestamp: Date,             // Quand l'événement s'est produit
  messages: string[],          // Ajoutez des messages ici pour les envoyer à l'utilisateur
  context: {
    // Événements de commande :
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // par ex., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // Événements de message (voir la section Événements de message pour tous les détails) :
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## Types d'événements

### Événements de commande

Déclenchés quand des commandes d'agent sont émises :

- **`command`** : tous les événements de commande (écouteur général)
- **`command:new`** : quand la commande `/new` est émise
- **`command:reset`** : quand la commande `/reset` est émise
- **`command:stop`** : quand la commande `/stop` est émise

### Événements d'agent

- **`agent:bootstrap`** : avant que les fichiers bootstrap d'espace de travail soient injectés (les hooks peuvent modifier `context.bootstrapFiles`)

### Événements du Gateway

Déclenchés quand le Gateway démarre :

- **`gateway:startup`** : après le démarrage des canaux et le chargement des hooks

### Événements de message

Déclenchés quand des messages sont reçus ou envoyés :

- **`message`** : tous les événements de message (écouteur général)
- **`message:received`** : quand un message entrant est reçu depuis n'importe quel canal
- **`message:sent`** : quand un message sortant est envoyé avec succès

#### Contexte d'événement de message

Les événements de message incluent un contexte riche sur le message :

```typescript
// contexte message:received
{
  from: string,           // Identifiant de l'expéditeur (numéro de téléphone, ID utilisateur, etc.)
  content: string,        // Contenu du message
  timestamp?: number,     // Horodatage Unix de réception
  channelId: string,      // Canal (par ex., "whatsapp", "telegram", "discord")
  accountId?: string,     // ID de compte du fournisseur pour les configurations multi-comptes
  conversationId?: string, // ID de chat/conversation
  messageId?: string,     // ID de message du fournisseur
  metadata?: {            // Données supplémentaires spécifiques au fournisseur
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// contexte message:sent
{
  to: string,             // Identifiant du destinataire
  content: string,        // Contenu du message envoyé
  success: boolean,       // Si l'envoi a réussi
  error?: string,         // Message d'erreur si l'envoi a échoué
  channelId: string,      // Canal (par ex., "whatsapp", "telegram", "discord")
  accountId?: string,     // ID de compte du fournisseur
  conversationId?: string, // ID de chat/conversation
  messageId?: string,     // ID de message retourné par le fournisseur
}
```

#### Exemple : hook de journalisation de messages

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Received from ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Sent to ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### Hooks de résultat d'outil (API Plugin)

Ces hooks ne sont pas des écouteurs de flux d'événements ; ils permettent aux plugins d'ajuster de manière synchrone les résultats d'outil avant qu'OpenClaw ne les persiste.

- **`tool_result_persist`** : transformer les résultats d'outil avant qu'ils soient écrits dans la transcription de session. Doit être synchrone ; retournez la charge utile de résultat d'outil mise à jour ou `undefined` pour la garder telle quelle. Voir [Boucle d'agent](/concepts/agent-loop).

### Événements futurs

Types d'événements planifiés :

- **`session:start`** : quand une nouvelle session commence
- **`session:end`** : quand une session se termine
- **`agent:error`** : quand un agent rencontre une erreur

## Créer des hooks personnalisés

### 1. Choisir l'emplacement

- **Hooks d'espace de travail** (`<workspace>/hooks/`) : par agent, priorité la plus haute
- **Hooks gérés** (`~/.openclaw/hooks/`) : partagés entre espaces de travail

### 2. Créer la structure de répertoire

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. Créer HOOK.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. Créer handler.ts

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### 5. Activer et tester

```bash
# Vérifier que le hook est découvert
openclaw hooks list

# L'activer
openclaw hooks enable my-hook

# Redémarrer votre processus Gateway (redémarrage de l'app barre de menu sur macOS, ou redémarrage de votre processus de dev)

# Déclencher l'événement
# Envoyez /new via votre canal de messagerie
```

## Configuration

### Nouveau format de configuration (recommandé)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Configuration par hook

Les hooks peuvent avoir une configuration personnalisée :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Répertoires supplémentaires

Charger des hooks depuis des répertoires supplémentaires :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Ancien format de configuration (toujours supporté)

L'ancien format de configuration fonctionne toujours pour la rétrocompatibilité :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

Note : `module` doit être un chemin relatif à l'espace de travail. Les chemins absolus et la traversée hors de l'espace de travail sont rejetés.

**Migration** : utilisez le nouveau système basé sur la découverte pour les nouveaux hooks. Les gestionnaires hérités sont chargés après les hooks basés sur les répertoires.

## Commandes CLI

### Lister les hooks

```bash
# Lister tous les hooks
openclaw hooks list

# Afficher uniquement les hooks éligibles
openclaw hooks list --eligible

# Sortie détaillée (afficher les exigences manquantes)
openclaw hooks list --verbose

# Sortie JSON
openclaw hooks list --json
```

### Informations sur un hook

```bash
# Afficher les informations détaillées d'un hook
openclaw hooks info session-memory

# Sortie JSON
openclaw hooks info session-memory --json
```

### Vérifier l'éligibilité

```bash
# Afficher le résumé d'éligibilité
openclaw hooks check

# Sortie JSON
openclaw hooks check --json
```

### Activer/Désactiver

```bash
# Activer un hook
openclaw hooks enable session-memory

# Désactiver un hook
openclaw hooks disable command-logger
```

## Référence des hooks intégrés

### session-memory

Sauvegarde le contexte de session en mémoire quand vous émettez `/new`.

**Événements** : `command:new`

**Exigences** : `workspace.dir` doit être configuré

**Sortie** : `<workspace>/memory/YYYY-MM-DD-slug.md` (par défaut `~/.openclaw/workspace`)

**Ce qu'il fait** :

1. Utilisé l'entrée de session pré-réinitialisation pour localiser la bonne transcription
2. Extrait les 15 dernières lignes de conversation
3. Utilisé le LLM pour générer un slug de nom de fichier descriptif
4. Sauvegarde les métadonnées de session dans un fichier mémoire daté

**Exemple de sortie** :

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Exemples de noms de fichier** :

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (horodatage de repli si la génération de slug échoue)

**Activer** :

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

Injecte des fichiers bootstrap supplémentaires (par exemple des `AGENTS.md` / `TOOLS.md` locaux au monorepo) pendant `agent:bootstrap`.

**Événements** : `agent:bootstrap`

**Exigences** : `workspace.dir` doit être configuré

**Sortie** : aucun fichier écrit ; le contexte bootstrap est modifié en mémoire uniquement.

**Configuration** :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**Notes** :

- Les chemins sont résolus relativement à l'espace de travail.
- Les fichiers doivent rester à l'intérieur de l'espace de travail (vérification de chemin réel).
- Seuls les noms de base bootstrap reconnus sont chargés.
- La liste d'autorisation des sous-agents est préservée (`AGENTS.md` et `TOOLS.md` uniquement).

**Activer** :

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

Journalise tous les événements de commande dans un fichier d'audit centralisé.

**Événements** : `command`

**Exigences** : aucune

**Sortie** : `~/.openclaw/logs/commands.log`

**Ce qu'il fait** :

1. Capture les détails de l'événement (action de commande, horodatage, clé de session, ID de l'expéditeur, source)
2. Ajouté au fichier de log au format JSONL
3. S'exécute silencieusement en arrière-plan

**Exemples d'entrées de log** :

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Consulter les logs** :

```bash
# Voir les commandes récentes
tail -n 20 ~/.openclaw/logs/commands.log

# Affichage formaté avec jq
cat ~/.openclaw/logs/commands.log | jq .

# Filtrer par action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Activer** :

```bash
openclaw hooks enable command-logger
```

### boot-md

Exécute `BOOT.md` quand le Gateway démarre (après le démarrage des canaux).
Les hooks internes doivent être activés pour que cela fonctionne.

**Événements** : `gateway:startup`

**Exigences** : `workspace.dir` doit être configuré

**Ce qu'il fait** :

1. Lit `BOOT.md` depuis votre espace de travail
2. Exécute les instructions via le moteur d'agent
3. Envoie les messages sortants demandés via l'outil de message

**Activer** :

```bash
openclaw hooks enable boot-md
```

## Bonnes pratiques

### Garder les gestionnaires rapides

Les hooks s'exécutent pendant le traitement des commandes. Gardez-les légers :

```typescript
// ✓ Bien - travail asynchrone, retour immédiat
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Mauvais - bloque le traitement des commandes
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Gérer les erreurs avec grâce

Toujours encadrer les opérations risquées :

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### Filtrer les événements tôt

Retourner tôt si l'événement n'est pas pertinent :

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Utiliser des clés d'événement spécifiques

Spécifier les événements exacts dans les métadonnées quand possible :

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Spécifique
```

Plutôt que :

```yaml
metadata: { "openclaw": { "events": ["command"] } } # Général - plus de surcharge
```

## Débogage

### Activer la journalisation des hooks

Le Gateway journalise le chargement des hooks au démarrage :

```
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Vérifier la découverte

Lister tous les hooks découverts :

```bash
openclaw hooks list --verbose
```

### Vérifier l'enregistrement

Dans votre gestionnaire, journalisez quand il est appelé :

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Vérifier l'éligibilité

Vérifier pourquoi un hook n'est pas éligible :

```bash
openclaw hooks info my-hook
```

Recherchez les exigences manquantes dans la sortie.

## Tests

### Logs du Gateway

Surveiller les logs du Gateway pour voir l'exécution des hooks :

```bash
# macOS
./scripts/clawlog.sh -f

# Autres plateformes
tail -f ~/.openclaw/gateway.log
```

### Tester les hooks directement

Tester vos gestionnaires en isolation :

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // Assert side effects
});
```

## Architecture

### Composants principaux

- **`src/hooks/types.ts`** : définitions de types
- **`src/hooks/workspace.ts`** : scan de répertoires et chargement
- **`src/hooks/frontmatter.ts`** : analyse des métadonnées HOOK.md
- **`src/hooks/config.ts`** : vérification d'éligibilité
- **`src/hooks/hooks-status.ts`** : rapports de statut
- **`src/hooks/loader.ts`** : chargeur de module dynamique
- **`src/cli/hooks-cli.ts`** : commandes CLI
- **`src/gateway/server-startup.ts`** : charge les hooks au démarrage du Gateway
- **`src/auto-reply/reply/commands-core.ts`** : déclenche les événements de commande

### Flux de découverte

```
Démarrage du Gateway
    ↓
Scan des répertoires (espace de travail → gérés → intégrés)
    ↓
Analyse des fichiers HOOK.md
    ↓
Vérification d'éligibilité (bins, env, config, os)
    ↓
Chargement des gestionnaires depuis les hooks éligibles
    ↓
Enregistrement des gestionnaires pour les événements
```

### Flux d'événements

```
L'utilisateur envoie /new
    ↓
Validation de commande
    ↓
Création de l'événement hook
    ↓
Déclenchement du hook (tous les gestionnaires enregistrés)
    ↓
Le traitement de la commande continue
    ↓
Réinitialisation de session
```

## Dépannage

### Hook non découvert

1. Vérifier la structure du répertoire :

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Doit afficher : HOOK.md, handler.ts
   ```

2. Vérifier le format HOOK.md :

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Doit avoir un frontmatter YAML avec name et metadata
   ```

3. Lister tous les hooks découverts :

   ```bash
   openclaw hooks list
   ```

### Hook non éligible

Vérifier les exigences :

```bash
openclaw hooks info my-hook
```

Recherchez les éléments manquants :

- Binaires (vérifier le PATH)
- Variables d'environnement
- Valeurs de configuration
- Compatibilité OS

### Hook non exécute

1. Vérifier que le hook est activé :

   ```bash
   openclaw hooks list
   # Doit afficher ✓ à côté des hooks activés
   ```

2. Redémarrer votre processus Gateway pour que les hooks se rechargent.

3. Vérifier les logs du Gateway pour les erreurs :

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Erreurs de gestionnaire

Vérifier les erreurs TypeScript/import :

```bash
# Tester l'import directement
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Guide de migration

### De l'ancienne configuration à la découverte

**Avant** :

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Après** :

1. Créer le répertoire du hook :

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. Créer HOOK.md :

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. Mettre à jour la configuration :

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Vérifier et redémarrer votre processus Gateway :

   ```bash
   openclaw hooks list
   # Doit afficher : 🎯 my-hook ✓
   ```

**Avantages de la migration** :

- Découverte automatique
- Gestion par CLI
- Vérification d'éligibilité
- Meilleure documentation
- Structure cohérente

## Voir aussi

- [Référence CLI : hooks](/cli/hooks)
- [README des hooks intégrés](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)
