---
summary: "Plan : un SDK plugin propre + runtime pour tous les connecteurs de messagerie"
read_when:
  - Définition ou refactorisation de l'architecture plugin
  - Migration des connecteurs de canal vers le SDK/runtime plugin
title: "Refactorisation du SDK plugin"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/refactor/plugin-sdk.md
  workflow: manual
---

# Plan de refactorisation SDK plugin + runtime

Objectif : chaque connecteur de messagerie est un plugin (bundlé ou externe) utilisant une API stable unique.
Aucun plugin n'importe directement depuis `src/**`. Toutes les dépendances passent par le SDK ou le runtime.

## Pourquoi maintenant

- Les connecteurs actuels mélangent les patterns : imports directs du cœur, bridges dist-only et helpers personnalisés.
- Cela rend les mises à jour fragiles et bloque une surface plugin externe propre.

## Architecture cible (deux couches)

### 1) SDK plugin (compile-time, stable, publiable)

Périmètre : types, helpers et utilitaires de config. Pas d'état runtime, pas d'effets de bord.

Contenu (exemples) :

- Types : `ChannelPlugin`, adaptateurs, `ChannelMeta`, `ChannelCapabilities`, `ChannelDirectoryEntry`.
- Helpers de config : `buildChannelConfigSchema`, `setAccountEnabledInConfigSection`, `deleteAccountFromConfigSection`,
  `applyAccountNameToChannelSection`.
- Helpers d'appairage : `PAIRING_APPROVED_MESSAGE`, `formatPairingApproveHint`.
- Helpers de configuration initiale : `promptChannelAccessConfig`, `addWildcardAllowFrom`, types de configuration initiale.
- Helpers de paramètres d'outil : `createActionGate`, `readStringParam`, `readNumberParam`, `readReactionParams`, `jsonResult`.
- Helper de lien docs : `formatDocsLink`.

Livraison :

- Publier comme `openclaw/plugin-sdk` (ou exporter depuis le cœur sous `openclaw/plugin-sdk`).
- Semver avec des garanties de stabilité explicites.

### 2) Runtime plugin (surface d'exécution, injectée)

Périmètre : tout ce qui touche au comportement runtime du cœur.
Accessible via `OpenClawPluginApi.runtime` pour que les plugins n'importent jamais `src/**`.

Surface proposée (minimale mais complète) :

```ts
export type PluginRuntime = {
  channel: {
    text: {
      chunkMarkdownText(text: string, limit: number): string[];
      resolveTextChunkLimit(cfg: OpenClawConfig, channel: string, accountId?: string): number;
      hasControlCommand(text: string, cfg: OpenClawConfig): boolean;
    };
    reply: {
      dispatchReplyWithBufferedBlockDispatcher(params: {
        ctx: unknown;
        cfg: unknown;
        dispatcherOptions: {
          deliver: (payload: {
            text?: string;
            mediaUrls?: string[];
            mediaUrl?: string;
          }) => void | Promise<void>;
          onError?: (err: unknown, info: { kind: string }) => void;
        };
      }): Promise<void>;
      createReplyDispatcherWithTyping?: unknown; // adaptateur pour les flux de type Teams
    };
    routing: {
      resolveAgentRoute(params: {
        cfg: unknown;
        channel: string;
        accountId: string;
        peer: { kind: RoutePeerKind; id: string };
      }): { sessionKey: string; accountId: string };
    };
    pairing: {
      buildPairingReply(params: { channel: string; idLine: string; code: string }): string;
      readAllowFromStore(channel: string): Promise<string[]>;
      upsertPairingRequest(params: {
        channel: string;
        id: string;
        meta?: { name?: string };
      }): Promise<{ code: string; created: boolean }>;
    };
    media: {
      fetchRemoteMedia(params: { url: string }): Promise<{ buffer: Buffer; contentType?: string }>;
      saveMediaBuffer(
        buffer: Uint8Array,
        contentType: string | undefined,
        direction: "inbound" | "outbound",
        maxBytes: number,
      ): Promise<{ path: string; contentType?: string }>;
    };
    mentions: {
      buildMentionRegexes(cfg: OpenClawConfig, agentId?: string): RegExp[];
      matchesMentionPatterns(text: string, regexes: RegExp[]): boolean;
    };
    groups: {
      resolveGroupPolicy(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
      ): {
        allowlistEnabled: boolean;
        allowed: boolean;
        groupConfig?: unknown;
        defaultConfig?: unknown;
      };
      resolveRequireMention(
        cfg: OpenClawConfig,
        channel: string,
        accountId: string,
        groupId: string,
        override?: boolean,
      ): boolean;
    };
    debounce: {
      createInboundDebouncer<T>(opts: {
        debounceMs: number;
        buildKey: (v: T) => string | null;
        shouldDebounce: (v: T) => boolean;
        onFlush: (entries: T[]) => Promise<void>;
        onError?: (err: unknown) => void;
      }): { push: (v: T) => void; flush: () => Promise<void> };
      resolveInboundDebounceMs(cfg: OpenClawConfig, channel: string): number;
    };
    commands: {
      resolveCommandAuthorizedFromAuthorizers(params: {
        useAccessGroups: boolean;
        authorizers: Array<{ configured: boolean; allowed: boolean }>;
      }): boolean;
    };
  };
  logging: {
    shouldLogVerbose(): boolean;
    getChildLogger(name: string): PluginLogger;
  };
  state: {
    resolveStateDir(cfg: OpenClawConfig): string;
  };
};
```

Notes :

- Le runtime est le seul moyen d'accéder au comportement du cœur.
- Le SDK est intentionnellement petit et stable.
- Chaque méthode runtime correspond à une implémentation existante du cœur (pas de duplication).

## Plan de migration (phasé, sûr)

### Phase 0 : scaffolding

- Introduire `openclaw/plugin-sdk`.
- Ajouter `api.runtime` à `OpenClawPluginApi` avec la surface ci-dessus.
- Maintenir les imports existants pendant une fenêtre de transition (avertissements de dépréciation).

### Phase 1 : nettoyage des bridges (faible risque)

- Remplacer les `core-bridge.ts` par extension par `api.runtime`.
- Migrer BlueBubbles, Zalo, Zalo Personal d'abord (déjà proches).
- Supprimer le code bridge dupliqué.

### Phase 2 : plugins à import direct léger

- Migrer Matrix vers SDK + runtime.
- Valider la configuration initiale, le répertoire, la logique de mention de groupe.

### Phase 3 : plugins à import direct lourd

- Migrer MS Teams (plus grand ensemble de helpers runtime).
- S'assurer que la sémantique reply/typing correspond au comportement actuel.

### Phase 4 : transformation d'iMessage en plugin

- Déplacer iMessage dans `extensions/imessage`.
- Remplacer les appels directs au cœur par `api.runtime`.
- Conserver les clés de config, le comportement CLI et la documentation intacts.

### Phase 5 : application

- Ajouter une règle lint / vérification CI : pas d'imports `extensions/**` depuis `src/**`.
- Ajouter des vérifications de compatibilité SDK/version plugin (semver runtime + SDK).

## Compatibilité et versioning

- SDK : semver, publié, changements documentés.
- Runtime : versionné par release du cœur. Ajouter `api.runtime.version`.
- Les plugins déclarent une plage de runtime requise (ex. `openclawRuntime: ">=2026.2.0"`).

## Stratégie de test

- Tests unitaires au niveau adaptateur (fonctions runtime exercées avec l'implémentation réelle du cœur).
- Tests golden par plugin : assurer aucune dérive de comportement (routage, appairage, allowlist, gating de mention).
- Un seul exemple de plugin end-to-end utilisé en CI (install + run + smoke).

## Questions ouvertes

- Où héberger les types SDK : package séparé ou export du cœur ?
- Distribution des types runtime : dans le SDK (types uniquement) ou dans le cœur ?
- Comment exposer les liens docs pour les plugins bundlés vs externes ?
- Autorisons-nous des imports directs limités du cœur pour les plugins in-repo pendant la transition ?

## Critères de succès

- Tous les connecteurs de canal sont des plugins utilisant SDK + runtime.
- Pas d'imports `extensions/**` depuis `src/**`.
- Les templates de nouveau connecteur ne dépendent que du SDK + runtime.
- Les plugins externes peuvent être développés et mis à jour sans accès au code source du cœur.

Documentation associée : [Plugins](/tools/plugin), [Canaux](/channels/index), [Configuration](/gateway/configuration).
