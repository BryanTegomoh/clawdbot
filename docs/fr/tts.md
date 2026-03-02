---
summary: "Synthèse vocale (TTS) pour les réponses sortantes"
read_when:
  - Activation de la synthèse vocale pour les réponses
  - Configuration des fournisseurs ou limités TTS
  - Utilisation des commandes /tts
title: "Synthèse vocale"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/tts.md
  workflow: manual
---

# Synthèse vocale (TTS)

OpenClaw peut convertir les réponses sortantes en audio en utilisant ElevenLabs, OpenAI ou Edge TTS.
Cela fonctionne partout où OpenClaw peut envoyer de l'audio ; Telegram affiche une bulle ronde de note vocale.

## Services supportés

- **ElevenLabs** (fournisseur principal ou de repli)
- **OpenAI** (fournisseur principal ou de repli ; aussi utilisé pour les résumés)
- **Edge TTS** (fournisseur principal ou de repli ; utilisé `node-edge-tts`, par défaut quand il n'y a pas de clés API)

### Notes sur Edge TTS

Edge TTS utilisé le service de TTS neural en ligne de Microsoft Edge via la bibliothèque `node-edge-tts`.
C'est un service héberge (pas local), utilisé les endpoints de Microsoft et ne
nécessite pas de clé API. `node-edge-tts` expose des options de configuration vocale et
des formats de sortie, mais toutes les options ne sont pas supportées par le service Edge. citeturn2search0

Comme Edge TTS est un service web public sans SLA ni quota publié, traitez-le
comme un service en mode best-effort. Si vous avez besoin de limités garanties et de support, utilisez OpenAI ou ElevenLabs.
L'API REST Speech de Microsoft documente une limite audio de 10 minutes par requête ; Edge TTS
ne publie pas de limités, donc supposez des limités similaires ou inférieures. citeturn0search3

## Clés optionnelles

Si vous souhaitez OpenAI ou ElevenLabs :

- `ELEVENLABS_API_KEY` (ou `XI_API_KEY`)
- `OPENAI_API_KEY`

Edge TTS ne nécessite **pas** de clé API. Si aucune clé API n'est trouvée, OpenClaw bascule par défaut
sur Edge TTS (sauf si désactivé via `messages.tts.edge.enabled=false`).

Si plusieurs fournisseurs sont configurés, le fournisseur sélectionné est utilisé en premier et les autres sont des options de repli.
Le résumé automatique utilisé le `summaryModel` configuré (ou `agents.defaults.model.primary`),
donc ce fournisseur doit aussi être authentifié si vous activez les résumés.

## Liens des services

- [Guide OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
- [Référence API Audio OpenAI](https://platform.openai.com/docs/api-reference/audio)
- [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
- [Authentification ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
- [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
- [Formats de sortie Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

## Est-ce active par défaut ?

Non. Le TTS automatique est **désactivé** par défaut. Activez-le dans la config avec
`messages.tts.auto` ou par session avec `/tts always` (alias : `/tts on`).

Edge TTS **est** activé par défaut une fois le TTS activé, et est utilisé automatiquement
quand aucune clé API OpenAI ou ElevenLabs n'est disponible.

## Configuration

La configuration TTS se trouve sous `messages.tts` dans `openclaw.json`.
Le schéma complet est dans [Configuration du Gateway](/gateway/configuration).

### Configuration minimale (activation + fournisseur)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI principal avec repli ElevenLabs

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTS principal (sans clé API)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```

### Désactiver Edge TTS

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### Limités personnalisées + chemin des préférences

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### Répondre en audio uniquement après une note vocale entrante

```json5
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### Désactiver le résumé automatique pour les longues réponses

```json5
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

Puis exécutez :

```
/tts summary off
```

### Notes sur les champs

- `auto` : mode TTS automatique (`off`, `always`, `inbound`, `tagged`).
  - `inbound` n'envoie de l'audio qu'après une note vocale entrante.
  - `tagged` n'envoie de l'audio que quand la réponse inclut des tags `[[tts]]`.
- `enabled` : toggle legacy (doctor migre ceci vers `auto`).
- `mode` : `"final"` (par défaut) ou `"all"` (inclut les réponses outil/bloc).
- `provider` : `"elevenlabs"`, `"openai"` ou `"edge"` (le repli est automatique).
- Si `provider` n'est **pas défini**, OpenClaw préfère `openai` (si clé), puis `elevenlabs` (si clé),
  sinon `edge`.
- `summaryModel` : modèle économique optionnel pour le résumé automatique ; par défaut `agents.defaults.model.primary`.
  - Accepte `provider/model` ou un alias de modèle configuré.
- `modelOverrides` : permet au modèle d'émettre des directives TTS (activé par défaut).
  - `allowProvider` est par défaut `false` (le changement de fournisseur est opt-in).
- `maxTextLength` : limité stricte pour l'entrée TTS (caractères). `/tts audio` échoue si dépassée.
- `timeoutMs` : timeout de requête (ms).
- `prefsPath` : remplacer le chemin JSON des préférences locales (fournisseur/limité/résumé).
- Les valeurs `apiKey` se replient sur les variables d'environnement (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
- `elevenlabs.baseUrl` : remplacer l'URL de base de l'API ElevenLabs.
- `elevenlabs.voiceSettings` :
  - `stability`, `similarityBoost`, `style` : `0..1`
  - `useSpeakerBoost` : `true|false`
  - `speed` : `0.5..2.0` (1.0 = normal)
- `elevenlabs.applyTextNormalization` : `auto|on|off`
- `elevenlabs.languageCode` : ISO 639-1 à 2 lettres (ex. `en`, `de`)
- `elevenlabs.seed` : entier `0..4294967295` (déterminisme best-effort)
- `edge.enabled` : autoriser l'utilisation d'Edge TTS (par défaut `true` ; pas de clé API).
- `edge.voice` : nom de voix neurale Edge (ex. `en-US-MichelleNeural`).
- `edge.lang` : code de langue (ex. `en-US`).
- `edge.outputFormat` : format de sortie Edge (ex. `audio-24khz-48kbitrate-mono-mp3`).
  - Voir les formats de sortie Microsoft Speech pour les valeurs valides ; tous les formats ne sont pas supportés par Edge.
- `edge.rate` / `edge.pitch` / `edge.volume` : chaînes en pourcentage (ex. `+10%`, `-5%`).
- `edge.saveSubtitles` : écrire des sous-titres JSON à côté du fichier audio.
- `edge.proxy` : URL de proxy pour les requêtes Edge TTS.
- `edge.timeoutMs` : remplacement du timeout de requête (ms).

## Surcharges pilotées par le modèle (activé par défaut)

Par défaut, le modèle **peut** émettre des directives TTS pour une seule réponse.
Quand `messages.tts.auto` est `tagged`, ces directives sont requises pour déclencher l'audio.

Quand activé, le modèle peut émettre des directives `[[tts:...]]` pour remplacer la voix
pour une seule réponse, plus un bloc optionnel `[[tts:text]]...[[/tts:text]]` pour
fournir des tags expressifs (rires, indices de chant, etc.) qui ne doivent apparaître que dans
l'audio.

Les directives `provider=...` sont ignorées sauf si `modelOverrides.allowProvider: true`.

Exemple de payload de réponse :

```
Here you go.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

Clés de directive disponibles (quand activé) :

- `provider` (`openai` | `elevenlabs` | `edge`, nécessite `allowProvider: true`)
- `voice` (voix OpenAI) ou `voiceId` (ElevenLabs)
- `model` (modèle TTS OpenAI ou id de modèle ElevenLabs)
- `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
- `applyTextNormalization` (`auto|on|off`)
- `languageCode` (ISO 639-1)
- `seed`

Désactiver toutes les surcharges du modèle :

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

Allowlist optionnelle (activer le changement de fournisseur tout en gardant les autres réglages configurables) :

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: true,
        allowSeed: false,
      },
    },
  },
}
```

## Préférences par utilisateur

Les commandes slash écrivent des surcharges locales dans `prefsPath` (par défaut :
`~/.openclaw/settings/tts.json`, remplaçable avec `OPENCLAW_TTS_PREFS` ou
`messages.tts.prefsPath`).

Champs stockés :

- `enabled`
- `provider`
- `maxLength` (seuil de résumé ; par défaut 1500 caractères)
- `summarize` (par défaut `true`)

Ces valeurs remplacent `messages.tts.*` pour cet hôte.

## Formats de sortie (fixes)

- **Telegram** : note vocale Opus (`opus_48000_64` depuis ElevenLabs, `opus` depuis OpenAI).
  - 48kHz / 64kbps est un bon compromis pour les notes vocales et requis pour la bulle ronde.
- **Autres canaux** : MP3 (`mp3_44100_128` depuis ElevenLabs, `mp3` depuis OpenAI).
  - 44.1kHz / 128kbps est l'équilibre par défaut pour la clarté vocale.
- **Edge TTS** : utilisé `edge.outputFormat` (par défaut `audio-24khz-48kbitrate-mono-mp3`).
  - `node-edge-tts` accepte un `outputFormat`, mais tous les formats ne sont pas disponibles
    depuis le service Edge. citeturn2search0
  - Les valeurs de format de sortie suivent les formats de sortie Microsoft Speech (y compris Ogg/WebM Opus). citeturn1search0
  - `sendVoice` de Telegram accepte OGG/MP3/M4A ; utilisez OpenAI/ElevenLabs si vous avez besoin
    de notes vocales Opus garanties. citeturn1search1
  - Si le format de sortie Edge configuré échoue, OpenClaw réessaie avec MP3.

Les formats OpenAI/ElevenLabs sont fixes ; Telegram attend Opus pour l'UX de note vocale.

## Comportement du TTS automatique

Quand activé, OpenClaw :

- ignoré le TTS si la réponse contient déjà un média ou une directive `MEDIA:`.
- ignoré les réponses très courtes (< 10 caractères).
- résume les longues réponses quand activé en utilisant `agents.defaults.model.primary` (ou `summaryModel`).
- attache l'audio généré à la réponse.

Si la réponse dépasse `maxLength` et que le résumé est désactivé (ou pas de clé API pour le
modèle de résumé), l'audio
est ignoré et la réponse texte normale est envoyée.

## Diagramme de flux

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```

## Utilisation de la commande slash

Il y a une seule commande : `/tts`.
Voir [Commandes slash](/tools/slash-commands) pour les détails d'activation.

Note Discord : `/tts` est une commande Discord intégrée, donc OpenClaw enregistre
`/voice` comme commande native là-bas. Le texte `/tts ...` fonctionne toujours.

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Notes :

- Les commandes nécessitent un expéditeur autorisé (les règles d'allowlist/propriétaire s'appliquent toujours).
- `commands.text` ou l'enregistrement de commande native doit être activé.
- `off|always|inbound|tagged` sont des toggles par session (`/tts on` est un alias pour `/tts always`).
- `limit` et `summary` sont stockés dans les préférences locales, pas dans la config principale.
- `/tts audio` génère une réponse audio ponctuelle (n'activé pas le TTS).

## Outil agent

L'outil `tts` convertit du texte en parole et retourne un chemin `MEDIA:`. Quand le
résultat est compatible Telegram, l'outil inclut `[[audio_as_voice]]` pour que
Telegram envoie une bulle vocale.

## RPC du Gateway

Méthodes du Gateway :

- `tts.status`
- `tts.enable`
- `tts.disable`
- `tts.convert`
- `tts.setProvider`
- `tts.providers`
