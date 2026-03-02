---
summary: "Compréhension des images/audio/vidéo entrants (optionnelle) avec replis fournisseur + CLI"
read_when:
  - Conception ou refactorisation de la compréhension des médias
  - Ajustement du prétraitement audio/vidéo/image entrant
title: "Compréhension des médias"
sidebarTitle: "Compréhension des médias"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/media-understanding.md
  workflow: manual
---

# Compréhension des médias (entrant) — 2026-01-17

OpenClaw peut **résumer les médias entrants** (image/audio/vidéo) avant l'exécution du pipeline de réponse. Il auto-détecte lorsque des outils locaux ou des clés de fournisseur sont disponibles, et peut être désactivé ou personnalisé. Si la compréhension est désactivée, les modèles reçoivent toujours les fichiers/URL originaux normalement.

## Objectifs

- Optionnel : pré-digérer les médias entrants en texte court pour un routage plus rapide + une meilleure analyse des commandes.
- Préserver la livraison originale des médias au modèle (toujours).
- Supporter les **API de fournisseurs** et les **replis CLI**.
- Permettre plusieurs modèles avec repli ordonné (erreur/taille/délai).

## Comportement de haut niveau

1. Collecter les pièces jointes entrantes (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Pour chaque capacité activée (image/audio/vidéo), sélectionner les pièces jointes selon la politique (par défaut : **première**).
3. Choisir la première entrée de modèle éligible (taille + capacité + authentification).
4. Si un modèle échoue ou si le média est trop volumineux, **se replier sur l'entrée suivante**.
5. En cas de succès :
   - `Body` devient un bloc `[Image]`, `[Audio]` ou `[Video]`.
   - L'audio définit `{{Transcript}}` ; l'analyse des commandes utilisé le texte de légende lorsque présent,
     sinon la transcription.
   - Les légendes sont préservées en tant que `User text:` à l'intérieur du bloc.

Si la compréhension échoue ou est désactivée, **le flux de réponse continue** avec le corps original + les pièces jointes.

## Vue d'ensemble de la configuration

`tools.média` supporte des **modèles partagés** plus des surcharges par capacité :

- `tools.média.models` : liste de modèles partagés (utiliser `capabilities` pour le filtrage).
- `tools.média.image` / `tools.média.audio` / `tools.média.video` :
  - valeurs par défaut (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - surcharges de fournisseur (`baseUrl`, `headers`, `providerOptions`)
  - options Deepgram audio via `tools.média.audio.providerOptions.deepgram`
  - liste optionnelle **de `models` par capacité** (prioritaire avant les modèles partagés)
  - politique `attachments` (`mode`, `maxAttachments`, `prefer`)
  - `scope` (filtrage optionnel par canal/chatType/clé de session)
- `tools.média.concurrency` : exécutions de capacité concurrentes maximales (par défaut **2**).

```json5
{
  tools: {
    media: {
      models: [
        /* liste partagée */
      ],
      image: {
        /* surcharges optionnelles */
      },
      audio: {
        /* surcharges optionnelles */
      },
      video: {
        /* surcharges optionnelles */
      },
    },
  },
}
```

### Entrées de modèle

Chaque entrée `models[]` peut être de type **fournisseur** ou **CLI** :

```json5
{
  type: "provider", // par défaut si omis
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optionnel, utilisé pour les entrées multi-modales
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

Les modèles CLI peuvent également utiliser :

- `{{MediaDir}}` (répertoire contenant le fichier média)
- `{{OutputDir}}` (répertoire de travail crée pour cette exécution)
- `{{OutputBase}}` (chemin de base du fichier de travail, sans extension)

## Valeurs par défaut et limités

Valeurs par défaut recommandées :

- `maxChars` : **500** pour image/vidéo (court, compatible avec les commandes)
- `maxChars` : **non défini** pour l'audio (transcription complète sauf si vous définissez une limite)
- `maxBytes` :
  - image : **10 Mo**
  - audio : **20 Mo**
  - vidéo : **50 Mo**

Règles :

- Si le média dépasse `maxBytes`, ce modèle est ignoré et le **modèle suivant est essayé**.
- Si le modèle retourne plus que `maxChars`, la sortie est tronquée.
- `prompt` par défaut est un simple « Describe the {média}. » plus les indications `maxChars` (image/vidéo uniquement).
- Si `<capability>.enabled: true` mais qu'aucun modèle n'est configuré, OpenClaw essaie le
  **modèle de réponse actif** lorsque son fournisseur supporte la capacité.

### Auto-détection de la compréhension des médias (par défaut)

Si `tools.média.<capability>.enabled` n'est **pas** défini à `false` et que vous n'avez pas
configuré de modèles, OpenClaw auto-détecte dans cet ordre et **s'arrête à la première
option fonctionnelle** :

1. **CLI locaux** (audio uniquement ; si installés)
   - `sherpa-onnx-offline` (nécessite `SHERPA_ONNX_MODEL_DIR` avec encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp` ; utilisé `WHISPER_CPP_MODEL` ou le modèle tiny intégré)
   - `whisper` (CLI Python ; télécharge les modèles automatiquement)
2. **Gemini CLI** (`gemini`) utilisant `read_many_files`
3. **Clés de fournisseur**
   - Audio : OpenAI → Groq → Deepgram → Google
   - Image : OpenAI → Anthropic → Google → MiniMax
   - Vidéo : Google

Pour désactiver l'auto-détection, définissez :

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Note : la détection binaire est au mieux sur macOS/Linux/Windows ; assurez-vous que le CLI est dans le `PATH` (nous étendons `~`), ou définissez un modèle CLI explicite avec un chemin de commande complet.

## Capacités (optionnel)

Si vous définissez `capabilities`, l'entrée ne s'exécute que pour ces types de médias. Pour les listes
partagées, OpenClaw peut inférer les valeurs par défaut :

- `openai`, `anthropic`, `minimax` : **image**
- `google` (API Gemini) : **image + audio + vidéo**
- `groq` : **audio**
- `deepgram` : **audio**

Pour les entrées CLI, **définissez `capabilities` explicitement** pour éviter les correspondances surprenantes.
Si vous omettez `capabilities`, l'entrée est éligible pour la liste dans laquelle elle apparaît.

## Matrice de support des fournisseurs (intégrations OpenClaw)

| Capacité | Intégration fournisseur                          | Notes                                                     |
| -------- | ------------------------------------------------ | --------------------------------------------------------- |
| Image    | OpenAI / Anthropic / Google / autres via `pi-ai` | Tout modèle capable d'images dans le registre fonctionne. |
| Audio    | OpenAI, Groq, Deepgram, Google, Mistral          | Transcription fournisseur (Whisper/Deepgram/Gemini/Voxtral). |
| Vidéo    | Google (API Gemini)                              | Compréhension vidéo par fournisseur.                      |

## Fournisseurs recommandés

**Image**

- Préférez votre modèle actif s'il supporte les images.
- Bons choix par défaut : `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Audio**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, `deepgram/nova-3`, ou `mistral/voxtral-mini-latest`.
- Repli CLI : `whisper-cli` (whisper-cpp) ou `whisper`.
- Configuration Deepgram : [Deepgram (transcription audio)](/providers/deepgram).

**Vidéo**

- `google/gemini-3-flash-preview` (rapide), `google/gemini-3-pro-preview` (plus riche).
- Repli CLI : CLI `gemini` (supporte `read_file` sur vidéo/audio).

## Politique de pièces jointes

La politique `attachments` par capacité contrôle quelles pièces jointes sont traitées :

- `mode` : `first` (par défaut) ou `all`
- `maxAttachments` : limité le nombre traité (par défaut **1**)
- `prefer` : `first`, `last`, `path`, `url`

Lorsque `mode: "all"`, les sorties sont étiquetées `[Image 1/2]`, `[Audio 2/2]`, etc.

## Exemples de configuration

### 1) Liste de modèles partagés + surcharges

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) Audio + Vidéo uniquement (image désactivée)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) Compréhension d'image optionnelle

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) Entrée unique multi-modale (capacités explicites)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## Sortie de statut

Lorsque la compréhension des médias s'exécute, `/status` inclut une ligne de résumé courte :

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Cela montre les résultats par capacité et le fournisseur/modèle choisi le cas échéant.

## Notes

- La compréhension est au **mieux**. Les erreurs ne bloquent pas les réponses.
- Les pièces jointes sont toujours transmises aux modèles même lorsque la compréhension est désactivée.
- Utilisez `scope` pour limiter où la compréhension s'exécute (ex. uniquement les messages directs).

## Documentation associée

- [Configuration](/gateway/configuration)
- [Support images et médias](/nodes/images)
