---
summary: "Cycle de vie de la boucle agent, streams et sémantique d'attente"
read_when:
  - Vous avez besoin d'un parcours exact de la boucle agent ou des événements de cycle de vie
title: "Boucle Agent"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/agent-loop.md
  workflow: manual
---

# Boucle Agent (OpenClaw)

Une boucle agentique est l'exécution complète « réelle » d'un agent : réception → assemblage du contexte → inférence du modèle → exécution des outils → streaming des réponses → persistance. C'est le chemin autoritaire qui transforme un message en actions et une réponse finale, tout en maintenant l'état de la session cohérent.

Dans OpenClaw, une boucle est une exécution unique et sérialisée par session qui émet des événements de cycle de vie et de stream pendant que le modèle réfléchit, appelle des outils et produit du texte en sortie. Ce document explique comment cette boucle authentique est câblée de bout en bout.

## Points d'entrée

- RPC Gateway : `agent` et `agent.wait`.
- CLI : commande `agent`.

## Fonctionnement (vue d'ensemble)

1. Le RPC `agent` valide les paramètres, résout la session (sessionKey/sessionId), persiste les métadonnées de session, retourne `{ runId, acceptedAt }` immédiatement.
2. `agentCommand` exécute l'agent :
   - résout les valeurs par défaut du modèle + thinking/verbose
   - charge un snapshot des skills
   - appelle `runEmbeddedPiAgent` (runtime pi-agent-core)
   - émet un **lifecycle end/error** si la boucle intégrée n'en émet pas
3. `runEmbeddedPiAgent` :
   - sérialise les exécutions via des files par session + globales
   - résout le modèle + profil d'authentification et construit la session pi
   - s'abonne aux événements pi et diffuse en continu les deltas assistant/outil
   - applique le timeout -> abandonne l'exécution si dépassé
   - retourne les charges utiles + métadonnées d'utilisation
4. `subscribeEmbeddedPiSession` fait le pont entre les événements pi-agent-core et le stream `agent` d'OpenClaw :
   - événements d'outils => `stream: "tool"`
   - deltas assistant => `stream: "assistant"`
   - événements de cycle de vie => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5. `agent.wait` utilisé `waitForAgentJob` :
   - attend le **lifecycle end/error** pour le `runId`
   - retourne `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## File d'attente + concurrence

- Les exécutions sont sérialisées par clé de session (voie de session) et optionnellement via une voie globale.
- Cela empêche les courses d'outils/sessions et maintient l'historique de session cohérent.
- Les canaux de messagerie peuvent choisir des modes de file d'attente (collect/steer/followup) qui alimentent ce système de voies.
  Voir [File d'attente de commandes](/concepts/queue).

## Préparation session + espace de travail

- L'espace de travail est résolu et crée ; les exécutions en bac à sable peuvent rediriger vers une racine d'espace de travail sandbox.
- Les skills sont chargés (ou réutilisés depuis un snapshot) et injectés dans l'environnement et le prompt.
- Les fichiers de bootstrap/contexte sont résolus et injectés dans le rapport du prompt système.
- Un verrou d'écriture de session est acquis ; le `SessionManager` est ouvert et préparé avant le streaming.

## Assemblage du prompt + prompt système

- Le prompt système est construit à partir du prompt de base d'OpenClaw, du prompt des skills, du contexte bootstrap et des surcharges par exécution.
- Les limités spécifiques au modèle et les tokens de réserve pour la compaction sont appliqués.
- Voir [Prompt système](/concepts/system-prompt) pour ce que le modèle voit.

## Points d'accroche (où vous pouvez intercepter)

OpenClaw dispose de deux systèmes de hooks :

- **Hooks internes** (hooks Gateway) : scripts pilotés par événements pour les commandes et événements de cycle de vie.
- **Hooks de plugins** : points d'extension dans le cycle de vie agent/outil et le pipeline du Gateway.

### Hooks internes (hooks Gateway)

- **`agent:bootstrap`** : s'exécute pendant la construction des fichiers bootstrap avant la finalisation du prompt système.
  Utilisez ceci pour ajouter/supprimer des fichiers de contexte bootstrap.
- **Hooks de commandes** : `/new`, `/reset`, `/stop` et autres événements de commandes (voir la doc Hooks).

Voir [Hooks](/automation/hooks) pour la configuration et les exemples.

### Hooks de plugins (cycle de vie agent + Gateway)

Ceux-ci s'exécutent à l'intérieur de la boucle agent ou du pipeline Gateway :

- **`before_model_resolve`** : s'exécute avant la session (pas de `messages`) pour surcharger de manière déterministe le fournisseur/modèle avant la résolution du modèle.
- **`before_prompt_build`** : s'exécute après le chargement de la session (avec `messages`) pour injecter `prependContext`/`systemPrompt` avant la soumission du prompt.
- **`before_agent_start`** : hook de compatibilité hérité qui peut s'exécuter dans l'une ou l'autre phase ; préférez les hooks explicites ci-dessus.
- **`agent_end`** : inspecte la liste finale de messages et les métadonnées d'exécution après complétion.
- **`before_compaction` / `after_compaction`** : observe ou annote les cycles de compaction.
- **`before_tool_call` / `after_tool_call`** : intercepte les paramètres/résultats des outils.
- **`tool_result_persist`** : transforme de manière synchrone les résultats d'outils avant leur écriture dans la transcription de session.
- **`message_received` / `message_sending` / `message_sent`** : hooks de messages entrants + sortants.
- **`session_start` / `session_end`** : limités du cycle de vie de session.
- **`gateway_start` / `gateway_stop`** : événements du cycle de vie du Gateway.

Voir [Plugins](/tools/plugin#plugin-hooks) pour l'API des hooks et les détails d'enregistrement.

## Streaming + réponses partielles

- Les deltas assistant sont diffusés en continu depuis pi-agent-core et émis comme événements `assistant`.
- Le streaming par blocs peut émettre des réponses partielles soit sur `text_end` soit sur `message_end`.
- Le streaming de raisonnement peut être émis comme un stream séparé ou comme réponses par blocs.
- Voir [Streaming](/concepts/streaming) pour le comportement de découpage et de réponse par blocs.

## Exécution des outils + outils de messagerie

- Les événements de démarrage/mise à jour/fin d'outil sont émis sur le stream `tool`.
- Les résultats d'outils sont assainis pour la taille et les charges utiles d'images avant journalisation/émission.
- Les envois par outil de messagerie sont suivis pour supprimer les confirmations assistant en double.

## Mise en forme des réponses + suppression

- Les charges utiles finales sont assemblées à partir de :
  - texte assistant (et raisonnement optionnel)
  - résumés d'outils en ligne (quand verbose + autorisé)
  - texte d'erreur assistant quand le modèle échoue
- `NO_REPLY` est traité comme un jeton silencieux et filtre des charges utiles sortantes.
- Les doublons d'outils de messagerie sont retirés de la liste de charges utiles finales.
- Si aucune charge utile affichable ne reste et qu'un outil a échoué, une réponse d'erreur d'outil de secours est émise (sauf si un outil de messagerie a déjà envoyé une réponse visible à l'utilisateur).

## Compaction + nouvelles tentatives

- La compaction automatique émet des événements de stream `compaction` et peut déclencher une nouvelle tentative.
- Lors d'une nouvelle tentative, les tampons en mémoire et les résumés d'outils sont réinitialisés pour éviter les sorties en double.
- Voir [Compaction](/concepts/compaction) pour le pipeline de compaction.

## Streams d'événements (aujourd'hui)

- `lifecycle` : émis par `subscribeEmbeddedPiSession` (et en secours par `agentCommand`)
- `assistant` : deltas diffusés en continu depuis pi-agent-core
- `tool` : événements d'outils diffusés en continu depuis pi-agent-core

## Gestion des canaux de chat

- Les deltas assistant sont mis en mémoire tampon dans des messages chat `delta`.
- Un chat `final` est émis lors du **lifecycle end/error**.

## Timeouts

- Valeur par défaut de `agent.wait` : 30 s (juste l'attente). Le paramètre `timeoutMs` surcharge.
- Runtime de l'agent : `agents.defaults.timeoutSeconds` par défaut 600 s ; appliqué dans le timer d'abandon de `runEmbeddedPiAgent`.

## Situations d'arrêt prématuré

- Timeout de l'agent (abandon)
- AbortSignal (annulation)
- Déconnexion du Gateway ou timeout RPC
- Timeout de `agent.wait` (attente uniquement, n'arrête pas l'agent)
