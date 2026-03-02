---
summary: "Vue d'ensemble du support des plateformes (Gateway + applications compagnon)"
read_when:
  - Recherche du support OS ou des chemins d'installation
  - Choix de l'emplacement pour exécuter le Gateway
title: "Plateformes"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/index.md
  workflow: manual
---

# Plateformes

Le coeur d'OpenClaw est écrit en TypeScript. **Node est le runtime recommandé**.
Bun n'est pas recommandé pour le Gateway (bugs WhatsApp/Telegram).

Des applications compagnon existent pour macOS (application barre de menus) et les nœuds mobiles (iOS/Android). Des applications compagnon pour Windows et
Linux sont prevues, mais le Gateway est pleinement supporte des aujourd'hui.
Des applications compagnon natives pour Windows sont également prevues ; le Gateway est recommandé via WSL2.

## Choisissez votre OS

- macOS : [macOS](/platforms/macos)
- iOS : [iOS](/platforms/ios)
- Android : [Android](/platforms/android)
- Windows : [Windows](/platforms/windows)
- Linux : [Linux](/platforms/linux)

## VPS et hébergément

- Hub VPS : [Hebergement VPS](/vps)
- Fly.io : [Fly.io](/install/fly)
- Hetzner (Docker) : [Hetzner](/install/hetzner)
- GCP (Compute Engine) : [GCP](/install/gcp)
- exe.dev (VM + proxy HTTPS) : [exe.dev](/install/exe-dev)

## Liens utiles

- Guide d'installation : [Premiers pas](/start/getting-started)
- Runbook Gateway : [Gateway](/gateway)
- Configuration du Gateway : [Configuration](/gateway/configuration)
- Statut du service : `openclaw gateway status`

## Installation du service Gateway (CLI)

Utilisez l'une de ces méthodes (toutes supportees) :

- Assistant (recommandé) : `openclaw onboard --install-daemon`
- Direct : `openclaw gateway install`
- Flux de configuration : `openclaw configuré` → selectionnez **Gateway service**
- Réparation/migration : `openclaw doctor` (propose d'installer ou de réparer le service)

La cible du service depend de l'OS :

- macOS : LaunchAgent (`ai.openclaw.gateway` ou `ai.openclaw.<profile>` ; ancien `com.openclaw.*`)
- Linux/WSL2 : service utilisateur systemd (`openclaw-gateway[-<profile>].service`)
