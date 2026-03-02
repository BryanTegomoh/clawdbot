---
summary: "Plan : isoler browser act:evaluate de la file Playwright via CDP, avec des délais de bout en bout et une résolution de ref plus sûre"
read_when:
  - Travail sur les problèmes de timeout, d'abandon ou de blocage de file pour browser `act:evaluate`
  - Planification de l'isolation basée sur CDP pour l'exécution d'evaluate
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Refactorisation CDP de Browser Evaluate"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/experiments/plans/browser-evaluate-cdp-refactor.md
  workflow: manual
---

# Plan de refactorisation CDP de Browser Evaluate

## Contexte

`act:evaluate` exécute du JavaScript fourni par l'utilisateur dans la page. Aujourd'hui, il s'exécute via Playwright
(`page.evaluate` ou `locator.evaluate`). Playwright sérialise les commandes CDP par page, donc un
evaluate bloqué ou de longue durée peut bloquer la file de commandes de la page et faire apparaître toute action
ultérieure sur cet onglet comme « bloquée ».

La PR #13498 ajouté un filet de sécurité pragmatique (evaluate borné, propagation d'abandon et
récupération au mieux). Ce document décrit une refactorisation plus large qui rend `act:evaluate` intrinsèquement
isolé de Playwright afin qu'un evaluate bloqué ne puisse pas coincer les opérations normales de Playwright.

## Objectifs

- `act:evaluate` ne peut pas bloquer de façon permanente les actions navigateur ultérieures sur le même onglet.
- Les timeouts sont une référence unique de bout en bout afin qu'un appelant puisse s'appuyer sur un budget.
- L'abandon et le timeout sont traités de la même manière via HTTP et en dispatch in-process.
- Le ciblage d'éléments pour evaluate est supporté sans tout basculer hors de Playwright.
- Maintenir la rétrocompatibilité pour les appelants et payloads existants.

## Hors périmètre

- Remplacer toutes les actions navigateur (click, type, wait, etc.) par des implémentations CDP.
- Supprimer le filet de sécurité existant introduit dans la PR #13498 (il reste un fallback utile).
- Introduire de nouvelles capacités non sécurisées au-delà du gate `browser.evaluateEnabled` existant.
- Ajouter de l'isolation de processus (worker process/thread) pour evaluate. Si nous voyons toujours des états
  bloqués difficiles à récupérer après cette refactorisation, c'est une idée de suivi.

## Architecture actuelle (pourquoi ça se bloque)

À haut niveau :

- Les appelants envoient `act:evaluate` au service de contrôle du navigateur.
- Le gestionnaire de route appelle Playwright pour exécuter le JavaScript.
- Playwright sérialise les commandes de la page, donc un evaluate qui ne se termine jamais bloque la file.
- Une file bloquée signifie que les opérations click/type/wait ultérieures sur l'onglet peuvent sembler suspendues.

## Architecture proposée

### 1. Propagation des délais

Introduire un concept de budget unique et en dériver tout :

- L'appelant définit `timeoutMs` (ou un délai dans le futur).
- Le timeout de la requête externe, la logique du gestionnaire de route et le budget d'exécution dans la page
  utilisent tous le même budget, avec une petite marge si nécessaire pour la surcharge de sérialisation.
- L'abandon est propagé sous forme d'`AbortSignal` partout afin que l'annulation soit cohérente.

Direction d'implémentation :

- Ajouter un petit helper (par exemple `createBudget({ timeoutMs, signal })`) qui retourne :
  - `signal` : l'AbortSignal lié
  - `deadlineAtMs` : date limité absolue
  - `remainingMs()` : budget restant pour les opérations enfants
- Utiliser ce helper dans :
  - `src/browser/client-fetch.ts` (dispatch HTTP et in-process)
  - `src/node-host/runner.ts` (chemin proxy)
  - les implémentations d'actions navigateur (Playwright et CDP)

### 2. Moteur Evaluate séparé (chemin CDP)

Ajouter une implémentation evaluate basée sur CDP qui ne partage pas la file de commandes
par page de Playwright. La propriété clé est que le transport evaluate est une connexion WebSocket séparée
et une session CDP séparée attachée à la cible.

Direction d'implémentation :

- Nouveau module, par exemple `src/browser/cdp-evaluate.ts`, qui :
  - Se connecté au point de terminaison CDP configuré (socket au niveau du navigateur).
  - Utilisé `Target.attachToTarget({ targetId, flatten: true })` pour obtenir un `sessionId`.
  - Exécute soit :
    - `Runtime.evaluate` pour l'evaluate au niveau de la page, soit
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` pour l'evaluate d'élément.
  - En cas de timeout ou d'abandon :
    - Envoie `Runtime.terminateExecution` au mieux pour la session.
    - Ferme le WebSocket et retourne une erreur claire.

Notes :

- Cela exécute toujours du JavaScript dans la page, donc la terminaison peut avoir des effets de bord. L'avantage
  est que cela ne coince pas la file Playwright, et c'est annulable au niveau de la couche
  transport en tuant la session CDP.

### 3. Gestion des refs (ciblage d'éléments sans réécriture complète)

La partie difficile est le ciblage d'éléments. CDP a besoin d'un handle DOM ou d'un `backendDOMNodeId`, tandis
qu'aujourd'hui la plupart des actions navigateur utilisent des localisateurs Playwright basés sur des refs provenant de snapshots.

Approche recommandée : conserver les refs existants, mais attacher un id résolvable par CDP optionnel.

#### 3.1 Étendre les informations de ref stockées

Étendre les métadonnées de ref de rôle stockées pour inclure optionnellement un id CDP :

- Aujourd'hui : `{ rôle, name, nth }`
- Proposé : `{ rôle, name, nth, backendDOMNodeId?: number }`

Cela maintient toutes les actions Playwright existantes fonctionnelles et permet à l'evaluate CDP d'accepter
la même valeur `ref` lorsque le `backendDOMNodeId` est disponible.

#### 3.2 Remplir backendDOMNodeId au moment du snapshot

Lors de la production d'un snapshot de rôle :

1. Générer la carte de ref de rôle existante comme aujourd'hui (rôle, name, nth).
2. Récupérer l'arbre AX via CDP (`Accessibility.getFullAXTree`) et calculer une carte parallèle de
   `(rôle, name, nth) -> backendDOMNodeId` en utilisant les mêmes règles de gestion des doublons.
3. Fusionner l'id dans les informations de ref stockées pour l'onglet courant.

Si le mapping échoue pour un ref, laisser `backendDOMNodeId` indéfini. Cela rend la fonctionnalité
best-effort et sûre à déployer.

#### 3.3 Comportement d'evaluate avec ref

Dans `act:evaluate` :

- Si `ref` est présent et possède un `backendDOMNodeId`, exécuter l'evaluate d'élément via CDP.
- Si `ref` est présent mais n'a pas de `backendDOMNodeId`, se replier sur le chemin Playwright (avec
  le filet de sécurité).

Échappatoire optionnelle :

- Étendre la forme de la requête pour accepter `backendDOMNodeId` directement pour les appelants avancés (et
  pour le debug), tout en conservant `ref` comme interface principale.

### 4. Conserver un chemin de récupération de dernier recours

Même avec l'evaluate CDP, il existe d'autres moyens de coincer un onglet ou une connexion. Conserver les
mécanismes de récupération existants (terminate exécution + déconnexion Playwright) en dernier recours
pour :

- les appelants legacy
- les environnements où l'attachement CDP est bloqué
- les cas limités Playwright inattendus

## Plan d'implémentation (itération unique)

### Livrables

- Un moteur evaluate basé sur CDP qui s'exécute en dehors de la file de commandes par page de Playwright.
- Un budget unique de timeout/abandon de bout en bout utilisé de manière cohérente par les appelants et les gestionnaires.
- Des métadonnées de ref qui peuvent optionnellement porter un `backendDOMNodeId` pour l'evaluate d'élément.
- `act:evaluate` préfère le moteur CDP quand c'est possible et se replie sur Playwright sinon.
- Des tests qui prouvent qu'un evaluate bloqué ne coince pas les actions suivantes.
- Des logs/métriques qui rendent les échecs et les replis visibles.

### Checklist d'implémentation

1. Ajouter un helper « budget » partagé pour lier `timeoutMs` + `AbortSignal` amont en :
   - un seul `AbortSignal`
   - un délai absolu
   - un helper `remainingMs()` pour les opérations en aval
2. Mettre à jour tous les chemins d'appel pour utiliser ce helper afin que `timeoutMs` signifie la même chose partout :
   - `src/browser/client-fetch.ts` (dispatch HTTP et in-process)
   - `src/node-host/runner.ts` (chemin proxy nœud)
   - les wrappers CLI qui appellent `/act` (ajouter `--timeout-ms` à `browser evaluate`)
3. Implémenter `src/browser/cdp-evaluate.ts` :
   - se connecter au socket CDP au niveau du navigateur
   - `Target.attachToTarget` pour obtenir un `sessionId`
   - exécuter `Runtime.evaluate` pour l'evaluate de page
   - exécuter `DOM.resolveNode` + `Runtime.callFunctionOn` pour l'evaluate d'élément
   - en cas de timeout/abandon : `Runtime.terminateExecution` au mieux puis fermer le socket
4. Étendre les métadonnées de ref de rôle stockées pour inclure optionnellement `backendDOMNodeId` :
   - conserver le comportement existant `{ rôle, name, nth }` pour les actions Playwright
   - ajouter `backendDOMNodeId?: number` pour le ciblage d'élément CDP
5. Remplir `backendDOMNodeId` lors de la création de snapshot (best-effort) :
   - récupérer l'arbre AX via CDP (`Accessibility.getFullAXTree`)
   - calculer `(rôle, name, nth) -> backendDOMNodeId` et fusionner dans la carte de ref stockée
   - si le mapping est ambigu ou manquant, laisser l'id indéfini
6. Mettre à jour le routage de `act:evaluate` :
   - si pas de `ref` : toujours utiliser l'evaluate CDP
   - si `ref` se résout en un `backendDOMNodeId` : utiliser l'evaluate d'élément CDP
   - sinon : se replier sur l'evaluate Playwright (toujours borné et annulable)
7. Conserver le chemin de récupération « dernier recours » existant comme fallback, pas comme chemin par défaut.
8. Ajouter des tests :
   - un evaluate bloqué expire dans le budget et le click/type suivant réussit
   - l'abandon annule l'evaluate (déconnexion client ou timeout) et débloque les actions suivantes
   - les échecs de mapping se replient proprement sur Playwright
9. Ajouter de l'observabilité :
   - durée d'evaluate et compteurs de timeout
   - utilisation de terminateExecution
   - taux de repli (CDP -> Playwright) et raisons

### Critères d'acceptation

- Un `act:evaluate` délibérément bloqué retourne dans le budget de l'appelant et ne coince pas
  l'onglet pour les actions suivantes.
- `timeoutMs` se comporte de manière cohérente entre CLI, outil agent, proxy nœud et appels in-process.
- Si `ref` peut être mappé à un `backendDOMNodeId`, l'evaluate d'élément utilisé CDP ; sinon le
  chemin de repli est toujours borné et récupérable.

## Plan de test

- Tests unitaires :
  - Logique de correspondance `(rôle, name, nth)` entre les refs de rôle et les nœuds de l'arbre AX.
  - Comportement du helper budget (marge, calcul du temps restant).
- Tests d'intégration :
  - Le timeout de l'evaluate CDP retourne dans le budget et ne bloque pas l'action suivante.
  - L'abandon annule l'evaluate et déclenche la terminaison au mieux.
- Tests de contrat :
  - S'assurer que `BrowserActRequest` et `BrowserActResponse` restent compatibles.

## Risques et atténuations

- Le mapping est imparfait :
  - Atténuation : mapping best-effort, repli sur l'evaluate Playwright, et ajouter des outils de debug.
- `Runtime.terminateExecution` a des effets de bord :
  - Atténuation : utiliser uniquement en cas de timeout/abandon et documenter le comportement dans les erreurs.
- Surcharge supplémentaire :
  - Atténuation : récupérer l'arbre AX uniquement quand les snapshots sont demandés, mettre en cache par cible, et garder
    les sessions CDP de courte durée.
- Limitations du relais d'extension :
  - Atténuation : utiliser les API d'attachement au niveau du navigateur quand les sockets par page ne sont pas disponibles, et
    conserver le chemin Playwright actuel comme fallback.

## Questions ouvertes

- Le nouveau moteur devrait-il être configurable en `playwright`, `cdp` ou `auto` ?
- Voulons-nous exposer un nouveau format « nodeRef » pour les utilisateurs avancés, ou conserver uniquement `ref` ?
- Comment les snapshots de frame et les snapshots scopés par sélecteur devraient-ils participer au mapping AX ?
