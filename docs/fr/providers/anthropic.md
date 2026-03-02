---
summary: "Utiliser Anthropic Claude via des clés API ou un setup-token dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles Anthropic dans OpenClaw
  - Vous préférez un setup-token plutôt que des clés API
title: "Anthropic"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/anthropic.md
  workflow: manual
---

# Anthropic (Claude)

Anthropic développe la famille de modèles **Claude** et y donne accès via une API.
Dans OpenClaw, vous pouvez vous authentifier avec une clé API ou un **setup-token**.

## Option A : Clé API Anthropic

**Idéal pour :** l'accès standard à l'API avec facturation à l'usage.
Créez votre clé API dans la Console Anthropic.

### Configuration CLI

```bash
openclaw onboard
# choisissez : Anthropic API key

# ou en mode non interactif
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### Extrait de configuration

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Mise en cache des prompts (API Anthropic)

OpenClaw prend en charge la fonctionnalité de mise en cache des prompts d'Anthropic. Elle est réservée à l'**API** ; l'authentification par abonnement ne respecte pas les paramètres de cache.

### Configuration

Utilisez le paramètre `cacheRetention` dans la configuration de votre modèle :

| Valeur  | Durée du cache | Description                                    |
| ------- | -------------- | ---------------------------------------------- |
| `none`  | Pas de cache   | Désactiver la mise en cache des prompts        |
| `short` | 5 minutes      | Par défaut pour l'authentification par clé API |
| `long`  | 1 heure        | Cache étendu (nécessite le flag beta)          |

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### Valeurs par défaut

Lors de l'utilisation de l'authentification par clé API Anthropic, OpenClaw applique automatiquement `cacheRetention: "short"` (cache de 5 minutes) pour tous les modèles Anthropic. Vous pouvez remplacer ce comportement en définissant explicitement `cacheRetention` dans votre configuration.

### Remplacements de cacheRetention par agent

Utilisez les paramètres au niveau du modèle comme base, puis surchargez pour des agents spécifiques via `agents.list[].params`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // base pour la plupart des agents
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // surcharge pour cet agent uniquement
    ],
  },
}
```

Ordre de fusion de la configuration pour les paramètres de cache :

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` (correspondant à `id`, surcharge par clé)

Cela permet à un agent de conserver un cache de longue durée tandis qu'un autre agent sur le même modèle désactive la mise en cache pour éviter les coûts d'écriture sur du trafic en rafale ou peu réutilisé.

### Notes sur Bedrock Claude

- Les modèles Anthropic Claude sur Bedrock (`amazon-bedrock/*anthropic.claude*`) acceptent le passage `cacheRetention` lorsqu'il est configuré.
- Les modèles Bedrock non-Anthropic sont forcés à `cacheRetention: "none"` à l'exécution.
- Les valeurs par défaut intelligentes de la clé API Anthropic initialisent également `cacheRetention: "short"` pour les références de modèles Claude-sur-Bedrock lorsqu'aucune valeur explicite n'est définie.

### Paramètre ancien

L'ancien paramètre `cacheControlTtl` est toujours pris en charge pour la compatibilité ascendante :

- `"5m"` correspond à `short`
- `"1h"` correspond à `long`

Nous recommandons de migrer vers le nouveau paramètre `cacheRetention`.

OpenClaw inclut le flag beta `extended-cache-ttl-2025-04-11` pour les requêtes API Anthropic ; conservez-le si vous surchargez les en-têtes du fournisseur (voir [/gateway/configuration](/gateway/configuration)).

## Fenêtre de contexte 1M (beta Anthropic)

La fenêtre de contexte 1M d'Anthropic est conditionnée par un flag beta. Dans OpenClaw, activez-la par modèle avec `params.context1m: true` pour les modèles Opus/Sonnet pris en charge.

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw traduit cela en `anthropic-beta: context-1m-2025-08-07` sur les requêtes Anthropic.

Note : Anthropic rejette actuellement les requêtes beta `context-1m-*` lors de l'utilisation de tokens OAuth/abonnement (`sk-ant-oat-*`). OpenClaw ignore automatiquement l'en-tête beta context1m pour l'authentification OAuth et conserve les betas OAuth requises.

## Option B : Setup-token Claude

**Idéal pour :** utiliser votre abonnement Claude.

### Où obtenir un setup-token

Les setup-tokens sont créés par le **CLI Claude Code**, pas par la Console Anthropic. Vous pouvez exécuter ceci sur **n'importe quelle machine** :

```bash
claude setup-token
```

Collez le token dans OpenClaw (assistant : **Anthropic token (paste setup-token)**), ou exécutez-le sur l'hôte du gateway :

```bash
openclaw models auth setup-token --provider anthropic
```

Si vous avez généré le token sur une autre machine, collez-le :

```bash
openclaw models auth paste-token --provider anthropic
```

### Configuration CLI (setup-token)

```bash
# Collez un setup-token pendant l'onboarding
openclaw onboard --auth-choice setup-token
```

### Extrait de configuration (setup-token)

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Notes

- Générez le setup-token avec `claude setup-token` et collez-le, ou exécutez `openclaw models auth setup-token` sur l'hôte du gateway.
- Si vous voyez "OAuth token refresh failed ..." avec un abonnement Claude, ré-authentifiez-vous avec un setup-token. Voir [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription).
- Les détails d'authentification et les règles de réutilisation sont dans [/concepts/oauth](/concepts/oauth).

## Dépannage

**Erreurs 401 / token soudainement invalide**

- L'authentification par abonnement Claude peut expirer ou être révoquée. Relancez `claude setup-token`
  et collez-le sur l'**hôte du gateway**.
- Si le login CLI Claude se trouve sur une autre machine, utilisez
  `openclaw models auth paste-token --provider anthropic` sur l'hôte du gateway.

**No API key found for provider "anthropic"**

- L'authentification est **par agent**. Les nouveaux agents n'héritent pas des clés de l'agent principal.
- Relancez l'onboarding pour cet agent, ou collez un setup-token / clé API sur
  l'hôte du gateway, puis vérifiez avec `openclaw models status`.

**No credentials found for profile `anthropic:default`**

- Exécutez `openclaw models status` pour voir quel profil d'authentification est actif.
- Relancez l'onboarding, ou collez un setup-token / clé API pour ce profil.

**No available auth profile (all in cooldown/unavailable)**

- Vérifiez `openclaw models status --json` pour `auth.unusableProfiles`.
- Ajoutez un autre profil Anthropic ou attendez la fin du cooldown.

Plus d'informations : [/gateway/troubleshooting](/gateway/troubleshooting) et [/help/faq](/help/faq).
