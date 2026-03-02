---
title: "Architecture d'intégration Pi"
summary: "Architecture de l'intégration de l'agent Pi embarqué dans OpenClaw et cycle de vie des sessions"
read_when:
  - Compréhension de la conception de l'intégration Pi SDK dans OpenClaw
  - Modification du cycle de vie des sessions d'agent, de l'outillage ou du câblage fournisseur pour Pi
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/pi.md
  workflow: manual
---

# Architecture d'intégration Pi

Ce document décrit comment OpenClaw s'intègre avec [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) et ses packages frères (`pi-ai`, `pi-agent-core`, `pi-tui`) pour alimenter ses capacités d'agent IA.

## Vue d'ensemble

OpenClaw utilise le SDK Pi pour embarquer un agent de codage IA dans son architecture de gateway de messagerie. Au lieu de lancer Pi comme sous-processus ou d'utiliser le mode RPC, OpenClaw importe et instancie directement la `AgentSession` de Pi via `createAgentSession()`. Cette approche embarquée fournit :

- Contrôle total du cycle de vie de session et de la gestion des événements
- Injection d'outils personnalisés (messagerie, bac à sable, actions spécifiques au canal)
- Personnalisation du prompt système par canal/contexte
- Persistance de session avec support de branchement/compaction
- Rotation de profils d'authentification multi-comptes avec basculement
- Changement de modèle agnostique du fournisseur

## Dépendances de packages

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| Package           | Objectif                                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------------ |
| `pi-ai`           | Abstractions LLM de base : `Model`, `streamSimple`, types de messages, APIs fournisseur                |
| `pi-agent-core`   | Boucle d'agent, exécution d'outils, types `AgentMessage`                                               |
| `pi-coding-agent` | SDK de haut niveau : `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, outils intégrés |
| `pi-tui`          | Composants d'interface terminal (utilisés dans le mode TUI local d'OpenClaw)                           |

## Structure des fichiers

```
src/agents/
├── pi-embedded-runner.ts          # Ré-exports depuis pi-embedded-runner/
├── pi-embedded-runner/
│   ├── run.ts                     # Point d'entrée principal : runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # Logique de tentative unique avec configuration de session
│   │   ├── params.ts              # Type RunEmbeddedPiAgentParams
│   │   ├── payloads.ts            # Construction des payloads de réponse depuis les résultats d'exécution
│   │   ├── images.ts              # Injection d'images pour le modèle de vision
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # Détection d'erreur d'abandon
│   ├── cache-ttl.ts               # Suivi du TTL de cache pour l'élagage de contexte
│   ├── compact.ts                 # Logique de compaction manuelle/automatique
│   ├── extensions.ts              # Chargement des extensions Pi pour les exécutions embarquées
│   ├── extra-params.ts            # Paramètres de streaming spécifiques au fournisseur
│   ├── google.ts                  # Corrections d'ordonnancement des tours Google/Gemini
│   ├── history.ts                 # Limitation de l'historique (DM vs groupe)
│   ├── lanes.ts                   # Voies de commande session/globales
│   ├── logger.ts                  # Logger de sous-système
│   ├── model.ts                   # Résolution de modèle via ModelRegistry
│   ├── runs.ts                    # Suivi des exécutions actives, abandon, file d'attente
│   ├── sandbox-info.ts            # Info bac à sable pour le prompt système
│   ├── session-manager-cache.ts   # Cache d'instances SessionManager
│   ├── session-manager-init.ts    # Initialisation du fichier de session
│   ├── system-prompt.ts           # Constructeur du prompt système
│   ├── tool-split.ts              # Séparation des outils en builtIn vs custom
│   ├── types.ts                   # EmbeddedPiAgentMeta, EmbeddedPiRunResult
│   └── utils.ts                   # Mapping ThinkLevel, description d'erreur
├── pi-embedded-subscribe.ts       # Souscription/dispatch d'événements de session
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # Fabrique de gestionnaires d'événements
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # Chunking de réponses en blocs par streaming
├── pi-embedded-messaging.ts       # Suivi d'envoi de l'outil de messagerie
├── pi-embedded-helpers.ts         # Classification d'erreurs, validation de tours
├── pi-embedded-helpers/           # Modules auxiliaires
├── pi-embedded-utils.ts           # Utilitaires de formatage
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # Wrapping AbortSignal pour les outils
├── pi-tools.policy.ts             # Politique d'allowlist/denylist d'outils
├── pi-tools.read.ts               # Personnalisations de l'outil Read
├── pi-tools.schema.ts             # Normalisation des schémas d'outils
├── pi-tools.types.ts              # Alias de type AnyAgentTool
├── pi-tool-definition-adapter.ts  # Adaptateur AgentTool -> ToolDefinition
├── pi-settings.ts                 # Surcharges de paramètres
├── pi-extensions/                 # Extensions Pi personnalisées
│   ├── compaction-safeguard.ts    # Extension de garde
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # Extension d'élagage de contexte par cache-TTL
│   └── context-pruning/
├── model-auth.ts                  # Résolution de profil d'authentification
├── auth-profiles.ts               # Store de profils, cooldown, basculement
├── model-selection.ts             # Résolution du modèle par défaut
├── models-config.ts               # Génération de models.json
├── model-catalog.ts               # Cache du catalogue de modèles
├── context-window-guard.ts        # Validation de la fenêtre de contexte
├── failover-error.ts              # Classe FailoverError
├── defaults.ts                    # DEFAULT_PROVIDER, DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # Résolution des paramètres du prompt système
├── system-prompt-report.ts        # Génération de rapport de débogage
├── tool-summaries.ts              # Résumés de descriptions d'outils
├── tool-policy.ts                 # Résolution de politique d'outils
├── transcript-policy.ts           # Politique de validation de transcription
├── skills.ts                      # Construction de snapshot/prompt de Skills
├── skills/                        # Sous-système Skills
├── sandbox.ts                     # Résolution du contexte bac à sable
├── sandbox/                       # Sous-système bac à sable
├── channel-tools.ts               # Injection d'outils spécifiques au canal
├── openclaw-tools.ts              # Outils spécifiques OpenClaw
├── bash-tools.ts                  # Outils exec/process
├── apply-patch.ts                 # Outil apply_patch (OpenAI)
├── tools/                         # Implémentations d'outils individuels
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```

## Flux d'intégration principal

### 1. Exécution d'un agent embarqué

Le point d'entrée principal est `runEmbeddedPiAgent()` dans `pi-embedded-runner/run.ts` :

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2. Création de session

Dans `runEmbeddedAttempt()` (appelé par `runEmbeddedPiAgent()`), le SDK Pi est utilisé :

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```

### 3. Souscription aux événements

`subscribeEmbeddedPiSession()` souscrit aux événements de la `AgentSession` de Pi :

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

Les événements gérés incluent :

- `message_start` / `message_end` / `message_update` (streaming texte/réflexion)
- `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
- `turn_start` / `turn_end`
- `agent_start` / `agent_end`
- `auto_compaction_start` / `auto_compaction_end`

### 4. Prompting

Après la configuration, la session est promptée :

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

Le SDK gère la boucle d'agent complète : envoi au LLM, exécution des appels d'outils, streaming des réponses.

## Architecture des outils

### Pipeline d'outils

1. **Outils de base** : `codingTools` de Pi (read, bash, edit, write)
2. **Remplacements personnalisés** : OpenClaw remplace bash par `exec`/`process`, personnalisé read/edit/write pour le bac à sable
3. **Outils OpenClaw** : messagerie, navigateur, canvas, sessions, cron, gateway, etc.
4. **Outils de canal** : outils d'actions spécifiques Discord/Telegram/Slack/WhatsApp
5. **Filtrage par politique** : outils filtrés par profil, fournisseur, agent, groupe, politiques de bac à sable
6. **Normalisation des schémas** : schémas nettoyés pour les particularités Gemini/OpenAI
7. **Wrapping AbortSignal** : outils encapsulés pour respecter les signaux d'abandon

### Adaptateur de définition d'outil

Le `AgentTool` de pi-agent-core a une signature `exécuté` différente de `ToolDefinition` de pi-coding-agent. L'adaptateur dans `pi-tool-définition-adapter.ts` fait le pont :

```typescript
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // La signature pi-coding-agent diffère de pi-agent-core
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### Stratégie de séparation des outils

`splitSdkTools()` passe tous les outils via `customTools` :

```typescript
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // Vide. Nous remplaçons tout
    customTools: toToolDefinitions(options.tools),
  };
}
```

Cela garantit que le filtrage par politique d'OpenClaw, l'intégration du bac à sable et le jeu d'outils étendu restent cohérents entre les fournisseurs.

## Construction du prompt système

Le prompt système est construit dans `buildAgentSystemPrompt()` (`system-prompt.ts`). Il assemble un prompt complet avec des sections incluant Outillage, Style d'appel d'outil, Garde-fous de sécurité, Référence CLI OpenClaw, Skills, Docs, Espace de travail, Bac à sable, Messagerie, Tags de réponse, Voix, Réponses silencieuses, Heartbeats, Métadonnées runtime, plus Mémoire et Réactions quand activées, et fichiers de contexte optionnels et contenu de prompt système supplémentaire. Les sections sont réduites pour le mode prompt minimal utilisé par les sous-agents.

Le prompt est appliqué après la création de session via `applySystemPromptOverrideToSession()` :

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```

## Gestion des sessions

### Fichiers de session

Les sessions sont des fichiers JSONL avec une structure arborescente (liaison id/parentId). Le `SessionManager` de Pi gère la persistance :

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

OpenClaw encapsule ceci avec `guardSessionManager()` pour la sécurité des résultats d'outils.

### Cache de sessions

`session-manager-cache.ts` met en cache les instances SessionManager pour éviter le parsing répété de fichiers :

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### Limitation de l'historique

`limitHistoryTurns()` tronque l'historique de conversation selon le type de canal (DM vs groupe).

### Compaction

La compaction automatique se déclenche sur le dépassement de contexte. `compactEmbeddedPiSessionDirect()` gère la compaction manuelle :

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```

## Authentification et résolution de modèle

### Profils d'authentification

OpenClaw maintient un store de profils d'authentification avec plusieurs clés API par fournisseur :

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

Les profils sont rotés sur échec avec suivi de cooldown :

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### Résolution de modèle

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// Utilise le ModelRegistry et AuthStorage de Pi
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### Basculement

`FailoverError` déclenche le repli de modèle quand configuré :

```typescript
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```

## Extensions Pi

OpenClaw charge des extensions Pi personnalisées pour des comportements spécialisés :

### Garde de compaction

`src/agents/pi-extensions/compaction-safeguard.ts` ajouté des garde-fous à la compaction, incluant un budget de tokens adaptatif plus des résumés d'échecs d'outils et d'opérations de fichiers :

```typescript
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### Élagage de contexte

`src/agents/pi-extensions/context-pruning.ts` implémente l'élagage de contexte basé sur le cache-TTL :

```typescript
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```

## Streaming et réponses par blocs

### Chunking par blocs

`EmbeddedBlockChunker` gère le streaming de texte en blocs de réponse discrets :

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### Suppression des tags Thinking/Final

La sortie en streaming est traitée pour supprimer les blocs `<think>`/`<thinking>` et extraire le contenu `<final>` :

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // Supprimer le contenu <think>...</think>
  // Si enforceFinalTag, ne retourner que le contenu <final>...</final>
};
```

### Directives de réponse

Les directives de réponse comme `[[média:url]]`, `[[voice]]`, `[[reply:id]]` sont parsées et extraites :

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```

## Gestion des erreurs

### Classification des erreurs

`pi-embedded-helpers.ts` classifie les erreurs pour un traitement approprié :

```typescript
isContextOverflowError(errorText)     // Contexte trop large
isCompactionFailureError(errorText)   // Échec de compaction
isAuthAssistantError(lastAssistant)   // Échec d'authentification
isRateLimitAssistantError(...)        // Limite de débit atteinte
isFailoverAssistantError(...)         // Doit basculer
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### Repli du niveau de réflexion

Si un niveau de réflexion n'est pas supporté, il se replie :

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```

## Intégration bac à sable

Quand le mode bac à sable est activé, les outils et chemins sont contraints :

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // Utiliser les outils read/edit/write sandboxés
  // Exec s'exécute dans le conteneur
  // Le navigateur utilise l'URL bridge
}
```

## Gestion spécifique aux fournisseurs

### Anthropic

- Nettoyage de chaîne magique de refus
- Validation des tours pour les rôles consécutifs
- Compatibilité des paramètres Claude Code

### Google/Gemini

- Corrections d'ordonnancement des tours (`applyGoogleTurnOrderingFix`)
- Assainissement des schémas d'outils (`sanitizeToolsForGoogle`)
- Assainissement de l'historique de session (`sanitizeSessionHistory`)

### OpenAI

- Outil `apply_patch` pour les modèles Codex
- Gestion du déclassement du niveau de réflexion

## Intégration TUI

OpenClaw dispose aussi d'un mode TUI local qui utilise directement les composants pi-tui :

```typescript
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

Cela fournit l'expérience terminal interactive similaire au mode natif de Pi.

## Différences clés avec le CLI Pi

| Aspect            | CLI Pi                  | Embarqué OpenClaw                                                                              |
| ----------------- | ----------------------- | ---------------------------------------------------------------------------------------------- |
| Invocation        | Commande `pi` / RPC     | SDK via `createAgentSession()`                                                                 |
| Outils            | Outils de codage par défaut | Suite d'outils personnalisés OpenClaw                                                        |
| Prompt système    | AGENTS.md + prompts     | Dynamique par canal/contexte                                                                   |
| Stockage de session | `~/.pi/agent/sessions/` | `~/.openclaw/agents/<agentId>/sessions/` (ou `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`) |
| Authentification  | Identifiant unique      | Multi-profil avec rotation                                                                     |
| Extensions        | Chargées depuis le disque | Chemins programmatiques + disque                                                              |
| Gestion d'événements | Rendu TUI            | Basé sur les callbacks (onBlockReply, etc.)                                                    |

## Considérations futures

Domaines de retravail potentiel :

1. **Alignement des signatures d'outils** : Adaptation actuelle entre les signatures pi-agent-core et pi-coding-agent
2. **Encapsulation du gestionnaire de session** : `guardSessionManager` ajouté de la sécurité mais augmente la complexité
3. **Chargement d'extensions** : Pourrait utiliser le `ResourceLoader` de Pi plus directement
4. **Complexité du gestionnaire de streaming** : `subscribeEmbeddedPiSession` a beaucoup grandi
5. **Particularités fournisseurs** : Nombreux chemins de code spécifiques aux fournisseurs que Pi pourrait potentiellement gérer

## Tests

La couverture d'intégration Pi couvre ces suites :

- `src/agents/pi-*.test.ts`
- `src/agents/pi-auth-json.test.ts`
- `src/agents/pi-embedded-*.test.ts`
- `src/agents/pi-embedded-helpers*.test.ts`
- `src/agents/pi-embedded-runner*.test.ts`
- `src/agents/pi-embedded-runner/**/*.test.ts`
- `src/agents/pi-embedded-subscribe*.test.ts`
- `src/agents/pi-tools*.test.ts`
- `src/agents/pi-tool-définition-adapter*.test.ts`
- `src/agents/pi-settings.test.ts`
- `src/agents/pi-extensions/**/*.test.ts`

Live/opt-in :

- `src/agents/pi-embedded-runner-extraparams.live.test.ts` (activer `OPENCLAW_LIVE_TEST=1`)

Pour les commandes d'exécution actuelles, voir [Workflow de développement Pi](/pi-dev).
