---
summary: "Mode Talk : conversations vocales continues avec TTS ElevenLabs"
read_when:
  - Implémentation du mode Talk sur macOS/iOS/Android
  - Modification du comportement voix/TTS/interruption
title: "Mode Talk"
sidebarTitle: "Mode Talk"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/talk.md
  workflow: manual
---

# Mode Talk

Le mode Talk est une boucle de conversation vocale continue :

1. Écouter la parole
2. Envoyer la transcription au modèle (session principale, chat.send)
3. Attendre la réponse
4. La prononcer via ElevenLabs (lecture en streaming)

## Comportement (macOS)

- **Overlay permanent** lorsque le mode Talk est activé.
- Transitions de phase **Écoute → Réflexion → Parole**.
- Lors d'une **courte pause** (fenêtre de silence), la transcription en cours est envoyée.
- Les réponses sont **écrites dans le WebChat** (comme en tapant).
- **Interruption à la parole** (activée par défaut) : si l'utilisateur commence à parler pendant que l'assistant parle, nous arrêtons la lecture et notons l'horodatage de l'interruption pour la prochaine invite.

## Directives vocales dans les réponses

L'assistant peut préfixer sa réponse avec une **seule ligne JSON** pour contrôler la voix :

```json
{ "voice": "<voice-id>", "once": true }
```

Règles :

- Première ligne non vide uniquement.
- Les clés inconnues sont ignorées.
- `once: true` s'applique uniquement à la réponse en cours.
- Sans `once`, la voix devient la nouvelle valeur par défaut pour le mode Talk.
- La ligne JSON est retirée avant la lecture TTS.

Clés supportées :

- `voice` / `voice_id` / `voiceId`
- `model` / `model_id` / `modelId`
- `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
- `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
- `once`

## Configuration (`~/.openclaw/openclaw.json`)

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

Valeurs par défaut :

- `interruptOnSpeech` : true
- `voiceId` : se rabat sur `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` (ou la première voix ElevenLabs lorsque la clé API est disponible)
- `modelId` : `eleven_v3` par défaut lorsque non défini
- `apiKey` : se rabat sur `ELEVENLABS_API_KEY` (ou le profil shell du gateway si disponible)
- `outputFormat` : `pcm_44100` par défaut sur macOS/iOS et `pcm_24000` sur Android (définir `mp3_*` pour forcer le streaming MP3)

## Interface macOS

- Bascule dans la barre de menus : **Talk**
- Onglet de configuration : groupe **Mode Talk** (identifiant de voix + bascule d'interruption)
- Overlay :
  - **Écoute** : nuage qui pulse avec le niveau du micro
  - **Réflexion** : animation de descente
  - **Parole** : anneaux qui rayonnent
  - Cliquer sur le nuage : arrêter la parole
  - Cliquer sur X : quitter le mode Talk

## Notes

- Nécessite les permissions Parole + Microphone.
- Utilisé `chat.send` contre la clé de session `main`.
- Le TTS utilisé l'API de streaming ElevenLabs avec `ELEVENLABS_API_KEY` et lecture incrémentale sur macOS/iOS/Android pour une latence réduite.
- `stability` pour `eleven_v3` est validé à `0.0`, `0.5` ou `1.0` ; les autres modèles acceptent `0..1`.
- `latency_tier` est validé de `0..4` lorsque défini.
- Android supporte les formats de sortie `pcm_16000`, `pcm_22050`, `pcm_24000` et `pcm_44100` pour le streaming AudioTrack à faible latence.
