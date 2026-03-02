---
summary: "Configuration de l'API Brave Search pour web_search"
read_when:
  - Vous souhaitez utiliser Brave Search pour web_search
  - Vous avez besoin d'une BRAVE_API_KEY ou de détails sur les forfaits
title: "Brave Search"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/brave-search.md
  workflow: manual
---

# API Brave Search

OpenClaw utilise Brave Search comme fournisseur par défaut pour `web_search`.

## Obtenir une clé API

1. Créez un compte API Brave Search sur [https://brave.com/search/api/](https://brave.com/search/api/)
2. Dans le tableau de bord, choisissez le forfait **Data for Search** et générez une clé API.
3. Stockez la clé dans la config (recommandé) ou définissez `BRAVE_API_KEY` dans l'environnement du Gateway.

## Exemple de configuration

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## Notes

- Le forfait Data for AI n'est **pas** compatible avec `web_search`.
- Brave propose un niveau gratuit ainsi que des forfaits payants ; consultez le portail API Brave pour les limités actuelles.

Voir [Outils web](/tools/web) pour la configuration complète de web_search.
