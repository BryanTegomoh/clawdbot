---
summary: "Plan de production pour une supervision fiable des processus interactifs (PTY + non-PTY) avec propriété explicite, cycle de vie unifié et nettoyage déterministe"
read_when:
  - Travail sur la propriété et le nettoyage du cycle de vie exec/process
  - Debug du comportement de supervision PTY et non-PTY
owner: "openclaw"
status: "in-progress"
last_updated: "2026-02-15"
title: "Plan de supervision PTY et processus"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/experiments/plans/pty-process-supervision.md
  workflow: manual
---

# Plan de supervision PTY et processus

## 1. Problème et objectif

Nous avons besoin d'un cycle de vie fiable unique pour l'exécution de commandes de longue durée à travers :

- les exécutions `exec` en avant-plan
- les exécutions `exec` en arrière-plan
- les actions de suivi `process` (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
- les sous-processus du runner d'agent CLI

L'objectif n'est pas seulement de supporter le PTY. L'objectif est une propriété, une annulation, un timeout et un nettoyage prévisibles sans heuristiques de correspondance de processus non sûres.

## 2. Périmètre et limités

- Conserver l'implémentation interne dans `src/process/supervisor`.
- Ne pas créer de nouveau package pour cela.
- Conserver la compatibilité de comportement actuel quand c'est pratique.
- Ne pas élargir le périmètre au terminal replay ou à la persistance de session de type tmux.

## 3. Implémenté dans cette branche

### Base du superviseur déjà présente

- Le module superviseur est en place sous `src/process/supervisor/*`.
- L'exec runtime et le CLI runner sont déjà routés à travers le spawn et le wait du superviseur.
- La finalisation du registre est idempotente.

### Cette passe a complété

1. Contrat de commande PTY explicite

- `SpawnInput` est maintenant une union discriminée dans `src/process/supervisor/types.ts`.
- Les exécutions PTY requièrent `ptyCommand` au lieu de réutiliser un `argv` générique.
- Le superviseur ne reconstruit plus les chaînes de commande PTY à partir de joins argv dans `src/process/supervisor/supervisor.ts`.
- L'exec runtime passe maintenant `ptyCommand` directement dans `src/agents/bash-tools.exec-runtime.ts`.

2. Découplage des types de la couche process

- Les types du superviseur n'importent plus `SessionStdin` depuis agents.
- Le contrat stdin local au processus réside dans `src/process/supervisor/types.ts` (`ManagedRunStdin`).
- Les adaptateurs ne dépendent plus que des types de niveau process :
  - `src/process/supervisor/adapters/child.ts`
  - `src/process/supervisor/adapters/pty.ts`

3. Amélioration de la propriété du cycle de vie de l'outil process

- `src/agents/bash-tools.process.ts` demande maintenant l'annulation via le superviseur en premier.
- `process kill/remove` utilisé maintenant la terminaison de repli par arbre de processus quand la recherche du superviseur échoue.
- `remove` conserve un comportement de suppression déterministe en supprimant immédiatement les entrées de session en cours après que la terminaison est demandée.

4. Valeurs par défaut du watchdog en source unique

- Ajout de valeurs par défaut partagées dans `src/agents/cli-watchdog-defaults.ts`.
- `src/agents/cli-backends.ts` consomme les valeurs par défaut partagées.
- `src/agents/cli-runner/reliability.ts` consomme les mêmes valeurs par défaut partagées.

5. Nettoyage de helpers obsolètes

- Suppression du chemin helper `killSession` inutilisé de `src/agents/bash-tools.shared.ts`.

6. Tests de chemin direct superviseur ajoutés

- Ajout de `src/agents/bash-tools.process.supervisor.test.ts` pour couvrir le routage kill et remove à travers l'annulation du superviseur.

7. Corrections de lacunes de fiabilité complétées

- `src/agents/bash-tools.process.ts` se replie maintenant sur une terminaison de processus réelle au niveau OS quand la recherche du superviseur échoue.
- `src/process/supervisor/adapters/child.ts` utilisé maintenant la sémantique de terminaison par arbre de processus pour les chemins de kill par défaut cancel/timeout.
- Ajout d'un utilitaire d'arbre de processus partagé dans `src/process/kill-tree.ts`.

8. Couverture de cas limités du contrat PTY ajoutée

- Ajout de `src/process/supervisor/supervisor.pty-command.test.ts` pour le transfert verbatim de commande PTY et le rejet de commande vide.
- Ajout de `src/process/supervisor/adapters/child.test.ts` pour le comportement de kill par arbre de processus lors de l'annulation de l'adaptateur child.

## 4. Lacunes restantes et décisions

### État de la fiabilité

Les deux lacunes de fiabilité requises pour cette passe sont maintenant comblées :

- `process kill/remove` dispose maintenant d'un repli de terminaison OS réelle quand la recherche du superviseur échoue.
- Le cancel/timeout child utilisé maintenant la sémantique de kill par arbre de processus pour le chemin de kill par défaut.
- Des tests de régression ont été ajoutés pour les deux comportements.

### Durabilité et réconciliation au démarrage

Le comportement au redémarrage est maintenant explicitement défini comme cycle de vie en mémoire uniquement.

- `reconcileOrphans()` reste un no-op dans `src/process/supervisor/supervisor.ts` par conception.
- Les exécutions activés ne sont pas récupérées après le redémarrage du processus.
- Cette limité est intentionnelle pour cette passe d'implémentation afin d'éviter les risques de persistance partielle.

### Suivis de maintenabilité

1. `runExecProcess` dans `src/agents/bash-tools.exec-runtime.ts` gère encore plusieurs responsabilités et peut être découpé en helpers ciblés dans un suivi.

## 5. Plan d'implémentation

La passe d'implémentation pour les items requis de fiabilité et de contrat est complète.

Complété :

- Repli de terminaison réelle pour `process kill/remove`
- Annulation par arbre de processus pour le chemin de kill par défaut de l'adaptateur child
- Tests de régression pour le kill de repli et le chemin de kill de l'adaptateur child
- Tests de cas limités de commande PTY sous `ptyCommand` explicite
- Limité explicite de redémarrage en mémoire avec `reconcileOrphans()` no-op par conception

Suivi optionnel :

- Découper `runExecProcess` en helpers ciblés sans dérive de comportement

## 6. Carte des fichiers

### Superviseur de processus

- `src/process/supervisor/types.ts` mis à jour avec l'entrée de spawn discriminée et le contrat stdin local au processus.
- `src/process/supervisor/supervisor.ts` mis à jour pour utiliser `ptyCommand` explicite.
- `src/process/supervisor/adapters/child.ts` et `src/process/supervisor/adapters/pty.ts` découplés des types agent.
- `src/process/supervisor/registry.ts` finalize idempotent inchangé et conservé.

### Intégration exec et process

- `src/agents/bash-tools.exec-runtime.ts` mis à jour pour passer la commande PTY explicitement et conserver le chemin de repli.
- `src/agents/bash-tools.process.ts` mis à jour pour annuler via le superviseur avec terminaison de repli par arbre de processus réel.
- `src/agents/bash-tools.shared.ts` chemin de helper kill direct supprimé.

### Fiabilité CLI

- `src/agents/cli-watchdog-defaults.ts` ajouté comme base partagée.
- `src/agents/cli-backends.ts` et `src/agents/cli-runner/reliability.ts` consomment maintenant les mêmes valeurs par défaut.

## 7. Exécution de validation dans cette passe

Tests unitaires :

- `pnpm vitest src/process/supervisor/registry.test.ts`
- `pnpm vitest src/process/supervisor/supervisor.test.ts`
- `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
- `pnpm vitest src/process/supervisor/adapters/child.test.ts`
- `pnpm vitest src/agents/cli-backends.test.ts`
- `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
- `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
- `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
- `pnpm vitest src/process/exec.test.ts`

Cibles E2E :

- `pnpm vitest src/agents/cli-runner.test.ts`
- `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

Note sur le typecheck :

- Utilisez `pnpm build` (et `pnpm check` pour le gate complet lint/docs) dans ce dépôt. Les notes plus anciennes qui mentionnent `pnpm tsgo` sont obsolètes.

## 8. Garanties opérationnelles préservées

- Le comportement de durcissement de l'environnement exec est inchangé.
- Le flux d'approbation et de liste d'autorisation est inchangé.
- L'assainissement de sortie et les limités de sortie sont inchangés.
- L'adaptateur PTY garantit toujours la résolution du wait lors d'un kill forcé et la libération des listeners.

## 9. Définition de terminé

1. Le superviseur est propriétaire du cycle de vie pour les exécutions gérées.
2. Le spawn PTY utilisé un contrat de commande explicite sans reconstruction d'argv.
3. La couche process n'a pas de dépendance de type sur la couche agent pour les contrats stdin du superviseur.
4. Les valeurs par défaut du watchdog sont en source unique.
5. Les tests unitaires et e2e ciblés restent verts.
6. La limité de durabilité au redémarrage est explicitement documentée ou complètement implémentée.

## 10. Résumé

La branche dispose maintenant d'une forme de supervision cohérente et plus sûre :

- contrat PTY explicite
- couche process plus propre
- chemin d'annulation piloté par le superviseur pour les opérations process
- terminaison de repli réelle quand la recherche du superviseur échoue
- annulation par arbre de processus pour les chemins de kill par défaut des exécutions child
- valeurs par défaut du watchdog unifiées
- limité explicite de redémarrage en mémoire (pas de réconciliation d'orphelins entre les redémarrages dans cette passe)
