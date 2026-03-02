---
summary: "Exécuter OpenClaw avec des LLMs locaux (LM Studio, vLLM, LiteLLM, endpoints OpenAI personnalisés)"
read_when:
  - Vous souhaitez servir des modèles depuis votre propre machine GPU
  - Vous connectez LM Studio ou un proxy compatible OpenAI
  - Vous avez besoin des conseils les plus sûrs pour les modèles locaux
title: "Modèles locaux"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/local-models.md
  workflow: manual
---

# Modèles locaux

Le local est faisable, mais OpenClaw attend un contexte large + des défenses solides contre l'injection de prompt. Les petites cartes tronquent le contexte et fuient la sécurité. Visez haut : **≥2 Mac Studios poussés au maximum ou rig GPU équivalent (~30k$+)**. Un seul GPU de **24 Go** fonctionne uniquement pour des prompts plus légers avec une latence plus élevée. Utilisez la **variante de modèle la plus grande / taille complète que vous pouvez exécuter** ; les checkpoints agressivement quantifiés ou « small » augmentent le risque d'injection de prompt (voir [Sécurité](/gateway/security)).

## Recommandé : LM Studio + MiniMax M2.1 (Responses API, taille complète)

Meilleure stack locale actuelle. Chargez MiniMax M2.1 dans LM Studio, activez le serveur local (par défaut `http://127.0.0.1:1234`), et utilisez Responses API pour séparer le raisonnement du texte final.

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

**Checklist d'installation**

- Installez LM Studio : [https://lmstudio.ai](https://lmstudio.ai)
- Dans LM Studio, téléchargez le **build MiniMax M2.1 le plus grand disponible** (évitez les variantes « small »/fortement quantifiées), démarrez le serveur, confirmez que `http://127.0.0.1:1234/v1/models` le liste.
- Gardez le modèle chargé ; le chargement à froid ajouté de la latence au démarrage.
- Ajustez `contextWindow`/`maxTokens` si votre build LM Studio diffère.
- Pour WhatsApp, restez avec Responses API pour que seul le texte final soit envoyé.

Gardez les modèles hébergés configurés même en exécutant du local ; utilisez `models.mode: "merge"` pour que les replis restent disponibles.

### Configuration hybride : hébergé en principal, local en repli

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-6"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-6": { alias: "Opus" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### Local d'abord avec filet de sécurité hébergé

Inversez l'ordre principal et repli ; gardez le même bloc providers et `models.mode: "merge"` pour pouvoir revenir à Sonnet ou Opus lorsque la machine locale est hors service.

### Hébergement régional / routage des données

- Les variantes hébergées MiniMax/Kimi/GLM existent aussi sur OpenRouter avec des endpoints épinglés par région (par ex. hébergés aux US). Choisissez la variante régionale pour garder le trafic dans votre juridiction choisie tout en utilisant `models.mode: "merge"` pour les replis Anthropic/OpenAI.
- Le local uniquement reste le chemin de confidentialité le plus fort ; le routage régional hébergé est le compromis lorsque vous avez besoin des fonctionnalités fournisseur mais voulez contrôler le flux de données.

## Autres proxys locaux compatibles OpenAI

vLLM, LiteLLM, OAI-proxy, ou les gateways personnalisés fonctionnent s'ils exposent un endpoint de style OpenAI `/v1`. Remplacez le bloc provider ci-dessus par votre endpoint et identifiant de modèle :

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Gardez `models.mode: "merge"` pour que les modèles hébergés restent disponibles comme replis.

## Dépannage

- Le gateway peut atteindre le proxy ? `curl http://127.0.0.1:1234/v1/models`.
- Modèle LM Studio déchargé ? Rechargez ; le démarrage à froid est une cause courante de « blocage ».
- Erreurs de contexte ? Abaissez `contextWindow` ou augmentez la limité de votre serveur.
- Sécurité : les modèles locaux contournent les filtrés côté fournisseur ; gardez les agents étroits et la compaction activée pour limiter le rayon d'explosion de l'injection de prompt.
