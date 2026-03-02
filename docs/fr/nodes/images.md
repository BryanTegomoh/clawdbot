---
summary: "Règles de gestion des images et médias pour l'envoi, le gateway et les réponses de l'agent"
read_when:
  - Modification du pipeline de médias ou des pièces jointes
title: "Support images et médias"
sidebarTitle: "Support images et médias"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/images.md
  workflow: manual
---

# Support images et médias — 2025-12-05

Le canal WhatsApp fonctionne via **Baileys Web**. Ce document décrit les règles actuelles de gestion des médias pour l'envoi, le gateway et les réponses de l'agent.

## Objectifs

- Envoyer des médias avec des légendes optionnelles via `openclaw message send --média`.
- Permettre aux réponses automatiques de la boîte de réception web d'inclure des médias avec du texte.
- Maintenir des limités par type saines et prévisibles.

## Surface CLI

- `openclaw message send --média <path-or-url> [--message <caption>]`
  - `--media` optionnel ; la légende peut être vide pour les envois de médias seuls.
  - `--dry-run` affiche la charge utile résolue ; `--json` émet `{ channel, to, messageId, mediaUrl, caption }`.

## Comportement du canal WhatsApp Web

- Entrée : chemin de fichier local **ou** URL HTTP(S).
- Flux : charger dans un Buffer, détecter le type de média et construire la charge utile appropriée :
  - **Images :** redimensionner et recompresser en JPEG (côté max 2048px) ciblant `agents.defaults.mediaMaxMb` (par défaut 5 Mo), plafonné à 6 Mo.
  - **Audio/Voix/Vidéo :** passage direct jusqu'à 16 Mo ; l'audio est envoyé comme note vocale (`ptt: true`).
  - **Documents :** tout le reste, jusqu'à 100 Mo, avec le nom de fichier préservé lorsque disponible.
- Lecture en boucle style GIF WhatsApp : envoyer un MP4 avec `gifPlayback: true` (CLI : `--gif-playback`) pour que les clients mobiles le lisent en boucle en ligne.
- La détection MIME préfère les octets magiques, puis les en-têtes, puis l'extension de fichier.
- La légende provient de `--message` ou `reply.text` ; une légende vide est autorisée.
- Journalisation : le mode non-verbeux affiche `↩️`/`✅` ; le mode verbeux inclut la taille et le chemin/URL source.

## Pipeline de réponse automatique

- `getReplyFromConfig` retourne `{ text?, mediaUrl?, mediaUrls? }`.
- Lorsque des médias sont présents, l'expéditeur web résout les chemins locaux ou URL en utilisant le même pipeline que `openclaw message send`.
- Les entrées de médias multiples sont envoyées séquentiellement si fournies.

## Médias entrants vers commandes (Pi)

- Lorsque les messages web entrants incluent des médias, OpenClaw télécharge dans un fichier temporaire et expose des variables de modèle :
  - `{{MediaUrl}}` pseudo-URL pour le média entrant.
  - `{{MediaPath}}` chemin temporaire local écrit avant l'exécution de la commande.
- Lorsqu'un bac à sable Docker par session est activé, les médias entrants sont copiés dans l'espace de travail du bac à sable et `MediaPath`/`MediaUrl` sont réécrits vers un chemin relatif comme `média/inbound/<filename>`.
- La compréhension des médias (si configurée via `tools.média.*` ou `tools.média.models` partagé) s'exécute avant le modèle et peut insérer des blocs `[Image]`, `[Audio]` et `[Video]` dans `Body`.
  - L'audio définit `{{Transcript}}` et utilise la transcription pour l'analyse des commandes afin que les commandes slash continuent de fonctionner.
  - Les descriptions de vidéo et d'image préservent le texte de légende pour l'analyse des commandes.
- Par défaut, seule la première pièce jointe image/audio/vidéo correspondante est traitée ; définissez `tools.média.<cap>.attachments` pour traiter plusieurs pièces jointes.

## Limités et erreurs

**Limités d'envoi sortant (envoi WhatsApp web)**

- Images : limité de ~6 Mo après recompression.
- Audio/voix/vidéo : limité de 16 Mo ; documents : limité de 100 Mo.
- Média surdimensionné ou illisible → erreur claire dans les journaux et la réponse est ignorée.

**Limités de compréhension des médias (transcription/description)**

- Image par défaut : 10 Mo (`tools.média.image.maxBytes`).
- Audio par défaut : 20 Mo (`tools.média.audio.maxBytes`).
- Vidéo par défaut : 50 Mo (`tools.média.video.maxBytes`).
- Les médias surdimensionnés ignorent la compréhension, mais les réponses passent toujours avec le corps original.

## Notes pour les tests

- Couvrir les flux d'envoi + réponse pour les cas image/audio/document.
- Valider la recompression pour les images (limité de taille) et le drapeau note vocale pour l'audio.
- S'assurer que les réponses multi-médias sont envoyées sous forme d'envois séquentiels.
