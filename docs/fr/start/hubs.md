---
read_when:
  - Vous souhaitez une carte complète de la documentation
summary: Centres de documentation qui renvoient vers chaque page de la documentation OpenClaw
title: Centres de documentation
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/hubs.md
  workflow: manual
---

# Centres de documentation

<Note>
Si vous êtes nouveau sur OpenClaw, commencez par [Premiers pas](/start/getting-started).
</Note>

Utilisez ces centres pour découvrir chaque page, y compris les guides approfondis et la documentation de référence qui n'apparaissent pas dans la navigation latérale.

## Commencer ici

- [Accueil](/)
- [Premiers pas](/start/getting-started)
- [Démarrage rapide](/start/quickstart)
- [Configuration initiale](/start/onboarding)
- [Assistant](/start/wizard)
- [Configuration](/start/setup)
- [Tableau de bord (Gateway local)](http://127.0.0.1:18789/)
- [Aide](/help)
- [Répertoire de la documentation](/start/docs-directory)
- [Configuration](/gateway/configuration)
- [Exemples de configuration](/gateway/configuration-examples)
- [Assistant OpenClaw](/start/openclaw)
- [Vitrine](/start/showcase)
- [Légendes](/start/lore)

## Installation et mises à jour

- [Docker](/install/docker)
- [Nix](/install/nix)
- [Mise à jour / rollback](/install/updating)
- [Flux Bun (expérimental)](/install/bun)

## Concepts fondamentaux

- [Architecture](/concepts/architecture)
- [Fonctionnalités](/concepts/features)
- [Centre réseau](/network)
- [Runtime de l'agent](/concepts/agent)
- [Espace de travail de l'agent](/concepts/agent-workspace)
- [Mémoire](/concepts/memory)
- [Boucle de l'agent](/concepts/agent-loop)
- [Streaming et découpage](/concepts/streaming)
- [Routage multi-agent](/concepts/multi-agent)
- [Compaction](/concepts/compaction)
- [Sessions](/concepts/session)
- [Sessions (alias)](/concepts/sessions)
- [Élagage de sessions](/concepts/session-pruning)
- [Outils de session](/concepts/session-tool)
- [File d'attente](/concepts/queue)
- [Commandes slash](/tools/slash-commands)
- [Adaptateurs RPC](/reference/rpc)
- [Schémas TypeBox](/concepts/typebox)
- [Gestion des fuseaux horaires](/concepts/timezone)
- [Présence](/concepts/presence)
- [Découverte + protocoles de transport](/gateway/discovery)
- [Bonjour](/gateway/bonjour)
- [Routage de canaux](/channels/channel-routing)
- [Groupes](/channels/groups)
- [Messages de groupe](/channels/group-messages)
- [Basculement de modèle](/concepts/model-failover)
- [OAuth](/concepts/oauth)

## Fournisseurs et entrée

- [Centre des canaux de chat](/channels)
- [Centre des fournisseurs de modèles](/providers/models)
- [WhatsApp](/channels/whatsapp)
- [Telegram](/channels/telegram)
- [Telegram (notes grammY)](/channels/grammy)
- [Slack](/channels/slack)
- [Discord](/channels/discord)
- [Mattermost](/channels/mattermost) (plugin)
- [Signal](/channels/signal)
- [BlueBubbles (iMessage)](/channels/bluebubbles)
- [iMessage (ancien)](/channels/imessage)
- [Analyse de localisation](/channels/location)
- [WebChat](/web/webchat)
- [Webhooks](/automation/webhook)
- [Gmail Pub/Sub](/automation/gmail-pubsub)

## Gateway et opérations

- [Guide opérationnel du Gateway](/gateway)
- [Modèle réseau](/gateway/network-model)
- [Appairage du Gateway](/gateway/pairing)
- [Verrouillage du Gateway](/gateway/gateway-lock)
- [Processus en arrière-plan](/gateway/background-process)
- [Santé](/gateway/health)
- [Heartbeat](/gateway/heartbeat)
- [Doctor](/gateway/doctor)
- [Journalisation](/gateway/logging)
- [Isolation en bac à sable](/gateway/sandboxing)
- [Tableau de bord](/web/dashboard)
- [Interface de contrôle](/web/control-ui)
- [Accès distant](/gateway/remote)
- [README du gateway distant](/gateway/remote-gateway-readme)
- [Tailscale](/gateway/tailscale)
- [Sécurité](/gateway/security)
- [Dépannage](/gateway/troubleshooting)

## Outils et automatisation

- [Surface des outils](/tools)
- [OpenProse](/prose)
- [Référence CLI](/cli)
- [Outil exec](/tools/exec)
- [Mode élevé](/tools/elevated)
- [Tâches cron](/automation/cron-jobs)
- [Cron vs Heartbeat](/automation/cron-vs-heartbeat)
- [Réflexion et mode verbeux](/tools/thinking)
- [Modèles](/concepts/models)
- [Sous-agents](/tools/subagents)
- [CLI agent send](/tools/agent-send)
- [Interface terminal](/web/tui)
- [Contrôle du navigateur](/tools/browser)
- [Navigateur (dépannage Linux)](/tools/browser-linux-troubleshooting)
- [Sondages](/automation/poll)

## Nœuds, médias, voix

- [Vue d'ensemble des nœuds](/nodes)
- [Caméra](/nodes/camera)
- [Images](/nodes/images)
- [Audio](/nodes/audio)
- [Commande de localisation](/nodes/location-command)
- [Réveil vocal](/nodes/voicewake)
- [Mode conversation](/nodes/talk)

## Plateformes

- [Vue d'ensemble des plateformes](/platforms)
- [macOS](/platforms/macos)
- [iOS](/platforms/ios)
- [Android](/platforms/android)
- [Windows (WSL2)](/platforms/windows)
- [Linux](/platforms/linux)
- [Surfaces Web](/web)

## Application compagnon macOS (avancé)

- [Configuration de développement macOS](/platforms/mac/dev-setup)
- [Barre de menus macOS](/platforms/mac/menu-bar)
- [Réveil vocal macOS](/platforms/mac/voicewake)
- [Overlay vocal macOS](/platforms/mac/voice-overlay)
- [WebChat macOS](/platforms/mac/webchat)
- [Canvas macOS](/platforms/mac/canvas)
- [Processus enfant macOS](/platforms/mac/child-process)
- [Santé macOS](/platforms/mac/health)
- [Icône macOS](/platforms/mac/icon)
- [Journalisation macOS](/platforms/mac/logging)
- [Permissions macOS](/platforms/mac/permissions)
- [Distant macOS](/platforms/mac/remote)
- [Signature macOS](/platforms/mac/signing)
- [Release macOS](/platforms/mac/release)
- [Gateway macOS (launchd)](/platforms/mac/bundled-gateway)
- [XPC macOS](/platforms/mac/xpc)
- [Skills macOS](/platforms/mac/skills)
- [Peekaboo macOS](/platforms/mac/peekaboo)

## Espace de travail et modèles

- [Skills](/tools/skills)
- [ClawHub](/tools/clawhub)
- [Configuration des Skills](/tools/skills-config)
- [AGENTS par défaut](/reference/AGENTS.default)
- [Modèles : AGENTS](/reference/templates/AGENTS)
- [Modèles : BOOTSTRAP](/reference/templates/BOOTSTRAP)
- [Modèles : HEARTBEAT](/reference/templates/HEARTBEAT)
- [Modèles : IDENTITY](/reference/templates/IDENTITY)
- [Modèles : SOUL](/reference/templates/SOUL)
- [Modèles : TOOLS](/reference/templates/TOOLS)
- [Modèles : USER](/reference/templates/USER)

## Expériences (exploratoire)

- [Protocole de configuration initiale](/experiments/onboarding-config-protocol)
- [Recherche : mémoire](/experiments/research/memory)
- [Exploration de configuration de modèle](/experiments/proposals/model-config)

## Projet

- [Crédits](/reference/credits)

## Tests et release

- [Tests](/reference/test)
- [Checklist de release](/reference/RELEASING)
- [Modèles d'appareils](/reference/device-models)
