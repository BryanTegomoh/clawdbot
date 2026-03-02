---
summary: "Sémantique des réactions partagée entre les canaux"
read_when:
  - Travail sur les réactions dans n'importe quel canal
title: "Réactions"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/reactions.md
  workflow: manual
---

# Outillage des réactions

Sémantique des réactions partagée entre les canaux :

- `emoji` est obligatoire lors de l'ajout d'une réaction.
- `emoji=""` supprimé la ou les réactions du bot lorsque c'est supporté.
- `remove: true` supprimé l'emoji spécifié lorsque c'est supporté (nécessite `emoji`).

Notes par canal :

- **Discord/Slack** : un `emoji` vide supprimé toutes les réactions du bot sur le message ; `remove: true` supprimé uniquement cet emoji.
- **Google Chat** : un `emoji` vide supprimé les réactions de l'application sur le message ; `remove: true` supprimé uniquement cet emoji.
- **Telegram** : un `emoji` vide supprimé les réactions du bot ; `remove: true` supprimé également les réactions mais nécessite quand même un `emoji` non vide pour la validation de l'outil.
- **WhatsApp** : un `emoji` vide supprimé la réaction du bot ; `remove: true` correspond à un emoji vide (nécessite quand même `emoji`).
- **Signal** : les notifications de réactions entrantes émettent des événements système lorsque `channels.signal.reactionNotifications` est activé.
