---
read_when:
  - Vous souhaitez une liste complète de ce que supporte OpenClaw
summary: Fonctionnalités d'OpenClaw pour les canaux, le routage, les médias et l'expérience utilisateur.
title: Fonctionnalités
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/features.md
  workflow: manual
---

## Points forts

<Columns>
  <Card title="Canaux" icon="message-square">
    WhatsApp, Telegram, Discord et iMessage avec un seul Gateway.
  </Card>
  <Card title="Plugins" icon="plug">
    Ajoutez Mattermost et d'autres avec des extensions.
  </Card>
  <Card title="Routage" icon="route">
    Routage multi-agent avec sessions isolées.
  </Card>
  <Card title="Prise en charge des médias" icon="image">
    Images, audio et documents en entrée et en sortie.
  </Card>
  <Card title="Applications et interface" icon="monitor">
    Interface de contrôle Web et application compagnon macOS.
  </Card>
  <Card title="Nœuds mobiles" icon="smartphone">
    Nœuds iOS et Android avec support Canvas.
  </Card>
</Columns>

## Liste complète

- Intégration WhatsApp via WhatsApp Web (Baileys)
- Support bot Telegram (grammY)
- Support bot Discord (channels.discord.js)
- Support bot Mattermost (plugin)
- Intégration iMessage via CLI imsg local (macOS)
- Pont agent pour Pi en mode RPC avec streaming d'outils
- Streaming et découpage pour les réponses longues
- Routage multi-agent pour sessions isolées par espace de travail ou expéditeur
- Authentification par abonnement pour Anthropic et OpenAI via OAuth
- Sessions : les discussions directes se regroupent dans `main` ; les groupes sont isolés
- Support des discussions de groupe avec activation par mention
- Prise en charge des médias pour images, audio et documents
- Hook optionnel de transcription des notes vocales
- WebChat et application barre de menus macOS
- Nœud iOS avec appairage et surface Canvas
- Nœud Android avec appairage, Canvas, chat et caméra

<Note>
Les anciens chemins Claude, Codex, Gemini et Opencode ont été supprimés. Pi est le seul
chemin d'agent de programmation.
</Note>
