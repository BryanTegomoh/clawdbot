---
summary: "Comment l'application mac intègre le WebChat du gateway et comment le deboguer"
read_when:
  - Debogage de la vue WebChat mac ou du port loopback
title: "WebChat"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/webchat.md
  workflow: manual
---

# WebChat (application macOS)

L'application barre de menus macOS intègre l'UI WebChat en tant que vue SwiftUI native. Elle
se connecte au Gateway et utilise par défaut la **session principale** pour l'agent
sélectionné (avec un sélecteur de session pour les autres sessions).

- **Mode local** : se connecte directement au WebSocket du Gateway local.
- **Mode distant** : redirige le port de contrôle du Gateway via SSH et utilise ce
  tunnel comme plan de données.

## Lancement et debogage

- Manuel : menu Lobster → « Open Chat ».
- Ouverture automatique pour les tests :

  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```

- Logs : `./scripts/clawlog.sh` (sous-système `ai.openclaw`, categorie `WebChatSwiftUI`).

## Comment c'est connecté

- Plan de données : méthodes WS du Gateway `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` et événements `chat`, `agent`, `presence`, `tick`, `health`.
- Session : utilisé par défaut la session principale (`main`, ou `global` lorsque la portee est
  globale). L'UI peut basculer entre les sessions.
- L'intégration utilise une session dédiée pour garder la configuration initiale séparée.

## Surface de sécurité

- Le mode distant redirige uniquement le port WebSocket de contrôle du Gateway via SSH.

## Limitations connues

- L'UI est optimisée pour les sessions de chat (pas un bac a sable de navigateur complet).
