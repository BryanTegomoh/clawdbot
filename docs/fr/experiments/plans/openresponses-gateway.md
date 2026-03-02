---
summary: "Plan : Ajouter le endpoint OpenResponses /v1/responses et déprécier proprement les chat completions"
read_when:
  - Conception ou implémentation du support Gateway pour `/v1/responses`
  - Planification de la migration depuis la compatibilité Chat Completions
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
title: "Plan Gateway OpenResponses"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/experiments/plans/openresponses-gateway.md
  workflow: manual
---

# Plan d'intégration Gateway OpenResponses

## Contexte

Le Gateway OpenClaw expose actuellement un endpoint minimal compatible OpenAI Chat Completions à
`/v1/chat/completions` (voir [API HTTP OpenAI Chat Completions](/gateway/openai-http-api)).

Open Responses est un standard d'inférence ouvert basé sur l'API Responses d'OpenAI. Il est conçu
pour les workflows agentiques et utilise des entrées basées sur des items plus des événements de streaming sémantiques. La spécification
OpenResponses définit `/v1/responses`, pas `/v1/chat/completions`.

## Objectifs

- Ajouter un endpoint `/v1/responses` qui adhère à la sémantique OpenResponses.
- Conserver Chat Completions comme couche de compatibilité, facile à désactiver et éventuellement à supprimer.
- Standardiser la validation et le parsing avec des schémas isolés et réutilisables.

## Hors périmètre

- Parité complète des fonctionnalités OpenResponses dans la première passe (images, fichiers, outils hébergés).
- Remplacement de la logique interne d'exécution d'agent ou d'orchestration d'outils.
- Modification du comportement existant de `/v1/chat/completions` durant la première phase.

## Résumé de la recherche

Sources : OpenAPI OpenResponses, site de spécification OpenResponses et article de blog Hugging Face.

Points clés extraits :

- `POST /v1/responses` accepte les champs `CreateResponseBody` comme `model`, `input` (chaîne ou
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` et
  `max_tool_calls`.
- `ItemParam` est une union discriminée de :
  - items `message` avec les rôles `system`, `developer`, `user`, `assistant`
  - `function_call` et `function_call_output`
  - `reasoning`
  - `item_reference`
- Les réponses réussies retournent un `ResponseResource` avec `object: "response"`, `status` et
  items `output`.
- Le streaming utilisé des événements sémantiques tels que :
  - `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  - `response.output_item.added`, `response.output_item.done`
  - `response.content_part.added`, `response.content_part.done`
  - `response.output_text.delta`, `response.output_text.done`
- La spécification requiert :
  - `Content-Type: text/event-stream`
  - `event:` doit correspondre au champ JSON `type`
  - l'événement terminal doit être le littéral `[DONE]`
- Les items de raisonnement peuvent exposer `content`, `encrypted_content` et `summary`.
- Les exemples HF incluent `OpenResponses-Version: latest` dans les requêtes (en-tête optionnel).

## Architecture proposée

- Ajouter `src/gateway/open-responses.schema.ts` contenant uniquement les schémas Zod (pas d'imports Gateway).
- Ajouter `src/gateway/openresponses-http.ts` (ou `open-responses-http.ts`) pour `/v1/responses`.
- Conserver `src/gateway/openai-http.ts` intact comme adaptateur de compatibilité legacy.
- Ajouter la config `gateway.http.endpoints.responses.enabled` (par défaut `false`).
- Conserver `gateway.http.endpoints.chatCompletions.enabled` indépendant ; permettre aux deux endpoints d'être
  activés/désactivés séparément.
- Émettre un avertissement au démarrage quand Chat Completions est activé pour signaler le statut legacy.

## Chemin de dépréciation pour Chat Completions

- Maintenir des frontières de modules strictes : aucun type de schéma partagé entre responses et chat completions.
- Rendre Chat Completions opt-in par config pour pouvoir le désactiver sans changement de code.
- Mettre à jour la documentation pour étiqueter Chat Completions comme legacy une fois `/v1/responses` stable.
- Étape future optionnelle : mapper les requêtes Chat Completions vers le handler Responses pour un chemin
  de suppression plus simple.

## Sous-ensemble supporté en Phase 1

- Accepter `input` comme chaîne ou `ItemParam[]` avec les rôles de message et `function_call_output`.
- Extraire les messages system et developer dans `extraSystemPrompt`.
- Utiliser le message `user` ou `function_call_output` le plus récent comme message courant pour les exécutions d'agent.
- Rejeter les parties de contenu non supportées (image/fichier) avec `invalid_request_error`.
- Retourner un seul message assistant avec du contenu `output_text`.
- Retourner `usage` avec des valeurs à zéro jusqu'à ce que la comptabilité des tokens soit connectée.

## Stratégie de validation (sans SDK)

- Implémenter des schémas Zod pour le sous-ensemble supporté de :
  - `CreateResponseBody`
  - `ItemParam` + unions de parties de contenu de message
  - `ResponseResource`
  - Formes d'événements de streaming utilisées par le Gateway
- Conserver les schémas dans un seul module isolé pour éviter la dérive et permettre la future génération de code.

## Implémentation du streaming (Phase 1)

- Lignes SSE avec à la fois `event:` et `data:`.
- Séquence requise (minimum viable) :
  - `response.created`
  - `response.output_item.added`
  - `response.content_part.added`
  - `response.output_text.delta` (répéter selon le besoin)
  - `response.output_text.done`
  - `response.content_part.done`
  - `response.completed`
  - `[DONE]`

## Plan de tests et vérification

- Ajouter une couverture e2e pour `/v1/responses` :
  - Authentification requise
  - Forme de réponse non-stream
  - Ordre des événements de stream et `[DONE]`
  - Routage de session avec en-têtes et `user`
- Conserver `src/gateway/openai-http.test.ts` inchangé.
- Manuel : curl vers `/v1/responses` avec `stream: true` et vérifier l'ordre des événements et le
  `[DONE]` terminal.

## Mises à jour de la documentation (suivi)

- Ajouter une nouvelle page de documentation pour l'utilisation et les exemples de `/v1/responses`.
- Mettre à jour `/gateway/openai-http-api` avec une note legacy et un pointeur vers `/v1/responses`.
