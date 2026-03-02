---
summary: "Transcription Deepgram pour les notes vocales entrantes"
read_when:
  - Vous souhaitez utiliser Deepgram pour la reconnaissance vocale des pièces jointes audio
  - Vous avez besoin d'un exemple rapide de configuration Deepgram
title: "Deepgram"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/deepgram.md
  workflow: manual
---

# Deepgram (Transcription audio)

Deepgram est une API de reconnaissance vocale. Dans OpenClaw, il est utilisé pour la **transcription des notes audio/vocales entrantes** via `tools.média.audio`.

Lorsqu'il est activé, OpenClaw envoie le fichier audio à Deepgram et injecte la transcription dans le pipeline de réponse (`{{Transcript}}` + bloc `[Audio]`). Il ne s'agit **pas** de streaming ; il utilise l'endpoint de transcription pré-enregistré.

Site web : [https://deepgram.com](https://deepgram.com)
Documentation : [https://developers.deepgram.com](https://developers.deepgram.com)

## Démarrage rapide

1. Définissez votre clé API :

```
DEEPGRAM_API_KEY=dg_...
```

2. Activez le fournisseur :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Options

- `model` : identifiant du modèle Deepgram (par défaut : `nova-3`)
- `language` : indication de langue (optionnel)
- `tools.média.audio.providerOptions.deepgram.detect_language` : activer la détection de langue (optionnel)
- `tools.média.audio.providerOptions.deepgram.punctuate` : activer la ponctuation (optionnel)
- `tools.média.audio.providerOptions.deepgram.smart_format` : activer le formatage intelligent (optionnel)

Exemple avec langue :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

Exemple avec options Deepgram :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Notes

- L'authentification suit l'ordre standard d'authentification des fournisseurs ; `DEEPGRAM_API_KEY` est le chemin le plus simple.
- Surchargez les endpoints ou en-têtes avec `tools.média.audio.baseUrl` et `tools.média.audio.headers` en cas d'utilisation d'un proxy.
- La sortie suit les mêmes règles audio que les autres fournisseurs (limités de taille, délais d'attente, injection de transcription).
