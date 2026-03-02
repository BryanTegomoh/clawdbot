---
summary: "Plateformes de messagerie auxquelles OpenClaw peut se connecter"
read_when:
  - Vous souhaitez choisir un canal de discussion pour OpenClaw
  - Vous avez besoin d'un aperçu rapide des plateformes de messagerie prises en charge
title: "Canaux de discussion"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/index.md
  workflow: manual
---

# Canaux de discussion

OpenClaw peut communiquer avec vous sur n'importe quelle application de messagerie que vous utilisez déjà. Chaque canal se connecte via le Gateway.
Le texte est pris en charge partout ; les médias et les reactions varient selon le canal.

## Canaux pris en charge

- [WhatsApp](/channels/whatsapp) : le plus populaire ; utilisé Baileys et nécessite un appairage par QR code.
- [Telegram](/channels/telegram) : Bot API via grammY ; prend en charge les groupes.
- [Discord](/channels/discord) : Discord Bot API + Gateway ; prend en charge les serveurs, les canaux et les messages privés.
- [IRC](/channels/irc) : serveurs IRC classiques ; canaux + messages privés avec contrôles d'appairage et de liste autorisée.
- [Slack](/channels/slack) : Bolt SDK ; applications d'espace de travail.
- [Feishu](/channels/feishu) : bot Feishu/Lark via WebSocket (plugin, installe séparément).
- [Google Chat](/channels/googlechat) : application Google Chat API via webhook HTTP.
- [Mattermost](/channels/mattermost) : Bot API + WebSocket ; canaux, groupes, messages privés (plugin, installe séparément).
- [Signal](/channels/signal) : signal-cli ; axe sur la confidentialité.
- [BlueBubbles](/channels/bluebubbles) : **Recommandé pour iMessage** ; utilisé l'API REST du serveur macOS BlueBubbles avec prise en charge complète des fonctionnalités (modification, annulation d'envoi, effets, reactions, gestion de groupe ; la modification est actuellement cassee sur macOS 26 Tahoe).
- [iMessage (ancien)](/channels/imessage) : intégration macOS historique via imsg CLI (obsolete, utilisez BlueBubbles pour les nouvelles installations).
- [Microsoft Teams](/channels/msteams) : Bot Framework ; support entreprise (plugin, installe séparément).
- [Synology Chat](/channels/synology-chat) : Synology NAS Chat via webhooks entrants+sortants (plugin, installe séparément).
- [LINE](/channels/line) : bot LINE Messaging API (plugin, installe séparément).
- [Nextcloud Talk](/channels/nextcloud-talk) : messagerie auto-hebergee via Nextcloud Talk (plugin, installe séparément).
- [Matrix](/channels/matrix) : protocole Matrix (plugin, installe séparément).
- [Nostr](/channels/nostr) : messages privés decentralises via NIP-04 (plugin, installe séparément).
- [Tlon](/channels/tlon) : messagerie basee sur Urbit (plugin, installe séparément).
- [Twitch](/channels/twitch) : chat Twitch via connexion IRC (plugin, installe séparément).
- [Zalo](/channels/zalo) : Zalo Bot API ; messagerie populaire au Vietnam (plugin, installe séparément).
- [Zalo Personnel](/channels/zalouser) : compte personnel Zalo via connexion QR (plugin, installe séparément).
- [WebChat](/web/webchat) : interface WebChat du Gateway par WebSocket.

## Notes

- Les canaux peuvent fonctionner simultanément ; configurez-en plusieurs et OpenClaw acheminera les messages par discussion.
- La configuration la plus rapide est généralement **Telegram** (simple jeton de bot). WhatsApp nécessite un appairage par QR et stocke davantage d'état sur le disque.
- Le comportement en groupe varie selon le canal ; voir [Groupes](/channels/groups).
- L'appairage des messages privés et les listes autorisées sont appliques pour la sécurité ; voir [Sécurité](/gateway/security).
- Details internes de Telegram : [notes grammY](/channels/grammy).
- Dépannage : [Dépannage des canaux](/channels/troubleshooting).
- Les fournisseurs de modèles sont documentes séparément ; voir [Fournisseurs de modèles](/providers/models).
