---
summary: "Politique de réessai pour les appels sortants aux fournisseurs"
read_when:
  - Mise à jour du comportement de réessai du fournisseur ou des valeurs par défaut
  - Débogage d'erreurs d'envoi du fournisseur ou de limités de débit
title: "Politique de réessai"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/retry.md
  workflow: manual
---

# Politique de réessai

## Objectifs

- Réessayer par requête HTTP, pas par flux multi-étapes.
- Préserver l'ordre en réessayant uniquement l'étape en cours.
- Éviter de dupliquer les opérations non idempotentes.

## Valeurs par défaut

- Tentatives : 3
- Plafond de délai max : 30000 ms
- Jitter : 0.1 (10 pour cent)
- Valeurs par défaut du fournisseur :
  - Délai min Telegram : 400 ms
  - Délai min Discord : 500 ms

## Comportement

### Discord

- Réessaie uniquement en cas d'erreurs de limité de débit (HTTP 429).
- Utilisé le `retry_after` de Discord quand disponible, sinon backoff exponentiel.

### Telegram

- Réessaie en cas d'erreurs transitoires (429, timeout, connect/reset/closed, temporairement indisponible).
- Utilisé `retry_after` quand disponible, sinon backoff exponentiel.
- Les erreurs d'analyse Markdown ne sont pas réessayées ; elles replient vers le texte brut.

## Configuration

Définir la politique de réessai par fournisseur dans `~/.openclaw/openclaw.json` :

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## Notes

- Les réessais s'appliquent par requête (envoi de message, upload de média, réaction, sondage, sticker).
- Les flux composites ne réessaient pas les étapes complétées.
