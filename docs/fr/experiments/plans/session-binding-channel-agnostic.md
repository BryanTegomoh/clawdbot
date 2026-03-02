---
summary: "Architecture de liaison de session agnostique au canal et périmètre de livraison de l'itération 1"
read_when:
  - Refactorisation du routage de session agnostique au canal et des liaisons
  - Investigation de livraisons dupliquées, obsolètes ou manquantes de session entre canaux
owner: "onutc"
status: "in-progress"
last_updated: "2026-02-21"
title: "Plan de liaison de session agnostique au canal"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/experiments/plans/session-binding-channel-agnostic.md
  workflow: manual
---

# Plan de liaison de session agnostique au canal

## Vue d'ensemble

Ce document définit le modèle de liaison de session agnostique au canal à long terme et le périmètre concret pour la prochaine itération d'implémentation.

Objectif :

- faire du routage de session lie aux sous-agents une capacité centrale
- conserver le comportement spécifique au canal dans les adaptateurs
- éviter les régressions dans le comportement Discord normal

## Pourquoi cela existe

Le comportement actuel mélange :

- la politique de contenu de complétion
- la politique de routage de destination
- les détails spécifiques à Discord

Cela a causé des cas limités tels que :

- livraison dupliquée main et thread sous des exécutions concurrentes
- utilisation de token obsolète sur les gestionnaires de liaison réutilisés
- comptabilité d'activité manquante pour les envois par webhook

## Périmètre de l'itération 1

Cette itération est intentionnellement limitée.

### 1. Ajouter des interfaces centrales agnostiques au canal

Ajouter des types centraux et des interfaces de service pour les liaisons et le routage.

Types centraux proposés :

```ts
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

Contrat de service central :

```ts
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2. Ajouter un routeur de livraison central pour les complétions de sous-agents

Ajouter un chemin unique de résolution de destination pour les événements de complétion.

Contrat du routeur :

```ts
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

Pour cette itération :

- seul `task_completion` est routé par ce nouveau chemin
- les chemins existants pour les autres types d'événements restent en l'état

### 3. Conserver Discord comme adaptateur

Discord reste la première implémentation d'adaptateur.

Responsabilités de l'adaptateur :

- créer/réutiliser les conversations thread
- envoyer les messages liés par webhook ou envoi de canal
- valider l'état du thread (archivé/supprimé)
- mapper les métadonnées de l'adaptateur (identité webhook, ids de thread)

### 4. Corriger les problèmes d'exactitude actuellement connus

Requis dans cette itération :

- rafraîchir l'utilisation du token lors de la réutilisation d'un gestionnaire de liaison de thread existant
- enregistrer l'activité sortante pour les envois Discord basés sur webhook
- arrêter le repli implicite vers le canal principal quand une destination de thread liée est sélectionnée pour la complétion en mode session

### 5. Préserver les valeurs par défaut de sécurité runtime actuelles

Aucun changement de comportement pour les utilisateurs avec le spawn lié aux threads désactivé.

Les valeurs par défaut restent :

- `channels.discord.threadBindings.spawnSubagentSessions = false`

Résultat :

- les utilisateurs Discord normaux restent sur le comportement actuel
- le nouveau chemin central n'affecte que le routage de complétion de session liée là où il est activé

## Hors itération 1

Explicitement différé :

- cibles de liaison ACP (`targetKind: "acp"`)
- nouveaux adaptateurs de canal au-delà de Discord
- remplacement global de tous les chemins de livraison (`spawn_ack`, futur `subagent_message`)
- changements au niveau du protocole
- refonte de la migration/versioning du stockage pour toute la persistance des liaisons

Notes sur ACP :

- la conception de l'interface laisse de la place pour ACP
- l'implémentation ACP n'est pas commencée dans cette itération

## Invariants de routage

Ces invariants sont obligatoires pour l'itération 1.

- la sélection de destination et la génération de contenu sont des étapes séparées
- si la complétion en mode session se résout vers une destination liée activé, la livraison doit cibler cette destination
- pas de re-routage caché de la destination liée vers le canal principal
- le comportement de repli doit être explicite et observable

## Compatibilité et déploiement

Cible de compatibilité :

- pas de régression pour les utilisateurs avec le spawn lié aux threads désactivé
- pas de changement pour les canaux non-Discord dans cette itération

Déploiement :

1. Déployer les interfaces et le routeur derrière les feature gates actuels.
2. Router les livraisons liées en mode complétion Discord à travers le routeur.
3. Conserver le chemin legacy pour les flux non liés.
4. Vérifier avec des tests ciblés et des logs runtime canary.

## Tests requis dans l'itération 1

Couverture unitaire et d'intégration requise :

- la rotation de token du gestionnaire utilisé le dernier token après réutilisation du gestionnaire
- les envois webhook mettent à jour les timestamps d'activité du canal
- deux sessions liées activés dans le même canal demandeur ne dupliquent pas vers le canal principal
- la complétion pour une exécution en mode session liée se résout uniquement vers la destination thread
- le flag de spawn désactivé conserve le comportement legacy inchangé

## Fichiers d'implémentation proposés

Central :

- `src/infra/outbound/session-binding-service.ts` (nouveau)
- `src/infra/outbound/bound-delivery-router.ts` (nouveau)
- `src/agents/subagent-announce.ts` (intégration de la résolution de destination de complétion)

Adaptateur Discord et runtime :

- `src/discord/monitor/thread-bindings.manager.ts`
- `src/discord/monitor/reply-delivery.ts`
- `src/discord/send.outbound.ts`

Tests :

- `src/discord/monitor/provider*.test.ts`
- `src/discord/monitor/reply-delivery.test.ts`
- `src/agents/subagent-announce.format.test.ts`

## Critères de terminé pour l'itération 1

- les interfaces centrales existent et sont connectées pour le routage de complétion
- les corrections d'exactitude ci-dessus sont mergées avec des tests
- pas de livraison dupliquée de complétion main et thread dans les exécutions liées en mode session
- pas de changement de comportement pour les déploiements avec le spawn lié désactivé
- ACP reste explicitement différé
