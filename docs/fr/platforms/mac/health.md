---
summary: "Comment l'application macOS rapporte les états de sante du gateway/Baileys"
read_when:
  - Debogage des indicateurs de sante de l'application mac
title: "Vérifications de sante"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/health.md
  workflow: manual
---

# Vérifications de sante sur macOS

Comment voir si le canal lie est sain depuis l'application barre de menus.

## Barre de menus

- Le point de statut reflete maintenant la sante de Baileys :
  - Vert : lié + socket ouvert recemment.
  - Orange : connexion en cours/nouvelle tentative.
  - Rouge : déconnecté ou sonde echouee.
- La ligne secondaire affiche « linked · auth 12m » ou la raison de l'échec.
- L'élément de menu « Run Health Check » déclenché une sonde a la demande.

## Paramètres

- L'onglet Général obtient une carte Sante affichant : age de l'authentification liée, chemin/nombre du magasin de sessions, heure de la dernière vérification, dernière erreur/code de statut, et boutons pour « Run Health Check » / « Reveal Logs ».
- Utilisé un instantane en cache pour que l'UI se charge instantanement et se degrade gracieusement hors ligne.
- L'**onglet Canaux** affiche le statut des canaux + contrôles pour WhatsApp/Telegram (QR de connexion, déconnexion, sonde, dernière déconnexion/erreur).

## Fonctionnement de la sonde

- L'application exécute `openclaw health --json` via `ShellExecutor` toutes les ~60s et a la demande. La sonde charge les identifiants et rapporte le statut sans envoyer de messages.
- Met en cache le dernier bon instantane et la dernière erreur séparément pour eviter le scintillement ; affiche l'horodatage de chacun.

## En cas de doute

- Vous pouvez toujours utiliser le flux CLI dans [Sante du Gateway](/gateway/health) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) et surveiller `/tmp/openclaw/openclaw-*.log` pour `web-heartbeat` / `web-reconnect`.
