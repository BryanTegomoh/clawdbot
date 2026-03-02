---
title: "Diffs"
summary: "Visualiseur de diff en lecture seule et générateur PNG pour les agents (outil plugin optionnel)"
description: "Utilisez le plugin optionnel Diffs pour afficher du texte avant/après ou des patches unifiés sous forme de vue diff hébergée par la passerelle ou de PNG."
read_when:
  - Vous voulez que les agents affichent des modifications de code ou de markdown sous forme de diffs
  - Vous voulez une URL de visualiseur prête pour le canvas ou un PNG de diff rendu
---

# Diffs

`diffs` est un **outil plugin optionnel** qui affiche un diff en lecture seule à partir de :

- texte `before` / `after` arbitraire
- un patch unifié

L'outil peut produire :

- une URL de visualiseur hébergée par la passerelle pour utilisation en canvas
- une image PNG pour la livraison de messages
- les deux sorties ensemble

## Activer le plugin

```json5
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## Ce que les agents reçoivent

- `mode: "view"` retourne `details.viewerUrl` et `details.viewerPath`
- `mode: "image"` retourne `details.imagePath` uniquement
- `mode: "both"` retourne les détails du visualiseur plus `details.imagePath`

Patterns d'agent typiques :

- ouvrir `details.viewerUrl` dans le canvas avec `canvas present`
- envoyer `details.imagePath` avec l'outil `message` en utilisant `path` ou `filePath`

## Entrées de l'outil

Entrée avant/après :

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

Entrée patch :

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

Options utiles :

- `mode` : `view`, `image` ou `both`
- `layout` : `unified` ou `split`
- `theme` : `light` ou `dark`
- `expandUnchanged` : développer les sections inchangées au lieu de les réduire
- `path` : nom d'affichage pour l'entrée avant/après
- `title` : titre explicite du diff
- `ttlSeconds` : durée de vie de l'artefact du visualiseur
- `baseUrl` : remplacer l'URL de base de la passerelle utilisée dans le lien du visualiseur retourné

## Valeurs par défaut du plugin

Définissez les valeurs par défaut à l'échelle du plugin dans `~/.openclaw/openclaw.json` :

```json5
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            layout: "unified",
            wordWrap: true,
            background: true,
            theme: "dark",
            mode: "both",
          },
        },
      },
    },
  },
}
```

Valeurs par défaut prises en charge :

- `fontFamily`
- `fontSize`
- `layout`
- `wordWrap`
- `background`
- `theme`
- `mode`

Les paramètres d'outil explicites remplacent les valeurs par défaut du plugin.

## Remarques

- Les pages du visualiseur sont hébergées localement par la passerelle sous `/plugins/diffs/...`.
- Les artefacts du visualiseur sont éphémères et stockés localement.
- `mode: "image"` utilise un chemin de rendu image uniquement plus rapide et ne crée pas d'URL de visualiseur.
- Le rendu PNG nécessite un navigateur compatible Chromium. Si la détection automatique ne suffit pas, définissez `browser.executablePath`.
- Le rendu de diff est propulsé par [Diffs](https://diffs.com).

## Documents connexes

- [Vue d'ensemble des outils](/tools)
- [Plugins](/tools/plugin)
- [Navigateur](/tools/browser)
