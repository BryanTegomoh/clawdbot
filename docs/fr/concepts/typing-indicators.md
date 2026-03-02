---
summary: "Quand OpenClaw affiche les indicateurs de saisie et comment les ajuster"
read_when:
  - Modification du comportement ou des valeurs par défaut des indicateurs de saisie
title: "Indicateurs de saisie"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/typing-indicators.md
  workflow: manual
---

# Indicateurs de saisie

Les indicateurs de saisie sont envoyés au canal de chat pendant qu'une exécution est activé. Utilisez `agents.defaults.typingMode` pour contrôler **quand** la saisie démarre et `typingIntervalSeconds` pour contrôler **a quelle fréquence** elle se rafraichit.

## Valeurs par défaut

Lorsque `agents.defaults.typingMode` n'est **pas défini**, OpenClaw conserve le comportement existant :

- **Chats directs** : la saisie démarre immédiatement des que la boucle du modèle commence.
- **Chats de groupe avec mention** : la saisie démarre immédiatement.
- **Chats de groupe sans mention** : la saisie ne démarre que lorsque le texte du message commence à être diffusé en continu.
- **Executions heartbeat** : la saisie est désactivée.

## Modes

Définissez `agents.defaults.typingMode` sur l'un des suivants :

- `never` : aucun indicateur de saisie, jamais.
- `instant` : démarrer la saisie **des que la boucle du modèle commence**, même si l'exécution ne retourne finalement que le token de réponse silencieuse.
- `thinking` : démarrer la saisie au **premier delta de raisonnement** (nécessite `reasoningLevel: "stream"` pour l'exécution).
- `message` : démarrer la saisie au **premier delta de texte non silencieux** (ignoré le token silencieux `NO_REPLY`).

Ordre de « précocité de déclenchement » :
`never` → `message` → `thinking` → `instant`

## Configuration

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6,
  },
}
```

Vous pouvez surcharger le mode ou la cadence par session :

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4,
  },
}
```

## Notes

- Le mode `message` n'affichera pas la saisie pour les réponses uniquement silencieuses (par ex. le token `NO_REPLY` utilisé pour supprimer la sortie).
- `thinking` ne se déclenche que si l'exécution diffuse le raisonnement en continu (`reasoningLevel: "stream"`). Si le modèle n'émet pas de deltas de raisonnement, la saisie ne démarrera pas.
- Les heartbeats n'affichent jamais la saisie, quel que soit le mode.
- `typingIntervalSeconds` contrôle la **cadence de rafraichissement**, pas le moment de démarrage. La valeur par défaut est de 6 secondes.
