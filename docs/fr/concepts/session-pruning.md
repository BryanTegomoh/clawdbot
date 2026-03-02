---
title: "Élagage de session"
summary: "Élagage de session : troncature des résultats d'outils pour réduire la surcharge de contexte"
read_when:
  - Vous souhaitez réduire la croissance du contexte LLM due aux sorties d'outils
  - Vous ajustez agents.defaults.contextPruning
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/session-pruning.md
  workflow: manual
---

# Élagage de session

L'élagage de session tronque les **anciens résultats d'outils** du contexte en mémoire juste avant chaque appel LLM. Il ne **réécrit pas** l'historique de session sur disque (`*.jsonl`).

## Quand il s'exécute

- Lorsque `mode: "cache-ttl"` est activé et que le dernier appel Anthropic pour la session est plus ancien que `ttl`.
- N'affecte que les messages envoyés au modèle pour cette requête.
- Actif uniquement pour les appels API Anthropic (et les modèles Anthropic via OpenRouter).
- Pour de meilleurs résultats, faites correspondre `ttl` avec votre politique `cacheRetention` du modèle (`short` = 5 min, `long` = 1 h).
- Après un élagage, la fenêtre TTL est réinitialisée afin que les requêtes suivantes conservent le cache jusqu'à ce que `ttl` expire à nouveau.

## Valeurs par défaut intelligentes (Anthropic)

- Profils **OAuth ou setup-token** : activent l'élagage `cache-ttl` et définissent le heartbeat a `1h`.
- Profils **clé API** : activent l'élagage `cache-ttl`, définissent le heartbeat a `30m` et utilisent par défaut `cacheRetention: "short"` sur les modèles Anthropic.
- Si vous définissez explicitement l'une de ces valeurs, OpenClaw ne les **remplace pas**.

## Ce que cela améliore (coût + comportement du cache)

- **Pourquoi elaguer :** le cache de prompt Anthropic ne s'applique que pendant le TTL. Si une session reste inactive au-delà du TTL, la requête suivante re-met en cache le prompt complet sauf si vous le tronquez d'abord.
- **Ce qui devient moins cher :** l'élagage réduit la taille de **cacheWrite** pour cette première requête après l'expiration du TTL.
- **Pourquoi la réinitialisation du TTL est importante :** une fois l'élagage exécuté, la fenêtre de cache est réinitialisée, donc les requêtes suivantes peuvent réutiliser le prompt fraîchement mis en cache au lieu de re-mettre en cache l'historique complet à nouveau.
- **Ce que cela ne fait pas :** l'élagage n'ajouté pas de tokens ni ne « double » les coûts ; il change uniquement ce qui est mis en cache lors de cette première requête post-TTL.

## Ce qui peut être élagué

- Uniquement les messages `toolResult`.
- Les messages utilisateur + assistant ne sont **jamais** modifiés.
- Les derniers `keepLastAssistants` messages assistant sont protégés ; les résultats d'outils après ce seuil ne sont pas élagués.
- S'il n'y a pas assez de messages assistant pour établir le seuil, l'élagage est ignoré.
- Les résultats d'outils contenant des **blocs image** sont ignorés (jamais tronqués/supprimés).

## Estimation de la fenêtre de contexte

L'élagage utilisé une fenêtre de contexte estimée (caractères ≈ tokens x 4). La fenêtre de base est résolue dans cet ordre :

1. Surcharge `models.providers.*.models[].contextWindow`.
2. `contextWindow` de la définition du modèle (depuis le registre de modèles).
3. Défaut de `200000` tokens.

Si `agents.defaults.contextTokens` est défini, il est traité comme un plafond (min) sur la fenêtre résolue.

## Mode

### cache-ttl

- L'élagage ne s'exécute que si le dernier appel Anthropic est plus ancien que `ttl` (par défaut `5m`).
- Lorsqu'il s'exécute : même comportement de troncature douce + suppression nette qu'avant.

## Élagage doux vs dur

- **Troncature douce** : uniquement pour les résultats d'outils surdimensionnés.
  - Conserve le début + la fin, insère `...`, et ajoute une note avec la taille originale.
  - Ignoré les résultats avec des blocs image.
- **Suppression nette** : remplace l'intégralité du résultat d'outil par `hardClear.placeholder`.

## Sélection des outils

- `tools.allow` / `tools.deny` supportent les jokers `*`.
- Le refus l'emporte.
- La correspondance est insensible à la casse.
- Liste d'autorisation vide => tous les outils sont autorisés.

## Interaction avec les autres limités

- Les outils intégrés tronquent déjà leurs propres sorties ; l'élagage de session est une couche supplémentaire qui empêche les chats de longue durée d'accumuler trop de sorties d'outils dans le contexte du modèle.
- La compaction est séparée : la compaction résume et persiste, l'élagage est transitoire par requête. Voir [/concepts/compaction](/concepts/compaction).

## Valeurs par défaut (lorsque activé)

- `ttl` : `"5m"`
- `keepLastAssistants` : `3`
- `softTrimRatio` : `0.3`
- `hardClearRatio` : `0.5`
- `minPrunableToolChars` : `50000`
- `softTrim` : `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
- `hardClear` : `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

## Exemples

Défaut (désactivé) :

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

Activer l'élagage sensible au TTL :

```json5
{
  agents: { defaults: { contextPruning: { mode: "cache-ttl", ttl: "5m" } } },
}
```

Restreindre l'élagage à des outils spécifiques :

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl",
        tools: { allow: ["exec", "read"], deny: ["*image*"] },
      },
    },
  },
}
```

Voir la référence de configuration : [Configuration du Gateway](/gateway/configuration)
