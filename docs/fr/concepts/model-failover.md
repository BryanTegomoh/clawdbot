---
summary: "Comment OpenClaw effectué la rotation des profils d'authentification et le repli entre les modèles"
read_when:
  - Diagnostic de la rotation des profils d'authentification, des temps de repos ou du comportement de repli du modèle
  - Mise à jour des règles de repli pour les profils d'authentification ou les modèles
title: "Repli de modèle"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/model-failover.md
  workflow: manual
---

# Repli de modèle

OpenClaw gère les échecs en deux étapes :

1. **Rotation des profils d'authentification** au sein du fournisseur actuel.
2. **Repli de modèle** vers le modèle suivant dans `agents.defaults.model.fallbacks`.

Ce document explique les règles d'exécution et les données qui les soutiennent.

## Stockage d'authentification (clés + OAuth)

OpenClaw utilise des **profils d'authentification** pour les clés API et les tokens OAuth.

- Les secrets résident dans `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (ancien : `~/.openclaw/agent/auth-profiles.json`).
- La configuration `auth.profiles` / `auth.order` contient uniquement les **métadonnées + routage** (pas de secrets).
- Ancien fichier OAuth d'import uniquement : `~/.openclaw/credentials/oauth.json` (importé dans `auth-profiles.json` à la première utilisation).

Plus de détails : [/concepts/oauth](/concepts/oauth)

Types d'identifiants :

- `type: "api_key"` -> `{ provider, key }`
- `type: "oauth"` -> `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` pour certains fournisseurs)

## Identifiants de profil

Les connexions OAuth créent des profils distincts pour que plusieurs comptes puissent coexister.

- Par défaut : `provider:default` quand aucun email n'est disponible.
- OAuth avec email : `provider:<email>` (par exemple `google-antigravity:user@gmail.com`).

Les profils résident dans `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` sous `profiles`.

## Ordre de rotation

Quand un fournisseur a plusieurs profils, OpenClaw choisit un ordre ainsi :

1. **Configuration explicite** : `auth.order[provider]` (si défini).
2. **Profils configurés** : `auth.profiles` filtrés par fournisseur.
3. **Profils stockés** : entrées dans `auth-profiles.json` pour le fournisseur.

Si aucun ordre explicite n'est configuré, OpenClaw utilise un ordre circulaire :

- **Clé primaire :** type de profil (**OAuth avant les clés API**).
- **Clé secondaire :** `usageStats.lastUsed` (le plus ancien d'abord, au sein de chaque type).
- Les **profils en temps de repos/désactivés** sont déplacés à la fin, ordonnés par expiration la plus proche.

### Affinité de session (favorable au cache)

OpenClaw **fixe le profil d'authentification choisi par session** pour garder les caches du fournisseur actifs. Il ne tourne **pas** à chaque requête. Le profil fixé est réutilisé jusqu'à :

- la réinitialisation de la session (`/new` / `/reset`)
- la complétion d'une compaction (incrémentation du compteur de compaction)
- le profil est en temps de repos/désactivé

La sélection manuelle via `/model ...@<profileId>` définit une **surcharge utilisateur** pour cette session et n'est pas auto-rotée jusqu'au démarrage d'une nouvelle session.

Les profils fixés automatiquement (sélectionnés par le routeur de session) sont traités comme une **préférence** : ils sont essayés en premier, mais OpenClaw peut tourner vers un autre profil en cas de limités de débit/timeouts. Les profils fixés par l'utilisateur restent verrouillés sur ce profil ; s'il échoue et que des replis de modèle sont configurés, OpenClaw passe au modèle suivant au lieu de changer de profil.

### Pourquoi OAuth peut « sembler perdu »

Si vous avez à la fois un profil OAuth et un profil de clé API pour le même fournisseur, la rotation circulaire peut alterner entre eux d'un message à l'autre sauf si fixé. Pour forcer un seul profil :

- Fixer avec `auth.order[provider] = ["provider:profileId"]`, ou
- Utiliser une surcharge par session via `/model ...` avec une surcharge de profil (quand supporté par votre interface/surface de chat).

## Temps de repos

Quand un profil échoue en raison d'erreurs d'authentification/limités de débit (ou d'un timeout qui ressemble à une limitation de débit), OpenClaw le marque en temps de repos et passe au profil suivant. Les erreurs de format/requête invalide (par exemple les échecs de validation d'identifiant d'appel d'outil Cloud Code Assist) sont traitées comme nécessitant un repli et utilisent les mêmes temps de repos.

Les temps de repos utilisent un backoff exponentiel :

- 1 minute
- 5 minutes
- 25 minutes
- 1 heure (plafond)

L'état est stocké dans `auth-profiles.json` sous `usageStats` :

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## Désactivations de facturation

Les échecs de facturation/crédit (par exemple « crédits insuffisants » / « solde de crédit trop bas ») sont traités comme nécessitant un repli, mais ils ne sont généralement pas transitoires. Au lieu d'un court temps de repos, OpenClaw marque le profil comme **désactivé** (avec un backoff plus long) et tourne vers le profil/fournisseur suivant.

L'état est stocké dans `auth-profiles.json` :

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Valeurs par défaut :

- Le backoff de facturation commence à **5 heures**, double à chaque échec de facturation et plafonne à **24 heures**.
- Les compteurs de backoff se réinitialisent si le profil n'a pas échoué pendant **24 heures** (configurable).

## Repli de modèle

Si tous les profils d'un fournisseur échouent, OpenClaw passe au modèle suivant dans `agents.defaults.model.fallbacks`. Cela s'applique aux échecs d'authentification, limités de débit et timeouts qui ont épuisé la rotation de profils (les autres erreurs ne font pas avancer le repli).

Quand une exécution démarre avec une surcharge de modèle (hooks ou CLI), les replis se terminent toujours à `agents.defaults.model.primary` après avoir essayé les replis configurés.

## Configuration associée

Voir [Configuration du Gateway](/gateway/configuration) pour :

- `auth.profiles` / `auth.order`
- `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
- `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
- `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
- Routage `agents.defaults.imageModel`

Voir [Modèles](/concepts/models) pour la vue d'ensemble de la sélection de modèle et du repli.
