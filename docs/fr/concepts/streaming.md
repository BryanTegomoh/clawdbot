---
summary: "Comportement du streaming + découpe (réponses par blocs, streaming d'aperçu sur les canaux, correspondance des modes)"
read_when:
  - Explication du fonctionnement du streaming ou de la découpe sur les canaux
  - Modification du streaming par blocs ou du comportement de découpe des canaux
  - Débogage des réponses par blocs dupliquées/prématurées ou du streaming d'aperçu des canaux
title: "Streaming et découpe"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/streaming.md
  workflow: manual
---

# Streaming + découpe

OpenClaw dispose de deux couches de streaming séparées :

- **Streaming par blocs (canaux) :** émet des **blocs** terminés au fur et à mesure que l'assistant écrit. Ce sont des messages de canal normaux (pas des deltas de tokens).
- **Streaming d'aperçu (Telegram/Discord/Slack) :** met à jour un **message d'aperçu** temporaire pendant la génération.

Il n'y a **pas de vrai streaming delta de tokens** vers les messages de canal aujourd'hui. Le streaming d'aperçu est basé sur les messages (envoi + modifications/ajouts).

## Streaming par blocs (messages de canal)

Le streaming par blocs envoie la sortie de l'assistant en segments grossiers au fur et à mesure qu'elle devient disponible.

```
Sortie du modèle
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ le découpeur émet des blocs au fur et à mesure que le tampon grossit
       └─ (blockStreamingBreak=message_end)
            └─ le découpeur vide a message_end
                   └─ envoi au canal (réponses par blocs)
```

Légende :

- `text_delta/events` : événements du flux du modèle (peuvent être épars pour les modèles sans streaming).
- `chunker` : `EmbeddedBlockChunker` appliquant des bornes min/max + préférence de coupure.
- `channel send` : messages sortants réels (réponses par blocs).

**Contrôles :**

- `agents.defaults.blockStreamingDefault` : `"on"`/`"off"` (par défaut off).
- Surcharges par canal : `*.blockStreaming` (et variantes par compte) pour forcer `"on"`/`"off"` par canal.
- `agents.defaults.blockStreamingBreak` : `"text_end"` ou `"message_end"`.
- `agents.defaults.blockStreamingChunk` : `{ minChars, maxChars, breakPreference? }`.
- `agents.defaults.blockStreamingCoalesce` : `{ minChars?, maxChars?, idleMs? }` (fusionner les blocs diffusés avant l'envoi).
- Plafond dur par canal : `*.textChunkLimit` (par ex. `channels.whatsapp.textChunkLimit`).
- Mode de découpe par canal : `*.chunkMode` (`length` par défaut, `newline` découpe sur les lignes vides (limités de paragraphe) avant la découpe par longueur).
- Plafond souple Discord : `channels.discord.maxLinesPerMessage` (par défaut 17) découpe les réponses trop longues pour éviter le rognage de l'interface.

**Sémantique des limités :**

- `text_end` : diffuser les blocs en continu dès que le découpeur émet ; vider à chaque `text_end`.
- `message_end` : attendre la fin du message de l'assistant, puis vider la sortie mise en tampon.

`message_end` utilisé quand même le découpeur si le texte tamisé dépasse `maxChars`, donc il peut émettre plusieurs segments à la fin.

## Algorithme de découpe (bornes basse/haute)

La découpe par blocs est implémentée par `EmbeddedBlockChunker` :

- **Borne basse :** ne pas émettre tant que le tampon < `minChars` (sauf si force).
- **Borne haute :** préférer les coupures avant `maxChars` ; si force, couper à `maxChars`.
- **Préférence de coupure :** `paragraph` → `newline` → `sentence` → `whitespace` → coupure dure.
- **Blocs de code :** ne jamais couper à l'intérieur des blocs de code ; lorsque forcé à `maxChars`, fermer + rouvrir le bloc pour maintenir le Markdown valide.

`maxChars` est plafonné au `textChunkLimit` du canal, donc vous ne pouvez pas dépasser les plafonds par canal.

## Coalescence (fusion des blocs diffusés)

Lorsque le streaming par blocs est activé, OpenClaw peut **fusionner les segments de blocs consécutifs** avant de les envoyer. Cela réduit le « spam de lignes uniques » tout en fournissant une sortie progressive.

- La coalescence attend des **intervalles d'inactivité** (`idleMs`) avant de vider.
- Les tampons sont plafonnés par `maxChars` et sont vides s'ils le dépassent.
- `minChars` empêche les petits fragments d'être envoyés tant que suffisamment de texte n'est pas accumulé (le vidage final envoie toujours le texte restant).
- Le séparateur est dérivé de `blockStreamingChunk.breakPreference` (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → espace).
- Des surcharges par canal sont disponibles via `*.blockStreamingCoalesce` (y compris les configurations par compte).
- Le `minChars` de coalescence par défaut est augmenté à 1500 pour Signal/Slack/Discord sauf surcharge.

## Cadence de type humain entre les blocs

Lorsque le streaming par blocs est activé, vous pouvez ajouter une **pause aléatoire** entre les réponses par blocs (après le premier bloc). Cela rend les réponses multi-bulles plus naturelles.

- Configuration : `agents.defaults.humanDelay` (surcharge par agent via `agents.list[].humanDelay`).
- Modes : `off` (par défaut), `natural` (800-2500 ms), `custom` (`minMs`/`maxMs`).
- S'applique uniquement aux **réponses par blocs**, pas aux réponses finales ni aux résumés d'outils.

## « Diffuser les segments ou tout »

Cela correspond à :

- **Diffuser les segments :** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (émettre au fur et à mesure). Les canaux non-Telegram nécessitent également `*.blockStreaming: true`.
- **Diffuser tout à la fin :** `blockStreamingBreak: "message_end"` (vider une fois, possiblement plusieurs segments si très long).
- **Pas de streaming par blocs :** `blockStreamingDefault: "off"` (uniquement la réponse finale).

**Note sur les canaux :** Le streaming par blocs est **désactivé sauf si** `*.blockStreaming` est explicitement défini à `true`. Les canaux peuvent diffuser un aperçu en direct (`channels.<channel>.streaming`) sans réponses par blocs.

Rappel de l'emplacement de la configuration : les valeurs par défaut `blockStreaming*` se trouvent sous `agents.defaults`, pas à la racine de la configuration.

## Modes de streaming d'aperçu

Clé canonique : `channels.<channel>.streaming`

Modes :

- `off` : désactiver le streaming d'aperçu.
- `partial` : un seul aperçu remplacé par le dernier texte.
- `block` : mises à jour d'aperçu en étapes découpées/ajoutées.
- `progress` : aperçu de progression/statut pendant la génération, réponse finale à la fin.

### Correspondance des canaux

| Canal    | `off` | `partial` | `block` | `progress`              |
| -------- | ----- | --------- | ------- | ----------------------- |
| Telegram | ✅    | ✅        | ✅      | correspond à `partial`  |
| Discord  | ✅    | ✅        | ✅      | correspond à `partial`  |
| Slack    | ✅    | ✅        | ✅      | ✅                      |

Spécifique à Slack :

- `channels.slack.nativeStreaming` activé les appels API de streaming natif Slack lorsque `streaming=partial` (par défaut : `true`).

Migration des clés héritées :

- Telegram : `streamMode` + boolean `streaming` migrent automatiquement vers l'enum `streaming`.
- Discord : `streamMode` + boolean `streaming` migrent automatiquement vers l'enum `streaming`.
- Slack : `streamMode` migre automatiquement vers l'enum `streaming` ; boolean `streaming` migre automatiquement vers `nativeStreaming`.

### Comportement à l'exécution

Telegram :

- Utilisé l'API Bot `sendMessage` + `editMessageText`.
- Le streaming d'aperçu est ignoré lorsque le streaming par blocs Telegram est explicitement activé (pour éviter le double-streaming).
- `/reasoning stream` peut écrire le raisonnement dans l'aperçu.

Discord :

- Utilisé envoi + modification des messages d'aperçu.
- Le mode `block` utilisé la découpe de brouillon (`draftChunk`).
- Le streaming d'aperçu est ignoré lorsque le streaming par blocs Discord est explicitement activé.

Slack :

- `partial` peut utiliser le streaming natif Slack (`chat.startStream`/`append`/`stop`) lorsque disponible.
- `block` utilisé des aperçus de brouillon en mode ajout.
- `progress` utilisé un texte d'aperçu de statut, puis la réponse finale.
