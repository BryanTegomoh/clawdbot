---
summary: "Outils de recherche web + fetch (Brave Search API, Perplexity direct/OpenRouter, Gemini Google Search grounding)"
read_when:
  - Vous souhaitez activer web_search ou web_fetch
  - Vous avez besoin de la configuration de la clé API Brave Search
  - Vous souhaitez utiliser Perplexity Sonar pour la recherche web
  - Vous souhaitez utiliser Gemini avec le grounding Google Search
title: "Outils Web"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/web.md
  workflow: manual
---

# Outils web

OpenClaw embarque deux outils web légers :

- `web_search` : Rechercher le web via Brave Search API (défaut), Perplexity Sonar, ou Gemini avec le grounding Google Search.
- `web_fetch` : Fetch HTTP + extraction lisible (HTML → markdown/texte).

Ce ne sont **pas** de l'automatisation de navigateur. Pour les sites lourds en JS ou les connexions, utilisez l'[outil Navigateur](/tools/browser).

## Comment ça fonctionne

- `web_search` appelle votre fournisseur configuré et retourne des résultats.
  - **Brave** (défaut) : retourne des résultats structurés (titre, URL, extrait).
  - **Perplexity** : retourne des réponses synthétisées par IA avec des citations à partir d'une recherche web en temps réel.
  - **Gemini** : retourne des réponses synthétisées par IA basées sur Google Search avec des citations.
- Les résultats sont mis en cache par requête pendant 15 minutes (configurable).
- `web_fetch` effectué un GET HTTP simple et extrait le contenu lisible (HTML → markdown/texte). Il n'exécute **pas** de JavaScript.
- `web_fetch` est activé par défaut (sauf si explicitement désactivé).

## Choisir un fournisseur de recherche

| Fournisseur          | Avantages                                     | Inconvénients                             | Clé API                                      |
| -------------------- | --------------------------------------------- | ----------------------------------------- | -------------------------------------------- |
| **Brave** (défaut)   | Rapide, résultats structurés, niveau gratuit   | Résultats de recherche traditionnels       | `BRAVE_API_KEY`                              |
| **Perplexity**       | Réponses synthétisées par IA, citations, temps réel | Nécessite un accès Perplexity ou OpenRouter | `OPENROUTER_API_KEY` ou `PERPLEXITY_API_KEY` |
| **Gemini**           | Grounding Google Search, synthétisé par IA     | Nécessite une clé API Gemini              | `GEMINI_API_KEY`                             |

Voir [Configuration Brave Search](/brave-search) et [Perplexity Sonar](/perplexity) pour les détails spécifiques au fournisseur.

### Détection automatique

Si aucun `provider` n'est explicitement défini, OpenClaw détecte automatiquement quel fournisseur utiliser en fonction des clés API disponibles, vérifiant dans cet ordre :

1. **Brave** : variable d'environnement `BRAVE_API_KEY` ou configuration `search.apiKey`
2. **Gemini** : variable d'environnement `GEMINI_API_KEY` ou configuration `search.gemini.apiKey`
3. **Perplexity** : variable d'environnement `PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY` ou configuration `search.perplexity.apiKey`
4. **Grok** : variable d'environnement `XAI_API_KEY` ou configuration `search.grok.apiKey`

Si aucune clé n'est trouvée, il se rabat sur Brave (vous obtiendrez une erreur de clé manquante vous invitant à en configurer une).

### Fournisseur explicite

Définissez le fournisseur dans la configuration :

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave", // ou "perplexity" ou "gemini"
      },
    },
  },
}
```

Exemple : basculer vers Perplexity Sonar (API directe) :

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Obtenir une clé API Brave

1. Créez un compte Brave Search API sur [https://brave.com/search/api/](https://brave.com/search/api/)
2. Dans le tableau de bord, choisissez le plan **Data for Search** (pas « Data for AI ») et générez une clé API.
3. Exécutez `openclaw configuré --section web` pour stocker la clé dans la configuration (recommandé), ou définissez `BRAVE_API_KEY` dans votre environnement.

Brave fournit un niveau gratuit plus des plans payants ; consultez le portail API Brave pour les limités et tarifs actuels.

### Où définir la clé (recommandé)

**Recommandé :** exécutez `openclaw configuré --section web`. Cela stocke la clé dans `~/.openclaw/openclaw.json` sous `tools.web.search.apiKey`.

**Alternative via l'environnement :** définissez `BRAVE_API_KEY` dans l'environnement du processus Gateway. Pour une installation gateway, mettez-la dans `~/.openclaw/.env` (ou l'environnement de votre service). Voir [Variables d'environnement](/help/faq#how-does-openclaw-load-environment-variables).

## Utiliser Perplexity (direct ou via OpenRouter)

Les modèles Perplexity Sonar ont des capacités de recherche web intégrées et retournent des réponses synthétisées par IA avec des citations. Vous pouvez les utiliser via OpenRouter (pas de carte de crédit requise, supporte crypto/prépayé).

### Obtenir une clé API OpenRouter

1. Créez un compte sur [https://openrouter.ai/](https://openrouter.ai/)
2. Ajoutez des crédits (supporte crypto, prépayé, ou carte de crédit)
3. Générez une clé API dans les paramètres de votre compte

### Configurer la recherche Perplexity

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // Clé API (optionnel si OPENROUTER_API_KEY ou PERPLEXITY_API_KEY est défini)
          apiKey: "sk-or-v1-...",
          // URL de base (défaut basé sur la clé si omis)
          baseUrl: "https://openrouter.ai/api/v1",
          // Modèle (défaut : perplexity/sonar-pro)
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

**Alternative via l'environnement :** définissez `OPENROUTER_API_KEY` ou `PERPLEXITY_API_KEY` dans l'environnement du Gateway. Pour une installation gateway, mettez-la dans `~/.openclaw/.env`.

Si aucune URL de base n'est définie, OpenClaw choisit un défaut basé sur la source de la clé API :

- `PERPLEXITY_API_KEY` ou `pplx-...` → `https://api.perplexity.ai`
- `OPENROUTER_API_KEY` ou `sk-or-...` → `https://openrouter.ai/api/v1`
- Formats de clés inconnus → OpenRouter (repli sûr)

### Modèles Perplexity disponibles

| Modèle                           | Description                              | Idéal pour              |
| -------------------------------- | ---------------------------------------- | ----------------------- |
| `perplexity/sonar`               | Q&R rapide avec recherche web             | Recherches rapides      |
| `perplexity/sonar-pro` (défaut)  | Raisonnement multi-étapes avec recherche web | Questions complexes     |
| `perplexity/sonar-reasoning-pro` | Analyse en chaîne de pensée               | Recherche approfondie   |

## Utiliser Gemini (grounding Google Search)

Les modèles Gemini supportent le [grounding Google Search](https://ai.google.dev/gemini-api/docs/grounding) intégré, qui retourne des réponses synthétisées par IA soutenues par des résultats Google Search en direct avec des citations.

### Obtenir une clé API Gemini

1. Allez sur [Google AI Studio](https://aistudio.google.com/apikey)
2. Créez une clé API
3. Définissez `GEMINI_API_KEY` dans l'environnement du Gateway, ou configurez `tools.web.search.gemini.apiKey`

### Configurer la recherche Gemini

```json5
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // Clé API (optionnel si GEMINI_API_KEY est défini)
          apiKey: "AIza...",
          // Modèle (défaut : "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**Alternative via l'environnement :** définissez `GEMINI_API_KEY` dans l'environnement du Gateway. Pour une installation gateway, mettez-la dans `~/.openclaw/.env`.

### Notes

- Les URLs de citation du grounding Gemini sont automatiquement résolues depuis les URLs de redirection de Google vers des URLs directes.
- La résolution de redirection utilise le chemin de garde SSRF (HEAD + vérification des redirections + validation http/https) avant de retourner l'URL de citation finale.
- Ce résolveur de redirections suit le modèle de réseau de confiance (réseaux privés/internes autorisés par défaut) pour correspondre aux hypothèses de confiance de l'opérateur du Gateway.
- Le modèle par défaut (`gemini-2.5-flash`) est rapide et rentable. Tout modèle Gemini supportant le grounding peut être utilisé.

## web_search

Rechercher le web en utilisant votre fournisseur configuré.

### Prérequis

- `tools.web.search.enabled` ne doit pas être `false` (défaut : activé)
- Clé API pour votre fournisseur choisi :
  - **Brave** : `BRAVE_API_KEY` ou `tools.web.search.apiKey`
  - **Perplexity** : `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY`, ou `tools.web.search.perplexity.apiKey`

### Configuration

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // optionnel si BRAVE_API_KEY est défini
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### Paramètres de l'outil

- `query` (obligatoire)
- `count` (1-10 ; défaut depuis la configuration)
- `country` (optionnel) : code pays à 2 lettres pour des résultats régionaux (par ex. « DE », « US », « ALL »). Si omis, Brave choisit sa région par défaut.
- `search_lang` (optionnel) : code de langue ISO pour les résultats de recherche (par ex. « de », « en », « fr »)
- `ui_lang` (optionnel) : code de langue ISO pour les éléments d'interface
- `freshness` (optionnel) : filtrer par date de découverte
  - Brave : `pd`, `pw`, `pm`, `py`, ou `YYYY-MM-DDtoYYYY-MM-DD`
  - Perplexity : `pd`, `pw`, `pm`, `py`

**Exemples :**

```javascript
// Recherche spécifique à l'Allemagne
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de",
});

// Recherche française avec interface française
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr",
});

// Résultats récents (semaine passée)
await web_search({
  query: "TMBG interview",
  freshness: "pw",
});
```

## web_fetch

Récupérer une URL et extraire le contenu lisible.

### Prérequis de web_fetch

- `tools.web.fetch.enabled` ne doit pas être `false` (défaut : activé)
- Repli Firecrawl optionnel : définissez `tools.web.fetch.firecrawl.apiKey` ou `FIRECRAWL_API_KEY`.

### Configuration de web_fetch

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // optionnel si FIRECRAWL_API_KEY est défini
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 jour)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### Paramètres de l'outil web_fetch

- `url` (obligatoire, http/https uniquement)
- `extractMode` (`markdown` | `text`)
- `maxChars` (tronquer les pages longues)

Notes :

- `web_fetch` utilisé Readability (extraction du contenu principal) d'abord, puis Firecrawl (si configuré). Si les deux échouent, l'outil retourne une erreur.
- Les requêtes Firecrawl utilisent le mode de contournement de bot et mettent en cache les résultats par défaut.
- `web_fetch` envoie un User-Agent similaire à Chrome et `Accept-Language` par défaut ; substituez `userAgent` si nécessaire.
- `web_fetch` bloque les noms d'hôte privés/internes et re-vérifie les redirections (limité avec `maxRedirects`).
- `maxChars` est limité à `tools.web.fetch.maxCharsCap`.
- `web_fetch` limité la taille du corps de réponse téléchargé à `tools.web.fetch.maxResponseBytes` avant l'analyse ; les réponses trop grandes sont tronquées et incluent un avertissement.
- `web_fetch` est une extraction best-effort ; certains sites nécessiteront l'outil navigateur.
- Voir [Firecrawl](/tools/firecrawl) pour la configuration de la clé et les détails du service.
- Les réponses sont mises en cache (15 minutes par défaut) pour réduire les récupérations répétées.
- Si vous utilisez des profils/listes d'autorisation d'outils, ajoutez `web_search`/`web_fetch` ou `group:web`.
- Si la clé Brave est manquante, `web_search` retourne un court indice de configuration avec un lien vers la documentation.
