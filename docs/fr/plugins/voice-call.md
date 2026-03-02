---
summary: "Plugin Voice Call : appels sortants et entrants via Twilio/Telnyx/Plivo (installation du plugin, configuration et CLI)"
read_when:
  - Vous souhaitez passer un appel vocal sortant depuis OpenClaw
  - Vous configurez ou developpez le plugin voice-call
title: "Plugin Voice Call"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/plugins/voice-call.md
  workflow: manual
---

# Voice Call (plugin)

Appels vocaux pour OpenClaw via un plugin. Prend en charge les notifications sortantes et
les conversations multi-tours avec des politiques d'appels entrants.

Fournisseurs actuels :

- `twilio` (Programmable Voice + Média Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (dev/sans réseau)

Modèle mental rapide :

- Installer le plugin
- Redémarrer le Gateway
- Configurer sous `plugins.entries.voice-call.config`
- Utiliser `openclaw voicecall ...` ou l'outil `voice_call`

## Ou s'exécute-t-il (local vs distant)

Le plugin Voice Call s'exécute **dans le processus du Gateway**.

Si vous utilisez un Gateway distant, installez/configurez le plugin sur la **machine exécutant le Gateway**, puis redémarrez le Gateway pour le charger.

## Installation

### Option A : installation depuis npm (recommandé)

```bash
openclaw plugins install @openclaw/voice-call
```

Redémarrez le Gateway ensuite.

### Option B : installation depuis un dossier local (dev, sans copie)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Redémarrez le Gateway ensuite.

## Configuration

Définissez la configuration sous `plugins.entries.voice-call.config` :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // or "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Telnyx webhook public key from the Telnyx Mission Control Portal
            // (Base64 string; can also be set via TELNYX_PUBLIC_KEY).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Webhook server
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Webhook security (recommended for tunnels/proxies)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Public exposure (pick one)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

Notes :

- Twilio/Telnyx nécessitent une URL de webhook **accessible publiquement**.
- Plivo nécessite une URL de webhook **accessible publiquement**.
- `mock` est un fournisseur de dev local (pas d'appels réseau).
- Telnyx nécessite `telnyx.publicKey` (ou `TELNYX_PUBLIC_KEY`) sauf si `skipSignatureVerification` est a true.
- `skipSignatureVerification` est réservé aux tests locaux uniquement.
- Si vous utilisez le niveau gratuit de ngrok, définissez `publicUrl` sur l'URL ngrok exacte ; la vérification de signature est toujours appliquee.
- `tunnel.allowNgrokFreeTierLoopbackBypass: true` autorisé les webhooks Twilio avec des signatures invalides **uniquement** lorsque `tunnel.provider="ngrok"` et `serve.bind` est en loopback (agent local ngrok). A utiliser uniquement pour le dev local.
- Les URL du niveau gratuit de ngrok peuvent changer ou ajouter un comportement interstitiel ; si `publicUrl` derive, les signatures Twilio echoueront. En production, préférez un domaine stable ou le funnel Tailscale.
- Valeurs par défaut de sécurité du streaming :
  - `streaming.preStartTimeoutMs` ferme les sockets qui n'envoient jamais de trame `start` valide.
  - `streaming.maxPendingConnections` limité le nombre total de sockets pre-démarrage non authentifié.
  - `streaming.maxPendingConnectionsPerIp` limité les sockets pre-démarrage non authentifiés par IP source.
  - `streaming.maxConnections` limité le nombre total de sockets de flux média ouverts (en attente + actifs).

## Nettoyeur d'appels obsoletes

Utilisez `staleCallReaperSeconds` pour mettre fin aux appels qui ne reçoivent jamais de webhook terminal
(par exemple, les appels en mode notification qui ne se terminent jamais). La valeur par défaut est `0`
(désactivé).

Plages recommandées :

- **Production :** `120`-`300` secondes pour les flux de type notification.
- Gardez cette valeur **supérieure a `maxDurationSeconds`** pour que les appels normaux puissent
  se terminer. Un bon point de depart est `maxDurationSeconds + 30-60` secondes.

Exemple :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## Sécurité des webhooks

Lorsqu'un proxy ou un tunnel se trouve devant le Gateway, le plugin reconstruit
l'URL publique pour la vérification de signature. Ces options controlent quels en-têtes
de redirection sont approuves.

`webhookSecurity.allowedHosts` etablit une liste d'autorisation des hôtes a partir des en-têtes de redirection.

`webhookSecurity.trustForwardingHeaders` approuve les en-têtes de redirection sans liste d'autorisation.

`webhookSecurity.trustedProxyIPs` approuve les en-têtes de redirection uniquement lorsque l'IP
distante de la requête correspond a la liste.

La protection contre la relecture des webhooks est activée pour Twilio et Plivo. Les requêtes de webhook
valides rejouees sont acquittees mais ignorées pour les effets secondaires.

Les tours de conversation Twilio incluent un jeton par tour dans les callbacks `<Gather>`, de sorte que
les callbacks de parole obsoletes/rejouees ne peuvent pas satisfaire un tour de transcription en attente plus recent.

Exemple avec un hôte public stable :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## TTS pour les appels

Voice Call utilisé la configuration TTS principale `messages.tts` (OpenAI ou ÉlevénLabs) pour
le streaming vocal pendant les appels. Vous pouvez la remplacer dans la configuration du plugin avec la
**même structure** : elle est fusionnée en profondeur avec `messages.tts`.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

Notes :

- **Edge TTS est ignoré pour les appels vocaux** (l'audio telephonique nécessite du PCM ; la sortie Edge n'est pas fiable).
- Le TTS principal est utilisé lorsque le streaming média Twilio est activé ; sinon les appels se rabattent sur les voix natives du fournisseur.

### Exemples supplementaires

Utiliser uniquement le TTS principal (sans remplacement) :

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

Remplacer par ÉlevénLabs uniquement pour les appels (conserver le défaut principal ailleurs) :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

Remplacer uniquement le modèle OpenAI pour les appels (exemple de fusion en profondeur) :

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## Appels entrants

La politique d'appels entrants est par défaut `disabled`. Pour activer les appels entrants, définissez :

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?",
}
```

Les réponses automatiques utilisent le système d'agent. Ajustez avec :

- `responseModel`
- `responseSystemPrompt`
- `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## Outil d'agent

Nom de l'outil : `voice_call`

Actions :

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

Ce dépôt contient une documentation de Skill correspondante dans `skills/voice-call/SKILL.md`.

## Gateway RPC

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)
