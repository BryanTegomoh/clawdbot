---
summary: "Comment les fichiers audio/notes vocales entrants sont téléchargés, transcrits et injectés dans les réponses"
read_when:
  - Modification de la transcription audio ou de la gestion des médias
title: "Audio et notes vocales"
sidebarTitle: "Audio et notes vocales"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/audio.md
  workflow: manual
---

# Audio / Notes vocales — 2026-01-17

## Ce qui fonctionne

- **Compréhension des médias (audio)** : si la compréhension audio est activée (ou auto-détectée), OpenClaw :
  1. Localise la première pièce jointe audio (chemin local ou URL) et la télécharge si nécessaire.
  2. Applique `maxBytes` avant l'envoi à chaque entrée de modèle.
  3. Exécute la première entrée de modèle éligible dans l'ordre (fournisseur ou CLI).
  4. En cas d'échec ou de saut (taille/délai), essaie l'entrée suivante.
  5. En cas de succès, remplace `Body` par un bloc `[Audio]` et définit `{{Transcript}}`.
- **Analyse des commandes** : lorsque la transcription réussit, `CommandBody`/`RawBody` sont définis sur la transcription pour que les commandes slash continuent de fonctionner.
- **Journalisation verbeuse** : en mode `--verbose`, nous enregistrons quand la transcription s'exécute et quand elle remplace le corps.

## Auto-détection (par défaut)

Si vous **ne configurez pas de modèles** et que `tools.média.audio.enabled` n'est **pas** défini à `false`,
OpenClaw auto-détecte dans cet ordre et s'arrête à la première option fonctionnelle :

1. **CLI locaux** (si installés)
   - `sherpa-onnx-offline` (nécessite `SHERPA_ONNX_MODEL_DIR` avec encoder/decoder/joiner/tokens)
   - `whisper-cli` (de `whisper-cpp` ; utilisé `WHISPER_CPP_MODEL` ou le modèle tiny intégré)
   - `whisper` (CLI Python ; télécharge les modèles automatiquement)
2. **Gemini CLI** (`gemini`) utilisant `read_many_files`
3. **Clés de fournisseur** (OpenAI → Groq → Deepgram → Google)

Pour désactiver l'auto-détection, définissez `tools.média.audio.enabled: false`.
Pour personnaliser, définissez `tools.média.audio.models`.
Note : la détection binaire est au mieux sur macOS/Linux/Windows ; assurez-vous que le CLI est dans le `PATH` (nous étendons `~`), ou définissez un modèle CLI explicite avec un chemin de commande complet.

## Exemples de configuration

### Fournisseur + repli CLI (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Fournisseur uniquement avec filtrage par portée

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Fournisseur uniquement (Deepgram)

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

### Fournisseur uniquement (Mistral Voxtral)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## Notes et limités

- L'authentification du fournisseur suit l'ordre standard d'authentification des modèles (profils d'authentification, variables d'environnement, `models.providers.*.apiKey`).
- Deepgram utilisé `DEEPGRAM_API_KEY` lorsque `provider: "deepgram"` est utilisé.
- Détails de la configuration Deepgram : [Deepgram (transcription audio)](/providers/deepgram).
- Détails de la configuration Mistral : [Mistral](/providers/mistral).
- Les fournisseurs audio peuvent surcharger `baseUrl`, `headers` et `providerOptions` via `tools.média.audio`.
- La limité de taille par défaut est 20 Mo (`tools.média.audio.maxBytes`). Les fichiers audio surdimensionnés sont ignorés pour ce modèle et l'entrée suivante est essayée.
- Le `maxChars` par défaut pour l'audio est **non défini** (transcription complète). Définissez `tools.média.audio.maxChars` ou `maxChars` par entrée pour limiter la sortie.
- Le modèle auto par défaut d'OpenAI est `gpt-4o-mini-transcribe` ; définissez `model: "gpt-4o-transcribe"` pour une meilleure précision.
- Utilisez `tools.média.audio.attachments` pour traiter plusieurs notes vocales (`mode: "all"` + `maxAttachments`).
- La transcription est disponible dans les modèles via `{{Transcript}}`.
- La sortie stdout du CLI est limitée (5 Mo) ; gardez la sortie CLI concise.

## Détection des mentions dans les groupes

Lorsque `requireMention: true` est défini pour un chat de groupe, OpenClaw transcrit désormais l'audio **avant** de vérifier les mentions. Cela permet le traitement des notes vocales même lorsqu'elles contiennent des mentions.

**Fonctionnement :**

1. Si un message vocal n'a pas de corps texte et que le groupe nécessite des mentions, OpenClaw effectué une transcription « préliminaire ».
2. La transcription est vérifiée pour les motifs de mention (ex. `@BotName`, déclencheurs emoji).
3. Si une mention est trouvée, le message passe par le pipeline de réponse complet.
4. La transcription est utilisée pour la détection des mentions afin que les notes vocales puissent passer le filtré de mention.

**Comportement de repli :**

- Si la transcription échoue lors de la phase préliminaire (délai, erreur API, etc.), le message est traité sur la base de la détection de mentions par texte uniquement.
- Cela garantit que les messages mixtes (texte + audio) ne sont jamais incorrectement ignorés.

**Exemple :** un utilisateur envoie une note vocale disant « Hey @Claude, quel temps fait-il ? » dans un groupe Telegram avec `requireMention: true`. La note vocale est transcrite, la mention est détectée et l'agent répond.

## Pièges

- Les règles de portée utilisent le principe du premier correspondant. `chatType` est normalisé en `direct`, `group` ou `room`.
- Assurez-vous que votre CLI sort avec le code 0 et affiche du texte brut ; le JSON doit être traité via `jq -r .text`.
- Gardez des délais raisonnables (`timeoutSeconds`, par défaut 60s) pour éviter de bloquer la file de réponse.
- La transcription préliminaire ne traite que la **première** pièce jointe audio pour la détection des mentions. Les fichiers audio supplémentaires sont traités lors de la phase principale de compréhension des médias.
