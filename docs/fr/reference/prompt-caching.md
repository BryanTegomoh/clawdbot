---
title: "Mise en cache des prompts"
summary: "Paramètres de mise en cache des prompts, ordre de fusion, comportement par fournisseur et schémas d'optimisation"
read_when:
  - Vous souhaitez réduire les coûts de jetons de prompt grâce à la rétention du cache
  - Vous avez besoin du comportement de cache par agent dans des configurations multi-agents
  - Vous ajustez conjointement le heartbeat et l'élagage cache-ttl
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/prompt-caching.md
  workflow: manual
---

# Mise en cache des prompts

La mise en cache des prompts signifie que le fournisseur de modèle peut réutiliser les préfixes de prompt inchangés (généralement les instructions système/développeur et autres contextes stables) d'un tour à l'autre au lieu de les retraiter à chaque fois. La première requête correspondante écrit les jetons de cache (`cacheWrite`), et les requêtes correspondantes suivantes peuvent les relire (`cacheRead`).

Pourquoi c'est important : coût de jetons réduit, réponses plus rapides et performances plus prévisibles pour les sessions longues. Sans mise en cache, les prompts répétés paient le coût complet du prompt à chaque tour même si la majorité de l'entrée n'a pas changé.

Cette page couvre tous les paramètres liés au cache qui affectent la réutilisation des prompts et le coût des jetons.

Pour les détails de tarification Anthropic, voir :
[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## Paramètres principaux

### `cacheRetention` (modèle et par agent)

Définir la rétention du cache dans les paramètres du modèle :

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

Surcharge par agent :

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

Ordre de fusion de la configuration :

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` (agent correspondant par id ; surcharge par clé)

### Legacy `cacheControlTtl`

Les valeurs legacy sont toujours acceptées et converties :

- `5m` -> `short`
- `1h` -> `long`

Préférez `cacheRetention` pour les nouvelles configurations.

### `contextPruning.mode: "cache-ttl"`

Élague l'ancien contexte de résultats d'outils après les fenêtres de TTL du cache afin que les requêtes post-inactivité ne remettent pas en cache un historique surdimensionné.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

Voir [Session Pruning](/concepts/session-pruning) pour le comportement complet.

### Maintien du cache par heartbeat

Le heartbeat peut maintenir les fenêtres de cache activés et réduire les écritures de cache répétées après des périodes d'inactivité.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

Le heartbeat par agent est pris en charge via `agents.list[].heartbeat`.

## Comportement par fournisseur

### Anthropic (API directe)

- `cacheRetention` est pris en charge.
- Avec les profils d'authentification par clé API Anthropic, OpenClaw initialisé `cacheRetention: "short"` pour les références de modèles Anthropic lorsque non défini.

### Amazon Bedrock

- Les références de modèles Claude Anthropic (`amazon-bedrock/*anthropic.claude*`) prennent en charge le passage explicite de `cacheRetention`.
- Les modèles Bedrock non-Anthropic sont forcés à `cacheRetention: "none"` à l'exécution.

### Modèles Anthropic sur OpenRouter

Pour les références de modèles `openrouter/anthropic/*`, OpenClaw injecte le `cache_control` Anthropic sur les blocs de prompt système/développeur pour améliorer la réutilisation du cache de prompt.

### Autres fournisseurs

Si le fournisseur ne prend pas en charge ce mode de cache, `cacheRetention` n'a aucun effet.

## Schémas d'optimisation

### Trafic mixte (valeur par défaut recommandée)

Maintenez une base de longue durée sur votre agent principal, désactivez la mise en cache sur les agents de notification en rafale :

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### Base orientée coût

- Définissez `cacheRetention: "short"` comme base.
- Activez `contextPruning.mode: "cache-ttl"`.
- Gardez le heartbeat en dessous de votre TTL uniquement pour les agents qui bénéficient de caches actifs.

## Diagnostics de cache

OpenClaw expose des diagnostics dédiés de trace de cache pour les exécutions d'agents intégrés.

### Configuration `diagnostics.cacheTrace`

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # optionnel
    includeMessages: false # défaut true
    includePrompt: false # défaut true
    includeSystem: false # défaut true
```

Valeurs par défaut :

- `filePath` : `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
- `includeMessages` : `true`
- `includePrompt` : `true`
- `includeSystem` : `true`

### Variables d'environnement (débogage ponctuel)

- `OPENCLAW_CACHE_TRACE=1` activé le traçage du cache.
- `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` surcharge le chemin de sortie.
- `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` activé/désactive la capture complète des messages.
- `OPENCLAW_CACHE_TRACE_PROMPT=0|1` activé/désactive la capture du texte du prompt.
- `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` activé/désactive la capture du prompt système.

### Que vérifier

- Les événements de trace de cache sont en JSONL et incluent des instantanés intermédiaires comme `session:loaded`, `prompt:before`, `stream:context` et `session:after`.
- L'impact des jetons de cache par tour est visible dans les surfaces d'utilisation normales via `cacheRead` et `cacheWrite` (par exemple `/usage full` et les résumés d'utilisation de session).

## Dépannage rapide

- `cacheWrite` élevé à la plupart des tours : vérifiez les entrées volatiles du prompt système et confirmez que le modèle/fournisseur prend en charge vos paramètres de cache.
- Pas d'effet de `cacheRetention` : confirmez que la clé du modèle correspond à `agents.defaults.models["provider/model"]`.
- Requêtes Bedrock Nova/Mistral avec paramètres de cache : forçage attendu à `none` à l'exécution.

Documentation connexe :

- [Anthropic](/providers/anthropic)
- [Token Use and Costs](/reference/token-use)
- [Session Pruning](/concepts/session-pruning)
- [Gateway Configuration Référence](/gateway/configuration-reference)
