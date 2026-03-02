---
summary: "Firecrawl comme solution de repli pour web_fetch (anti-bot + extraction mise en cache)"
read_when:
  - Vous souhaitez une extraction web via Firecrawl
  - Vous avez besoin d'une clé API Firecrawl
  - Vous souhaitez une extraction anti-bot pour web_fetch
title: "Firecrawl"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/firecrawl.md
  workflow: manual
---

# Firecrawl

OpenClaw peut utiliser **Firecrawl** comme extracteur de repli pour `web_fetch`. C'est un
service d'extraction de contenu hébergé qui prend en charge le contournement de bots et la mise en cache, ce qui aide
avec les sites lourds en JS ou les pages qui bloquent les récupérations HTTP simples.

## Obtenir une clé API

1. Créez un compte Firecrawl et générez une clé API.
2. Stockez-la dans la configuration ou définissez `FIRECRAWL_API_KEY` dans l'environnement du Gateway.

## Configurer Firecrawl

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Notes :

- `firecrawl.enabled` est `true` par défaut lorsqu'une clé API est présente.
- `maxAgeMs` contrôle l'âge maximal des résultats en cache (ms). La valeur par défaut est de 2 jours.

## Furtivité / contournement de bots

Firecrawl expose un paramètre de **mode proxy** pour le contournement de bots (`basic`, `stealth` ou `auto`).
OpenClaw utilise toujours `proxy: "auto"` plus `storeInCache: true` pour les requêtes Firecrawl.
Si proxy est omis, Firecrawl utilisé `auto` par défaut. `auto` réessaie avec des proxys furtifs si une tentative basique échoue, ce qui peut consommer plus de crédits
qu'un scraping basique uniquement.

## Comment `web_fetch` utilisé Firecrawl

Ordre d'extraction de `web_fetch` :

1. Readability (local)
2. Firecrawl (si configuré)
3. Nettoyage HTML basique (dernier recours)

Voir [Outils web](/tools/web) pour la configuration complète des outils web.
