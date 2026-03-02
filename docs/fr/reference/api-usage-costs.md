---
summary: "Auditer ce qui peut engendrer des coûts, quelles clés sont utilisées, et comment consulter l'utilisation"
read_when:
  - Vous voulez comprendre quelles fonctionnalités peuvent appeler des API payantes
  - Vous devez auditer les clés, les coûts et la visibilité de l'utilisation
  - Vous expliquez le reporting des coûts via /status ou /usage
title: "Utilisation et coûts des API"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/api-usage-costs.md
  workflow: manual
---

# Utilisation et coûts des API

Ce document liste les **fonctionnalités pouvant invoquer des clés API** et où leurs coûts apparaissent. Il se concentre sur les fonctionnalités OpenClaw susceptibles de générer de l'utilisation chez un fournisseur ou des appels API payants.

## Où les coûts apparaissent (chat + CLI)

**Aperçu des coûts par session**

- `/status` affiche le modèle de la session en cours, l'utilisation du contexte et les jetons de la dernière réponse.
- Si le modèle utilisé une **authentification par clé API**, `/status` affiche également le **coût estimé** de la dernière réponse.

**Pied de page de coût par message**

- `/usage full` ajouté un pied de page d'utilisation à chaque réponse, incluant le **coût estimé** (clé API uniquement).
- `/usage tokens` affiche uniquement les jetons ; les flux OAuth masquent le coût en dollars.

**Fenêtres d'utilisation CLI (quotas fournisseur)**

- `openclaw status --usage` et `openclaw channels list` affichent les **fenêtres d'utilisation** du fournisseur
  (aperçus de quotas, pas les coûts par message).

Voir [Utilisation des jetons et coûts](/reference/token-use) pour les détails et exemples.

## Comment les clés sont découvertes

OpenClaw peut récupérer les identifiants depuis :

- **Profils d'authentification** (par agent, stockés dans `auth-profiles.json`).
- **Variables d'environnement** (ex. `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
- **Configuration** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
- **Skills** (`skills.entries.<name>.apiKey`) qui peuvent exporter des clés vers l'environnement du processus du skill.

## Fonctionnalités pouvant consommer des clés

### 1) Réponses du modèle principal (chat + outils)

Chaque réponse ou appel d'outil utilise le **fournisseur de modèle actuel** (OpenAI, Anthropic, etc.). C'est la source principale d'utilisation et de coût.

Voir [Models](/providers/models) pour la configuration tarifaire et [Utilisation des jetons et coûts](/reference/token-use) pour l'affichage.

### 2) Compréhension multimédia (audio/image/vidéo)

Les médias entrants peuvent être résumés/transcrits avant l'exécution de la réponse. Cela utilise les API du modèle/fournisseur.

- Audio : OpenAI / Groq / Deepgram (désormais **activé automatiquement** lorsque les clés existent).
- Image : OpenAI / Anthropic / Google.
- Vidéo : Google.

Voir [Média understanding](/nodes/media-understanding).

### 3) Embeddings mémoire + recherche sémantique

La recherche sémantique en mémoire utilisé des **API d'embedding** lorsqu'elle est configurée pour des fournisseurs distants :

- `memorySearch.provider = "openai"` → embeddings OpenAI
- `memorySearch.provider = "gemini"` → embeddings Gemini
- `memorySearch.provider = "voyage"` → embeddings Voyage
- `memorySearch.provider = "mistral"` → embeddings Mistral
- Repli optionnel vers un fournisseur distant si les embeddings locaux échouent

Vous pouvez rester en local avec `memorySearch.provider = "local"` (pas d'utilisation API).

Voir [Memory](/concepts/memory).

### 4) Outil de recherche web (Brave / Perplexity via OpenRouter)

`web_search` utilisé des clés API et peut engendrer des frais d'utilisation :

- **Brave Search API** : `BRAVE_API_KEY` ou `tools.web.search.apiKey`
- **Perplexity** (via OpenRouter) : `PERPLEXITY_API_KEY` ou `OPENROUTER_API_KEY`

**Offre gratuite Brave (généreuse) :**

- **2 000 requêtes/mois**
- **1 requête/seconde**
- **Carte bancaire requise** pour la vérification (aucun frais sauf mise à niveau)

Voir [Web tools](/tools/web).

### 5) Outil de récupération web (Firecrawl)

`web_fetch` peut appeler **Firecrawl** lorsqu'une clé API est présente :

- `FIRECRAWL_API_KEY` ou `tools.web.fetch.firecrawl.apiKey`

Si Firecrawl n'est pas configuré, l'outil se rabat sur la récupération directe + readability (pas d'API payante).

Voir [Web tools](/tools/web).

### 6) Aperçus d'utilisation fournisseur (status/health)

Certaines commandes de statut appellent les **points de terminaison d'utilisation du fournisseur** pour afficher les fenêtres de quota ou l'état de l'authentification.
Ce sont généralement des appels à faible volume, mais ils sollicitent tout de même les API du fournisseur :

- `openclaw status --usage`
- `openclaw models status --json`

Voir [Models CLI](/cli/models).

### 7) Résumé de sauvegarde lors de la compaction

La sauvegarde de compaction peut résumer l'historique de session en utilisant le **modèle actuel**, ce qui invoque les API du fournisseur lors de son exécution.

Voir [Gestion de session + compaction](/reference/session-management-compaction).

### 8) Scan / sonde de modèle

`openclaw models scan` peut sonder les modèles OpenRouter et utilise `OPENROUTER_API_KEY` lorsque le sondage est activé.

Voir [Models CLI](/cli/models).

### 9) Talk (parole)

Le mode Talk peut invoquer **ElevenLabs** lorsqu'il est configuré :

- `ELEVENLABS_API_KEY` ou `talk.apiKey`

Voir [Talk mode](/nodes/talk).

### 10) Skills (API tierces)

Les skills peuvent stocker une `apiKey` dans `skills.entries.<name>.apiKey`. Si un skill utilisé cette clé pour des API externes, cela peut engendrer des coûts selon le fournisseur du skill.

Voir [Skills](/tools/skills).
