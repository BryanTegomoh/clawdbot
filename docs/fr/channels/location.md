---
summary: "Analyse de localisation des canaux entrants (Telegram + WhatsApp) et champs de contexte"
read_when:
  - Ajout ou modification de l'analyse de localisation des canaux
  - Utilisation des champs de contexte de localisation dans les prompts ou outils de l'agent
title: "Analyse de localisation des canaux"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/location.md"
  workflow: "manual"
---

# Analyse de localisation des canaux

OpenClaw normalise les localisations partagees depuis les canaux de chat en :

- texte lisible par l'humain ajouté au corps du message entrant, et
- champs structures dans le payload de contexte de réponse automatique.

Actuellement pris en charge :

- **Telegram** (epingles de localisation + lieux + localisations en direct)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` avec `geo_uri`)

## Formatage du texte

Les localisations sont rendues sous forme de lignes conviviales sans crochets :

- Epingle :
  - `📍 48.858844, 2.294351 ±12m`
- Lieu nomme :
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- Partage en direct :
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

Si le canal inclut une légende/commentaire, il est ajouté sur la ligne suivante :

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## Champs de contexte

Quand une localisation est présenté, ces champs sont ajoutés a `ctx` :

- `LocationLat` (number)
- `LocationLon` (number)
- `LocationAccuracy` (number, metres ; optionnel)
- `LocationName` (string ; optionnel)
- `LocationAddress` (string ; optionnel)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)

## Notes par canal

- **Telegram** : les lieux sont associés a `LocationName/LocationAddress` ; les localisations en direct utilisent `live_period`.
- **WhatsApp** : `locationMessage.comment` et `liveLocationMessage.caption` sont ajoutés comme ligne de légende.
- **Matrix** : `geo_uri` est analyse comme une localisation epinglee ; l'altitude est ignorée et `LocationIsLive` est toujours false.
