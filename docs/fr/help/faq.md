---
summary: "Questions fréquemment posées sur l'installation, la configuration et l'utilisation d'OpenClaw"
read_when:
  - Répondre aux questions courantes d'installation, de configuration initiale ou de support à l'exécution
  - Trier les problèmes signalés par les utilisateurs avant un débogage approfondi
title: "FAQ"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: help/faq.md
  workflow: manual
---

# FAQ

Réponses rapides et dépannage approfondi pour les configurations réelles (dev local, VPS, multi-agent, OAuth/clés API, basculement de modèle). Pour les diagnostics à l'exécution, voir [Dépannage](/gateway/troubleshooting). Pour la référence complète de configuration, voir [Configuration](/gateway/configuration).

## Table des matières

- [Démarrage rapide et première configuration]
  - [Je suis bloqué, quel est le moyen le plus rapide de me débloquer ?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [Quelle est la méthode recommandée pour installer et configurer OpenClaw ?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Comment ouvrir le tableau de bord après la configuration initiale ?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Comment authentifier le tableau de bord (jeton) en localhost vs distant ?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Quel runtime me faut-il ?](#what-runtime-do-i-need)
  - [Est-ce que ça tourne sur Raspberry Pi ?](#does-it-run-on-raspberry-pi)
  - [Des conseils pour les installations Raspberry Pi ?](#any-tips-for-raspberry-pi-installs)
  - [C'est bloqué sur « wake up my friend » / la configuration initiale ne démarre pas. Que faire ?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Puis-je migrer ma configuration vers une nouvelle machine (Mac mini) sans refaire la configuration initiale ?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Où voir les nouveautés de la dernière version ?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [Je ne peux pas accéder à docs.openclaw.ai (erreur SSL). Que faire ?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Quelle est la différence entre stable et beta ?](#whats-the-difference-between-stable-and-beta)
  - [Comment installer la version beta, et quelle est la différence entre beta et dev ?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [Comment essayer les dernières modifications ?](#how-do-i-try-the-latest-bits)
  - [Combien de temps prennent l'installation et la configuration initiale ?](#how-long-does-install-and-onboarding-usually-take)
  - [L'installeur est bloqué ? Comment obtenir plus de retour ?](#installer-stuck-how-do-i-get-more-feedback)
  - [L'installation Windows dit git non trouvé ou openclaw non reconnu](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [La documentation n'a pas répondu à ma question, comment obtenir une meilleure réponse ?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Comment installer OpenClaw sur Linux ?](#how-do-i-install-openclaw-on-linux)
  - [Comment installer OpenClaw sur un VPS ?](#how-do-i-install-openclaw-on-a-vps)
  - [Où sont les guides d'installation cloud/VPS ?](#where-are-the-cloudvps-install-guides)
  - [Puis-je demander à OpenClaw de se mettre à jour ?](#can-i-ask-openclaw-to-update-itself)
  - [Que fait réellement l'assistant de configuration initiale ?](#what-does-the-onboarding-wizard-actually-do)
  - [Ai-je besoin d'un abonnement Claude ou OpenAI pour utiliser ceci ?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Puis-je utiliser l'abonnement Claude Max sans clé API ?](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Comment fonctionne l'authentification « setup-token » Anthropic ?](#how-does-anthropic-setuptoken-auth-work)
  - [Où trouver un setup-token Anthropic ?](#where-do-i-find-an-anthropic-setuptoken)
  - [Supportez-vous l'authentification par abonnement Claude (Claude Pro ou Max) ?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Pourquoi est-ce que je vois `HTTP 429: rate_limit_error` depuis Anthropic ?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [AWS Bedrock est-il supporte ?](#is-aws-bedrock-supported)
  - [Comment fonctionne l'authentification Codex ?](#how-does-codex-auth-work)
  - [Supportez-vous l'authentification par abonnement OpenAI (Codex OAuth) ?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Comment configurer Gemini CLI OAuth ?](#how-do-i-set-up-gemini-cli-oauth)
  - [Un modèle local convient-il pour du chat casual ?](#is-a-local-model-ok-for-casual-chats)
  - [Comment garder le trafic de modèles hébergés dans une région spécifique ?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Dois-je acheter un Mac Mini pour installer ceci ?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Ai-je besoin d'un Mac mini pour le support iMessage ?](#do-i-need-a-mac-mini-for-imessage-support)
  - [Si j'achète un Mac mini pour OpenClaw, puis-je le connecter à mon MacBook Pro ?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Puis-je utiliser Bun ?](#can-i-use-bun)
  - [Telegram : que mettre dans `allowFrom` ?](#telegram-what-goes-in-allowfrom)
  - [Plusieurs personnes peuvent-elles utiliser un numéro WhatsApp avec différentes instances OpenClaw ?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Puis-je avoir un agent « chat rapide » et un agent « Opus pour le codage » ?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Homebrew fonctionne-t-il sur Linux ?](#does-homebrew-work-on-linux)
  - [Quelle est la différence entre l'installation hackable (git) et l'installation npm ?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Puis-je basculer entre les installations npm et git plus tard ?](#can-i-switch-between-npm-and-git-installs-later)
  - [Dois-je exécuter le Gateway sur mon laptop ou un VPS ?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [À quel point est-il important d'exécuter OpenClaw sur une machine dédiée ?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Quels sont les prérequis VPS minimaux et l'OS recommandé ?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Puis-je exécuter OpenClaw dans une VM et quels sont les prérequis ?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [Qu'est-ce qu'OpenClaw ?](#what-is-openclaw)
  - [Qu'est-ce qu'OpenClaw, en un paragraphe ?](#what-is-openclaw-in-one-paragraph)
  - [Quelle est la proposition de valeur ?](#whats-the-value-proposition)
  - [Je viens de l'installer, que dois-je faire en premier ?](#i-just-set-it-up-what-should-i-do-first)
  - [Quels sont les cinq cas d'usage quotidiens principaux d'OpenClaw ?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [OpenClaw peut-il aider pour la génération de leads, la prospection, les pubs et les blogs pour un SaaS ?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Quels sont les avantages par rapport à Claude Code pour le développement web ?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills et automatisation](#skills-and-automation)
  - [Comment personnaliser les Skills sans salir le dépôt ?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Puis-je charger des Skills depuis un dossier personnalisé ?](#can-i-load-skills-from-a-custom-folder)
  - [Comment utiliser différents modèles pour différentes tâches ?](#how-can-i-use-different-models-for-different-tasks)
  - [Le bot se fige pendant un travail lourd. Comment déléguer ça ?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Le cron ou les rappels ne se déclenchent pas. Que vérifier ?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Comment installer des Skills sur Linux ?](#how-do-i-install-skills-on-linux)
  - [OpenClaw peut-il exécuter des tâches selon un planning ou en continu en arrière-plan ?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Puis-je exécuter des Skills macOS uniquement depuis Linux ?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Avez-vous une intégration Notion ou HeyGen ?](#do-you-have-a-notion-or-heygen-intégration)
  - [Comment installer l'extension Chrome pour la prise de contrôle du navigateur ?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Bac à sable et mémoire](#sandboxing-and-memory)
  - [Existe-t-il une documentation dédiée au bac à sable ?](#is-there-a-dedicated-sandboxing-doc)
  - [Comment monter un dossier hôte dans le bac à sable ?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [Comment fonctionne la mémoire ?](#how-does-memory-work)
  - [La mémoire oublie sans cesse. Comment la rendre persistante ?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [La mémoire persiste-t-elle indéfiniment ? Quelles sont les limités ?](#does-memory-persist-forever-what-are-the-limits)
  - [La recherche sémantique de mémoire nécessite-t-elle une clé API OpenAI ?](#does-semantic-memory-search-require-an-openai-api-key)
- [Où les données sont stockées sur le disque](#where-things-live-on-disk)
  - [Toutes les données utilisées avec OpenClaw sont-elles sauvegardées localement ?](#is-all-data-used-with-openclaw-saved-locally)
  - [Où OpenClaw stocke-t-il ses données ?](#where-does-openclaw-store-its-data)
  - [Où doivent se trouver AGENTS.md / SOUL.md / USER.md / MEMORY.md ?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [Quelle est la stratégie de sauvegarde recommandée ?](#whats-the-recommended-backup-strategy)
  - [Comment désinstaller complètement OpenClaw ?](#how-do-i-completely-uninstall-openclaw)
  - [Les agents peuvent-ils travailler en dehors de l'espace de travail ?](#can-agents-work-outside-the-workspace)
  - [Je suis en mode distant, où se trouve le magasin de sessions ?](#im-in-remote-mode-where-is-the-session-store)
- [Bases de la configuration](#config-basics)
  - [Quel est le format de la configuration ? Où est-elle ?](#what-format-is-the-config-where-is-it)
  - [J'ai défini `gateway.bind: "lan"` (ou `"tailnet"`) et maintenant rien n'écoute / l'interface dit unauthorized](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Pourquoi ai-je besoin d'un jeton sur localhost maintenant ?](#why-do-i-need-a-token-on-localhost-now)
  - [Dois-je redémarrer après avoir modifié la configuration ?](#do-i-have-to-restart-after-changing-config)
  - [Comment activer la recherche web (et le web fetch) ?](#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply a effacé ma configuration. Comment récupérer et éviter ça ?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [Comment exécuter un Gateway central avec des workers spécialisés sur différents appareils ?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [Le navigateur OpenClaw peut-il fonctionner en headless ?](#can-the-openclaw-browser-run-headless)
  - [Comment utiliser Brave pour le contrôle du navigateur ?](#how-do-i-use-brave-for-browser-control)
- [Gateways distants et nœuds](#remote-gateways-and-nodes)
  - [Comment les commandes se propagent-elles entre Telegram, le Gateway et les nœuds ?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [Comment mon agent peut-il accéder à mon ordinateur si le Gateway est hébergé à distance ?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale est connecté mais je ne reçois pas de réponses. Que faire ?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [Deux instances OpenClaw peuvent-elles communiquer entre elles (local + VPS) ?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [Ai-je besoin de VPS séparés pour plusieurs agents ?](#do-i-need-separate-vpses-for-multiple-agents)
  - [Y a-t-il un avantage à utiliser un nœud sur mon laptop personnel plutôt que SSH depuis un VPS ?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Les nœuds exécutent-ils un service Gateway ?](#do-nodes-run-a-gateway-service)
  - [Existe-t-il une API / RPC pour appliquer la configuration ?](#is-there-an-api-rpc-way-to-apply-config)
  - [Quelle est la configuration minimale « saine » pour une première installation ?](#whats-a-minimal-sane-config-for-a-first-install)
  - [Comment configurer Tailscale sur un VPS et se connecter depuis mon Mac ?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [Comment connecter un nœud Mac à un Gateway distant (Tailscale Serve) ?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [Dois-je installer sur un second laptop ou simplement ajouter un nœud ?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [Variables d'environnement et chargement .env](#env-vars-and-env-loading)
  - [Comment OpenClaw charge-t-il les variables d'environnement ?](#how-does-openclaw-load-environment-variables)
  - [« J'ai démarré le Gateway via le service et mes variables d'environnement ont disparu. » Que faire ?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [J'ai défini `COPILOT_GITHUB_TOKEN`, mais models status affiche « Shell env: off. » Pourquoi ?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [Sessions et conversations multiples](#sessions-and-multiple-chats)
  - [Comment démarrer une nouvelle conversation ?](#how-do-i-start-a-fresh-conversation)
  - [Les sessions se réinitialisent-elles automatiquement si je n'envoie jamais `/new` ?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [Existe-t-il un moyen de créer une équipe d'instances OpenClaw : un CEO et plusieurs agents ?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [Pourquoi le contexte a-t-il été tronqué en pleine tâche ? Comment l'empêcher ?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [Comment réinitialiser complètement OpenClaw tout en le gardant installé ?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - [J'obtiens des erreurs « context too large », comment réinitialiser ou compacter ?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - [Pourquoi est-ce que je vois « LLM request rejected: messages.content.tool_use.input field required » ?](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
  - [Pourquoi est-ce que je reçois des messages heartbeat toutes les 30 minutes ?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [Dois-je ajouter un « compte bot » à un groupe WhatsApp ?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [Comment obtenir le JID d'un groupe WhatsApp ?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [Pourquoi OpenClaw ne répond-il pas dans un groupe ?](#why-doesnt-openclaw-reply-in-a-group)
  - [Les groupes/threads partagent-ils le contexte avec les DM ?](#do-groupsthreads-share-context-with-dms)
  - [Combien d'espaces de travail et d'agents puis-je créer ?](#how-many-workspaces-and-agents-can-i-create)
  - [Puis-je exécuter plusieurs bots ou chats en même temps (Slack) et comment configurer cela ?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [Modèles : défauts, sélection, alias, changement](#models-defaults-sélection-aliases-switching)
  - [Quel est le « modèle par défaut » ?](#what-is-the-default-model)
  - [Quel modèle recommandez-vous ?](#what-model-do-you-recommend)
  - [Comment changer de modèle sans écraser ma configuration ?](#how-do-i-switch-models-without-wiping-my-config)
  - [Puis-je utiliser des modèles auto-hébergés (llama.cpp, vLLM, Ollama) ?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [Quels modèles utilisent OpenClaw, Flawd et Krill ?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [Comment changer de modèle à la volée (sans redémarrer) ?](#how-do-i-switch-models-on-the-fly-without-restarting)
  - [Puis-je utiliser GPT 5.2 pour les tâches quotidiennes et Codex 5.3 pour le codage ?](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - [Pourquoi est-ce que je vois « Model ... is not allowed » puis aucune réponse ?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - [Pourquoi est-ce que je vois « Unknown model: minimax/MiniMax-M2.1 » ?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [Puis-je utiliser MiniMax comme défaut et OpenAI pour les tâches complexes ?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [Les raccourcis opus / sonnet / gpt sont-ils intégrés ?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [Comment définir/remplacer les raccourcis de modèle (alias) ?](#how-do-i-defineoverride-model-shortcuts-aliases)
  - [Comment ajouter des modèles d'autres fournisseurs comme OpenRouter ou Z.AI ?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [Basculement de modèle et « All models failed »](#model-failover-and-all-models-failed)
  - [Comment fonctionne le basculement ?](#how-does-failover-work)
  - [Que signifie cette erreur ?](#what-does-this-error-mean)
  - [Checklist de correction pour `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [Pourquoi a-t-il aussi essayé Google Gemini et échoué ?](#why-did-it-also-try-google-gemini-and-fail)
- [Profils d'authentification : ce qu'ils sont et comment les gérer](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [Qu'est-ce qu'un profil d'authentification ?](#what-is-an-auth-profile)
  - [Quels sont les IDs de profil typiques ?](#what-are-typical-profile-ids)
  - [Puis-je contrôler quel profil d'authentification est essayé en premier ?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth vs clé API : quelle est la différence ?](#oauth-vs-api-key-whats-the-difference)
- [Gateway : ports, « already running » et mode distant](#gateway-ports-already-running-and-remote-mode)
  - [Quel port utilisé le Gateway ?](#what-port-does-the-gateway-use)
  - [Pourquoi `openclaw gateway status` dit `Runtime: running` mais `RPC probe: failed` ?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [Pourquoi `openclaw gateway status` affiche `Config (cli)` et `Config (service)` différents ?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - [Que signifie « another gateway instance is already listening » ?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [Comment exécuter OpenClaw en mode distant (le client se connecte à un Gateway ailleurs) ?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [L'interface de contrôle dit « unauthorized » (ou se reconnecte sans cesse). Que faire ?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [J'ai défini `gateway.bind: "tailnet"` mais ça ne peut pas se lier / rien n'écoute](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Puis-je exécuter plusieurs Gateways sur le même hôte ?](#can-i-run-multiple-gateways-on-the-same-host)
  - [Que signifie « invalid handshake » / code 1008 ?](#what-does-invalid-handshake-code-1008-mean)
- [Journalisation et débogage](#logging-and-debugging)
  - [Où sont les journaux ?](#where-are-logs)
  - [Comment démarrer/arrêter/redémarrer le service Gateway ?](#how-do-i-startstoprestart-the-gateway-service)
  - [J'ai fermé mon terminal sous Windows, comment redémarrer OpenClaw ?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [Le Gateway est actif mais les réponses n'arrivent jamais. Que vérifier ?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - [« Disconnected from gateway: no reason », que faire ?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands échoue avec des erreurs réseau. Que vérifier ?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [Le TUI n'affiche rien. Que vérifier ?](#tui-shows-no-output-what-should-i-check)
  - [Comment arrêter complètement puis démarrer le Gateway ?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5 : `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [Quel est le moyen le plus rapide d'obtenir plus de détails quand quelque chose échoue ?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Médias et pièces jointes](#média-and-attachments)
  - [Ma Skill a généré une image/PDF, mais rien n'a été envoyé](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [Sécurité et contrôle d'accès](#security-and-access-control)
  - [Est-il sûr d'exposer OpenClaw aux DM entrants ?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [L'injection de prompt est-elle uniquement un problème pour les bots publics ?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Mon bot devrait-il avoir son propre email, compte GitHub ou numéro de téléphone ?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [Puis-je lui donner l'autonomie sur mes messages texte et est-ce sûr ?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [Puis-je utiliser des modèles moins chers pour les tâches d'assistant personnel ?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  - [J'ai exécuté `/start` dans Telegram mais je n'ai pas reçu de code d'appariement](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp : va-t-il envoyer des messages à mes contacts ? Comment fonctionne l'appariement ?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [Commandes de chat, annulation de tâches et « ça ne s'arrête pas »](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [Comment empêcher les messages système internes d'apparaître dans le chat ?](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [Comment arrêter/annuler une tâche en cours ?](#how-do-i-stopcancel-a-running-task)
  - [Comment envoyer un message Discord depuis Telegram ? (« Cross-context messaging denied »)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [Pourquoi on dirait que le bot « ignoré » les messages rapides ?](#why-does-it-feel-like-the-bot-ignorés-rapidfire-messages)

## Les 60 premières secondes si quelque chose est cassé

1. **Statut rapide (première vérification)**

   ```bash
   openclaw status
   ```

   Résumé local rapide : OS + mise à jour, joignabilité du Gateway/service, agents/sessions, configuration du fournisseur + problèmes à l'exécution (quand le Gateway est joignable).

2. **Rapport copiable (sûr à partager)**

   ```bash
   openclaw status --all
   ```

   Diagnostic en lecture seule avec queue de journal (jetons masqués).

3. **État du démon + du port**

   ```bash
   openclaw gateway status
   ```

   Affiche l'état de l'exécution du superviseur vs la joignabilité RPC, l'URL cible de la sonde, et quelle configuration le service a probablement utilisée.

4. **Sondes approfondies**

   ```bash
   openclaw status --deep
   ```

   Exécute des vérifications de santé du Gateway + sondes de fournisseurs (nécessite un Gateway joignable). Voir [Health](/gateway/health).

5. **Suivre le dernier journal**

   ```bash
   openclaw logs --follow
   ```

   Si le RPC est en panne, utilisez en repli :

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   Les journaux fichiers sont séparés des journaux de service ; voir [Journalisation](/logging) et [Dépannage](/gateway/troubleshooting).

6. **Exécuter le doctor (réparations)**

   ```bash
   openclaw doctor
   ```

   Répare/migre la configuration/l'état + exécute des vérifications de santé. Voir [Doctor](/gateway/doctor).

7. **Instantané du Gateway**

   ```bash
   openclaw health --json
   openclaw health --verbose   # affiche l'URL cible + le chemin de configuration en cas d'erreurs
   ```

   Demande au Gateway en cours d'exécution un instantané complet (WS uniquement). Voir [Health](/gateway/health).

## Démarrage rapide et première configuration

### Je suis bloqué, quel est le moyen le plus rapide de me débloquer

Utilisez un agent IA local qui peut **voir votre machine**. C'est bien plus efficace que de demander sur Discord, car la plupart des cas « je suis bloqué » sont des **problèmes locaux de configuration ou d'environnement** que les aidants distants ne peuvent pas inspecter.

- **Claude Code** : [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
- **OpenAI Codex** : [https://openai.com/codex/](https://openai.com/codex/)

Ces outils peuvent lire le dépôt, exécuter des commandes, inspecter les journaux et aider à corriger la configuration de votre machine (PATH, services, permissions, fichiers d'authentification). Donnez-leur le **checkout complet des sources** via l'installation hackable (git) :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Cela installe OpenClaw **depuis un checkout git**, pour que l'agent puisse lire le code + la documentation et raisonner sur la version exacte que vous exécutez. Vous pouvez toujours revenir à stable plus tard en relançant l'installeur sans `--install-method git`.

Astuce : demandez à l'agent de **planifier et superviser** la correction (étape par étape), puis n'exécutez que les commandes nécessaires. Cela garde les modifications petites et plus faciles à auditer.

Si vous découvrez un vrai bug ou une correction, veuillez créer un ticket GitHub ou envoyer une PR :
[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues)
[https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls)

Commencez par ces commandes (partagez les sorties quand vous demandez de l'aide) :

```bash
openclaw status
openclaw models status
openclaw doctor
```

Ce qu'elles font :

- `openclaw status` : instantané rapide de la santé du Gateway/agent + configuration de base.
- `openclaw models status` : vérifie l'authentification du fournisseur + la disponibilité des modèles.
- `openclaw doctor` : valide et répare les problèmes courants de configuration/état.

Autres vérifications CLI utiles : `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

Boucle de débogage rapide : [Les 60 premières secondes si quelque chose est cassé](#les-60-premières-secondes-si-quelque-chose-est-cassé).
Documentation d'installation : [Install](/install), [Flags de l'installeur](/install/installer), [Mise à jour](/install/updating).

### Quelle est la méthode recommandée pour installer et configurer OpenClaw

Le dépôt recommandé d'exécuter depuis les sources et d'utiliser l'assistant de configuration initiale :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

L'assistant peut aussi construire les assets UI automatiquement. Après la configuration initiale, vous exécutez typiquement le Gateway sur le port **18789**.

Depuis les sources (contributeurs/dev) :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # installe automatiquement les dépendances UI à la première exécution
openclaw onboard
```

Si vous n'avez pas encore d'installation globale, exécutez via `pnpm openclaw onboard`.

### Comment ouvrir le tableau de bord après la configuration initiale

L'assistant ouvre votre navigateur avec une URL de tableau de bord propre (sans jeton) juste après la configuration initiale et affiche aussi le lien dans le résumé. Gardez cet onglet ouvert ; s'il ne s'est pas lancé, copiez/collez l'URL affichée sur la même machine.

### Comment authentifier le tableau de bord (jeton) en localhost vs distant

**Localhost (même machine) :**

- Ouvrez `http://127.0.0.1:18789/`.
- S'il demande l'authentification, collez le jeton de `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) dans les paramètres de l'interface de contrôle.
- Récupérez-le depuis l'hôte du Gateway : `openclaw config get gateway.auth.token` (ou générez-en un : `openclaw doctor --generate-gateway-token`).

**Hors localhost :**

- **Tailscale Serve** (recommandé) : gardez la liaison loopback, exécutez `openclaw gateway --tailscale serve`, ouvrez `https://<magicdns>/`. Si `gateway.auth.allowTailscale` est `true`, les en-têtes d'identité satisfont l'authentification de l'interface de contrôle/WebSocket (pas de jeton, suppose un hôte Gateway de confiance) ; les APIs HTTP nécessitent toujours un jeton/mot de passe.
- **Liaison tailnet** : exécutez `openclaw gateway --bind tailnet --token "<token>"`, ouvrez `http://<tailscale-ip>:18789/`, collez le jeton dans les paramètres du tableau de bord.
- **Tunnel SSH** : `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/` et collez le jeton dans les paramètres de l'interface de contrôle.

Voir [Dashboard](/web/dashboard) et [Surfaces web](/web) pour les modes de liaison et les détails d'authentification.

### Quel runtime me faut-il

Node **>= 22** est requis. `pnpm` est recommandé. Bun n'est **pas recommandé** pour le Gateway.

### Est-ce que ça tourne sur Raspberry Pi

Oui. Le Gateway est léger : la documentation indique **512 Mo-1 Go de RAM**, **1 cœur** et environ **500 Mo** de disque comme suffisants pour un usage personnel, et note qu'un **Raspberry Pi 4 peut le faire tourner**.

Si vous voulez plus de marge (journaux, médias, autres services), **2 Go sont recommandés**, mais ce n'est pas un minimum strict.

Astuce : un petit Pi/VPS peut héberger le Gateway, et vous pouvez appairer des **nœuds** sur votre laptop/téléphone pour l'écran/caméra/canvas local ou l'exécution de commandes. Voir [Nodes](/nodes).

### Des conseils pour les installations Raspberry Pi

Version courte : ça fonctionne, mais attendez-vous à des aspérités.

- Utilisez un OS **64 bits** et gardez Node >= 22.
- Préférez l'**installation hackable (git)** pour voir les journaux et mettre à jour rapidement.
- Commencez sans canaux/Skills, puis ajoutez-les un par un.
- Si vous rencontrez des problèmes bizarres de binaires, c'est généralement un problème de **compatibilité ARM**.

Documentation : [Linux](/platforms/linux), [Install](/install).

### C'est bloqué sur « wake up my friend » / la configuration initiale ne démarre pas. Que faire

Cet écran dépend de la joignabilité et de l'authentification du Gateway. Le TUI envoie aussi « Wake up, my friend! » automatiquement au premier démarrage. Si vous voyez cette ligne avec **aucune réponse** et que les jetons restent à 0, l'agent ne s'est jamais exécuté.

1. Redémarrez le Gateway :

```bash
openclaw gateway restart
```

2. Vérifiez le statut + l'authentification :

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. Si c'est toujours bloqué, exécutez :

```bash
openclaw doctor
```

Si le Gateway est distant, assurez-vous que la connexion tunnel/Tailscale est activé et que l'interface pointe vers le bon Gateway. Voir [Accès distant](/gateway/remote).

### Puis-je migrer ma configuration vers une nouvelle machine (Mac mini) sans refaire la configuration initiale

Oui. Copiez le **répertoire d'état** et l'**espace de travail**, puis exécutez Doctor une fois. Cela garde votre bot « exactement le même » (mémoire, historique de session, authentification et état du canal) tant que vous copiez **les deux** emplacements :

1. Installez OpenClaw sur la nouvelle machine.
2. Copiez `$OPENCLAW_STATE_DIR` (par défaut : `~/.openclaw`) depuis l'ancienne machine.
3. Copiez votre espace de travail (par défaut : `~/.openclaw/workspace`).
4. Exécutez `openclaw doctor` et redémarrez le service Gateway.

Cela préserve la configuration, les profils d'authentification, les identifiants WhatsApp, les sessions et la mémoire. Si vous êtes en mode distant, rappelez-vous que l'hôte du Gateway possède le magasin de sessions et l'espace de travail.

**Important :** si vous ne faites que commit/push de votre espace de travail sur GitHub, vous sauvegardez la **mémoire + les fichiers de bootstrap**, mais **pas** l'historique de session ou l'authentification. Ceux-ci se trouvent sous `~/.openclaw/` (par exemple `~/.openclaw/agents/<agentId>/sessions/`).

Documentation connexe : [Migration](/install/migrating), [Où les données sont stockées sur le disque](/help/faq#where-does-openclaw-store-its-data),
[Espace de travail de l'agent](/concepts/agent-workspace), [Doctor](/gateway/doctor),
[Mode distant](/gateway/remote).

### Où voir les nouveautés de la dernière version

Consultez le changelog sur GitHub :
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

Les entrées les plus récentes sont en haut. Si la section du haut est marquée **Unreleased**, la section datée suivante est la dernière version livrée. Les entrées sont regroupées par **Highlights**, **Changes** et **Fixes** (plus des sections docs/autres si nécessaire).

### Je ne peux pas accéder à docs.openclaw.ai (erreur SSL). Que faire

Certaines connexions Comcast/Xfinity bloquent incorrectement `docs.openclaw.ai` via Xfinity Advanced Security. Désactivez-le ou ajoutez `docs.openclaw.ai` à la liste blanche, puis réessayez. Plus de détails : [Dépannage](/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
Aidez-nous à le débloquer en signalant ici : [https://spa.xfinity.com/check_url_status](https://spa.xfinity.com/check_url_status).

Si vous ne pouvez toujours pas accéder au site, la documentation est miroir sur GitHub :
[https://github.com/openclaw/openclaw/tree/main/docs](https://github.com/openclaw/openclaw/tree/main/docs)

### Quelle est la différence entre stable et beta

**Stable** et **beta** sont des **dist-tags npm**, pas des lignes de code séparées :

- `latest` = stable
- `beta` = build anticipé pour les tests

Nous publions les builds sur **beta**, les testons, et quand un build est solide nous **promouvons cette même version sur `latest`**. C'est pourquoi beta et stable peuvent pointer sur la **même version**.

Voir les changements :
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

### Comment installer la version beta et quelle est la différence entre beta et dev

**Beta** est le dist-tag npm `beta` (peut correspondre à `latest`).
**Dev** est le head mouvant de `main` (git) ; quand publié, il utilise le dist-tag npm `dev`.

Commandes en une ligne (macOS/Linux) :

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Installeur Windows (PowerShell) :
[https://openclaw.ai/install.ps1](https://openclaw.ai/install.ps1)

Plus de détails : [Canaux de développement](/install/development-channels) et [Flags de l'installeur](/install/installer).

### Combien de temps prennent l'installation et la configuration initiale

Guide approximatif :

- **Installation :** 2-5 minutes
- **Configuration initiale :** 5-15 minutes selon le nombre de canaux/modèles que vous configurez

Si c'est bloqué, utilisez [Installeur bloqué](/help/faq#installer-stuck-how-do-i-get-more-feedback)
et la boucle de débogage rapide dans [Je suis bloqué](/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

### Comment essayer les dernières modifications

Deux options :

1. **Canal dev (checkout git) :**

```bash
openclaw update --channel dev
```

Cela bascule sur la branche `main` et met à jour depuis les sources.

2. **Installation hackable (depuis le site de l'installeur) :**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Cela vous donne un dépôt local que vous pouvez modifier, puis mettre à jour via git.

Si vous préférez un clone propre manuellement, utilisez :

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

Documentation : [Update](/cli/update), [Canaux de développement](/install/development-channels),
[Install](/install).

### L'installeur est bloqué. Comment obtenir plus de retour

Relancez l'installeur avec une **sortie verbeuse** :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

Installation beta avec verbose :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

Pour une installation hackable (git) :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --verbose
```

Équivalent Windows (PowerShell) :

```powershell
# install.ps1 n'a pas encore de flag -Verbose dédié.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

Plus d'options : [Flags de l'installeur](/install/installer).

### L'installation Windows dit git non trouvé ou openclaw non reconnu

Deux problèmes Windows courants :

**1) npm error spawn git / git non trouvé**

- Installez **Git for Windows** et assurez-vous que `git` est dans votre PATH.
- Fermez et rouvrez PowerShell, puis relancez l'installeur.

**2) openclaw n'est pas reconnu après l'installation**

- Votre dossier bin global npm n'est pas dans le PATH.
- Vérifiez le chemin :

  ```powershell
  npm config get prefix
  ```

- Assurez-vous que `<prefix>\\bin` est dans le PATH (sur la plupart des systèmes c'est `%AppData%\\npm`).
- Fermez et rouvrez PowerShell après la mise à jour du PATH.

Si vous voulez la configuration Windows la plus fluide, utilisez **WSL2** au lieu de Windows natif.
Documentation : [Windows](/platforms/windows).

### La documentation n'a pas répondu à ma question. Comment obtenir une meilleure réponse

Utilisez l'**installation hackable (git)** pour avoir les sources et la documentation complètes en local, puis demandez à votre bot (ou Claude/Codex) _depuis ce dossier_ pour qu'il puisse lire le dépôt et répondre précisément.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Plus de détails : [Install](/install) et [Flags de l'installeur](/install/installer).

### Comment installer OpenClaw sur Linux

Réponse courte : suivez le guide Linux, puis exécutez l'assistant de configuration initiale.

- Chemin rapide Linux + installation du service : [Linux](/platforms/linux).
- Guide complet : [Premiers pas](/start/getting-started).
- Installeur + mises à jour : [Installation & mises à jour](/install/updating).

### Comment installer OpenClaw sur un VPS

N'importe quel VPS Linux fonctionne. Installez sur le serveur, puis utilisez SSH/Tailscale pour atteindre le Gateway.

Guides : [exe.dev](/install/exe-dev), [Hetzner](/install/hetzner), [Fly.io](/install/fly).
Accès distant : [Gateway remote](/gateway/remote).

### Où sont les guides d'installation cloud/VPS

Nous maintenons un **hub d'hébergement** avec les fournisseurs courants. Choisissez-en un et suivez le guide :

- [Hébergement VPS](/vps) (tous les fournisseurs en un seul endroit)
- [Fly.io](/install/fly)
- [Hetzner](/install/hetzner)
- [exe.dev](/install/exe-dev)

Comment ça fonctionne dans le cloud : le **Gateway tourne sur le serveur**, et vous y accédez depuis votre laptop/téléphone via l'interface de contrôle (ou Tailscale/SSH). Votre état + espace de travail vivent sur le serveur, donc traitez l'hôte comme la source de vérité et sauvegardez-le.

Vous pouvez appairer des **nœuds** (Mac/iOS/Android/headless) à ce Gateway cloud pour accéder à l'écran/caméra/canvas local ou exécuter des commandes sur votre laptop tout en gardant le Gateway dans le cloud.

Hub : [Plateformes](/platforms). Accès distant : [Gateway remote](/gateway/remote).
Nœuds : [Nodes](/nodes), [CLI Nodes](/cli/nodes).

### Puis-je demander à OpenClaw de se mettre à jour

Réponse courte : **possible, non recommandé**. Le flux de mise à jour peut redémarrer le Gateway (ce qui coupe la session active), peut nécessiter un checkout git propre et peut demander une confirmation. Plus sûr : exécutez les mises à jour depuis un shell en tant qu'opérateur.

Utilisez le CLI :

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

Si vous devez automatiser depuis un agent :

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

Documentation : [Update](/cli/update), [Mise à jour](/install/updating).

### Que fait réellement l'assistant de configuration initiale

`openclaw onboard` est le chemin de configuration recommandé. En **mode local** il vous guide à travers :

- **Configuration modèle/authentification** (**setup-token** Anthropic recommandé pour les abonnements Claude, OAuth OpenAI Codex supporté, clés API optionnelles, modèles locaux LM Studio supportés)
- Emplacement de l'**espace de travail** + fichiers de bootstrap
- **Paramètres du Gateway** (liaison/port/auth/tailscale)
- **Fournisseurs** (WhatsApp, Telegram, Discord, Mattermost (plugin), Signal, iMessage)
- **Installation du démon** (LaunchAgent sur macOS ; unité utilisateur systemd sur Linux/WSL2)
- **Vérifications de santé** et sélection de **Skills**

Il avertit aussi si votre modèle configuré est inconnu ou si l'authentification est manquante.

### Ai-je besoin d'un abonnement Claude ou OpenAI pour utiliser ceci

Non. Vous pouvez exécuter OpenClaw avec des **clés API** (Anthropic/OpenAI/autres) ou avec des **modèles locaux uniquement** pour que vos données restent sur votre appareil. Les abonnements (Claude Pro/Max ou OpenAI Codex) sont des moyens optionnels d'authentifier ces fournisseurs.

Documentation : [Anthropic](/providers/anthropic), [OpenAI](/providers/openai),
[Modèles locaux](/gateway/local-models), [Modèles](/concepts/models).

### Puis-je utiliser l'abonnement Claude Max sans clé API

Oui. Vous pouvez vous authentifier avec un **setup-token** au lieu d'une clé API. C'est le chemin par abonnement.

Les abonnements Claude Pro/Max **n'incluent pas de clé API**, donc c'est l'approche correcte pour les comptes par abonnement. Important : vous devez vérifier auprès d'Anthropic que cette utilisation est autorisée selon leur politique d'abonnement et leurs conditions. Si vous voulez le chemin le plus explicite et supporte, utilisez une clé API Anthropic.

### Comment fonctionne l'authentification setup-token Anthropic

`claude setup-token` génère une **chaîne de jeton** via le CLI Claude Code (elle n'est pas disponible dans la console web). Vous pouvez l'exécuter sur **n'importe quelle machine**. Choisissez **Anthropic token (paste setup-token)** dans l'assistant ou collez-le avec `openclaw models auth paste-token --provider anthropic`. Le jeton est stocké comme profil d'authentification pour le fournisseur **anthropic** et utilise comme une clé API (pas de rafraîchissement automatique). Plus de détails : [OAuth](/concepts/oauth).

### Où trouver un setup-token Anthropic

Ce n'est **pas** dans la Console Anthropic. Le setup-token est généré par le **CLI Claude Code** sur **n'importe quelle machine** :

```bash
claude setup-token
```

Copiez le jeton qu'il affiche, puis choisissez **Anthropic token (paste setup-token)** dans l'assistant. Si vous voulez l'exécuter sur l'hôte du Gateway, utilisez `openclaw models auth setup-token --provider anthropic`. Si vous avez exécuté `claude setup-token` ailleurs, collez-le sur l'hôte du Gateway avec `openclaw models auth paste-token --provider anthropic`. Voir [Anthropic](/providers/anthropic).

### Supportez-vous l'authentification par abonnement Claude (Claude Pro ou Max)

Oui, via **setup-token**. OpenClaw ne réutilise plus les jetons OAuth du CLI Claude Code ; utilisez un setup-token ou une clé API Anthropic. Générez le jeton n'importe où et collez-le sur l'hôte du Gateway. Voir [Anthropic](/providers/anthropic) et [OAuth](/concepts/oauth).

Note : l'accès par abonnement Claude est régi par les conditions d'Anthropic. Pour la production ou les charges de travail multi-utilisateurs, les clés API sont généralement le choix le plus sûr.

### Pourquoi est-ce que je vois HTTP 429 rate_limit_error depuis Anthropic

Cela signifie que votre **quota/limité de débit Anthropic** est épuisé pour la fenêtre en cours. Si vous utilisez un **abonnement Claude** (setup-token ou OAuth Claude Code), attendez la réinitialisation de la fenêtre ou mettez à niveau votre forfait. Si vous utilisez une **clé API Anthropic**, vérifiez la Console Anthropic pour l'utilisation/facturation et augmentez les limités si nécessaire.

Astuce : définissez un **modèle de repli** pour qu'OpenClaw puisse continuer à répondre pendant qu'un fournisseur est limité en débit. Voir [Models](/cli/models) et [OAuth](/concepts/oauth).

### AWS Bedrock est-il supporte

Oui, via le fournisseur **Amazon Bedrock (Converse)** de pi-ai avec **configuration manuelle**. Vous devez fournir les identifiants/région AWS sur l'hôte du Gateway et ajouter une entrée fournisseur Bedrock dans votre configuration de modèles. Voir [Amazon Bedrock](/providers/bedrock) et [Fournisseurs de modèles](/providers/models). Si vous préférez un flux de clé géré, un proxy compatible OpenAI devant Bedrock est toujours une option valide.

### Comment fonctionne l'authentification Codex

OpenClaw supporte **OpenAI Code (Codex)** via OAuth (connexion ChatGPT). L'assistant peut exécuter le flux OAuth et définira le modèle par défaut sur `openai-codex/gpt-5.3-codex` quand approprié. Voir [Fournisseurs de modèles](/concepts/model-providers) et [Assistant](/start/wizard).

### Supportez-vous l'authentification par abonnement OpenAI (Codex OAuth)

Oui. OpenClaw supporte entièrement **l'OAuth d'abonnement OpenAI Code (Codex)**. L'assistant de configuration initiale peut exécuter le flux OAuth pour vous.

Voir [OAuth](/concepts/oauth), [Fournisseurs de modèles](/concepts/model-providers) et [Assistant](/start/wizard).

### Comment configurer Gemini CLI OAuth

Gemini CLI utilisé un **flux d'authentification par plugin**, pas un client id ou secret dans `openclaw.json`.

Étapes :

1. Activez le plugin : `openclaw plugins enable google-gemini-cli-auth`
2. Connexion : `openclaw models auth login --provider google-gemini-cli --set-default`

Cela stocke les jetons OAuth dans les profils d'authentification sur l'hôte du Gateway. Détails : [Fournisseurs de modèles](/concepts/model-providers).

### Un modèle local convient-il pour du chat casual

Généralement non. OpenClaw a besoin d'un contexte large + d'une sécurité forte ; les petites cartes tronquent et fuient. Si vous devez le faire, exécutez le build **le plus grand** de MiniMax M2.1 que vous pouvez localement (LM Studio) et voir [/gateway/local-models](/gateway/local-models). Les modèles plus petits/quantifiés augmentent le risque d'injection de prompt : voir [Sécurité](/gateway/security).

### Comment garder le trafic de modèles hébergés dans une région spécifique

Choisissez des points d'accès avec région fixe. OpenRouter expose des options hébergées aux US pour MiniMax, Kimi et GLM ; choisissez la variante hébergée aux US pour garder les données dans la région. Vous pouvez toujours lister Anthropic/OpenAI à côté en utilisant `models.mode: "merge"` pour que les replis restent disponibles tout en respectant le fournisseur régioné que vous sélectionnez.

### Dois-je acheter un Mac Mini pour installer ceci

Non. OpenClaw fonctionne sur macOS ou Linux (Windows via WSL2). Un Mac mini est optionnel : certaines personnes en achètent un comme hôte toujours allumé, mais un petit VPS, serveur domestique ou boîte de type Raspberry Pi fonctionne aussi.

Vous n'avez besoin d'un Mac **que pour les outils macOS uniquement**. Pour iMessage, utilisez [BlueBubbles](/channels/bluebubbles) (recommandé) : le serveur BlueBubbles tourne sur n'importe quel Mac, et le Gateway peut tourner sur Linux ou ailleurs. Si vous voulez d'autres outils macOS uniquement, exécutez le Gateway sur un Mac ou appairez un nœud macOS.

Documentation : [BlueBubbles](/channels/bluebubbles), [Nodes](/nodes), [Mode distant Mac](/platforms/mac/remote).

### Ai-je besoin d'un Mac mini pour le support iMessage

Vous avez besoin d'**un appareil macOS** connecté à Messages. Ce n'est **pas forcément** un Mac mini : n'importe quel Mac fonctionne. **Utilisez [BlueBubbles](/channels/bluebubbles)** (recommandé) pour iMessage : le serveur BlueBubbles tourne sur macOS, tandis que le Gateway peut tourner sur Linux ou ailleurs.

Configurations courantes :

- Exécuter le Gateway sur Linux/VPS, et le serveur BlueBubbles sur n'importe quel Mac connecté à Messages.
- Tout exécuter sur le Mac si vous voulez la configuration mono-machine la plus simple.

Documentation : [BlueBubbles](/channels/bluebubbles), [Nodes](/nodes),
[Mode distant Mac](/platforms/mac/remote).

### Si j'achète un Mac mini pour exécuter OpenClaw, puis-je le connecter à mon MacBook Pro

Oui. Le **Mac mini peut exécuter le Gateway**, et votre MacBook Pro peut se connecter comme **nœud** (appareil compagnon). Les nœuds n'exécutent pas le Gateway : ils fournissent des capacités supplémentaires comme écran/caméra/canvas et `system.run` sur cet appareil.

Configuration courante :

- Gateway sur le Mac mini (toujours allumé).
- Le MacBook Pro exécute l'app macOS ou un hôte de nœud et s'apparié au Gateway.
- Utilisez `openclaw nodes status` / `openclaw nodes list` pour le voir.

Documentation : [Nodes](/nodes), [CLI Nodes](/cli/nodes).

### Puis-je utiliser Bun

Bun n'est **pas recommandé**. Nous constatons des bugs d'exécution, notamment avec WhatsApp et Telegram.
Utilisez **Node** pour des gateways stables.

Si vous voulez quand même expérimenter avec Bun, faites-le sur un gateway hors production
sans WhatsApp/Telegram.

### Dois-je exécuter le Gateway sur mon laptop ou un VPS

Réponse courte : **si vous voulez une fiabilité 24/7, utilisez un VPS**. Si vous voulez le minimum de friction et que les mises en veille/redémarrages ne vous dérangent pas, exécutez-le localement.

**Laptop (Gateway local)**

- **Avantages :** pas de coût serveur, accès direct aux fichiers locaux, fenêtre de navigateur visible.
- **Inconvénients :** mise en veille/coupures réseau = déconnexions, les mises à jour/redémarrages de l'OS interrompent, doit rester éveillé.

**VPS / cloud**

- **Avantages :** toujours allumé, réseau stable, pas de problèmes de mise en veille, plus facile à maintenir en fonctionnement.
- **Inconvénients :** souvent headless (utilisez les captures d'écran), accès fichiers distant uniquement, vous devez SSH pour les mises à jour.

**Note spécifique à OpenClaw :** WhatsApp/Telegram/Slack/Mattermost (plugin)/Discord fonctionnent tous bien depuis un VPS. Le seul vrai compromis est le **navigateur headless** vs une fenêtre visible. Voir [Navigateur](/tools/browser).

**Défaut recommandé :** VPS si vous avez eu des déconnexions du gateway auparavant. Local est idéal quand vous utilisez activement le Mac et que vous voulez l'accès aux fichiers locaux ou l'automatisation UI avec un navigateur visible.

### À quel point est-il important d'exécuter OpenClaw sur une machine dédiée

Pas obligatoire, mais **recommandé pour la fiabilité et l'isolation**.

- **Hôte dédié (VPS/Mac mini/Pi) :** toujours allumé, moins d'interruptions veille/redémarrage, permissions plus propres, plus facile à maintenir en fonctionnement.
- **Laptop/desktop partagé :** tout à fait correct pour les tests et l'utilisation active, mais attendez-vous à des pauses quand la machine se met en veille ou se met à jour.

Si vous voulez le meilleur des deux mondes, gardez le Gateway sur un hôte dédié et appairez votre laptop comme **nœud** pour les outils locaux écran/caméra/exec. Voir [Nœuds](/nodes).
Pour les conseils de sécurité, lisez [Sécurité](/gateway/security).

### Quels sont les prérequis VPS minimaux et l'OS recommandé

OpenClaw est léger. Pour un Gateway basique + un canal de chat :

- **Minimum absolu :** 1 vCPU, 1 Go de RAM, ~500 Mo de disque.
- **Recommandé :** 1-2 vCPU, 2 Go de RAM ou plus pour la marge (logs, médias, canaux multiples). Les outils de nœud et l'automatisation du navigateur peuvent être gourmands en ressources.

OS : utilisez **Ubuntu LTS** (ou tout Debian/Ubuntu moderne). Le chemin d'installation Linux est le mieux testé là.

Documentation : [Linux](/platforms/linux), [Hébergement VPS](/vps).

### Puis-je exécuter OpenClaw dans une VM et quels sont les prérequis

Oui. Traitez une VM comme un VPS : elle doit être toujours allumée, joignable, et avoir assez de RAM pour le Gateway et les canaux que vous activez.

Recommandations de base :

- **Minimum absolu :** 1 vCPU, 1 Go de RAM.
- **Recommandé :** 2 Go de RAM ou plus si vous exécutez plusieurs canaux, l'automatisation du navigateur ou des outils médias.
- **OS :** Ubuntu LTS ou un autre Debian/Ubuntu moderne.

Si vous êtes sous Windows, **WSL2 est la configuration de type VM la plus simple** et offre la meilleure compatibilité d'outillage. Voir [Windows](/platforms/windows), [Hébergement VPS](/vps).
Si vous exécutez macOS dans une VM, voir [macOS VM](/install/macos-vm).

## Qu'est-ce qu'OpenClaw ?

### Qu'est-ce qu'OpenClaw, en un paragraphe

OpenClaw est un assistant IA personnel que vous exécutez sur vos propres appareils. Il répond sur les surfaces de messagerie que vous utilisez déjà (WhatsApp, Telegram, Slack, Mattermost (plugin), Discord, Google Chat, Signal, iMessage, WebChat) et peut aussi faire de la voix + un Canvas en direct sur les plateformes supportées. Le **Gateway** est le plan de contrôle toujours actif ; l'assistant est le produit.

### Quelle est la proposition de valeur

OpenClaw n'est pas « juste un wrapper Claude ». C'est un **plan de contrôle local-first** qui vous permet d'exécuter un assistant capable sur **votre propre matériel**, joignable depuis les apps de chat que vous utilisez déjà, avec des sessions avec état, de la mémoire et des outils, sans confier le contrôle de vos workflows à un SaaS hébergé.

Points forts :

- **Vos appareils, vos données :** exécutez le Gateway où vous voulez (Mac, Linux, VPS) et gardez l'espace de travail + l'historique de sessions en local.
- **De vrais canaux, pas un bac à sable web :** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc., plus la voix mobile et le Canvas sur les plateformes supportées.
- **Agnostique en modèles :** utilisez Anthropic, OpenAI, MiniMax, OpenRouter, etc., avec routage par agent et basculement.
- **Option tout-en-local :** exécutez des modèles locaux pour que **toutes les données restent sur votre appareil** si vous le souhaitez.
- **Routage multi-agent :** des agents séparés par canal, compte ou tâche, chacun avec son propre espace de travail et ses défauts.
- **Open source et hackable :** inspectez, étendez et auto-hébergez sans verrouillage fournisseur.

Documentation : [Gateway](/gateway), [Canaux](/channels), [Multi-agent](/concepts/multi-agent),
[Mémoire](/concepts/memory).

### Je viens de l'installer, que dois-je faire en premier

Bons premiers projets :

- Construire un site web (WordPress, Shopify, ou un simple site statique).
- Prototyper une application mobile (plan, écrans, plan d'API).
- Organiser des fichiers et dossiers (nettoyage, nommage, étiquetage).
- Connecter Gmail et automatiser les résumés ou les relances.

Il peut gérer de grandes tâches, mais il fonctionne mieux quand vous les découpez en phases et utilisez des sous-agents pour le travail en parallèle.

### Quels sont les cinq cas d'usage quotidiens principaux d'OpenClaw

Les gains au quotidien ressemblent généralement à :

- **Briefings personnels :** résumés de la boîte de réception, du calendrier et des actualités qui vous intéressent.
- **Recherche et rédaction :** recherche rapide, résumés et premiers jets pour les emails ou les documents.
- **Rappels et relances :** nudges et checklists pilotés par cron ou heartbeat.
- **Automatisation du navigateur :** remplir des formulaires, collecter des données et répéter des tâches web.
- **Coordination multi-appareils :** envoyez une tâche depuis votre téléphone, laissez le Gateway l'exécuter sur un serveur, et recevez le résultat dans le chat.

### OpenClaw peut-il aider pour la génération de leads, la prospection, les pubs et les blogs pour un SaaS

Oui pour la **recherche, la qualification et la rédaction**. Il peut scanner des sites, construire des listes restreintes, résumer des prospects et rédiger des brouillons de prospection ou de textes publicitaires.

Pour les **campagnes de prospection ou de publicité**, gardez un humain dans la boucle. Évitez le spam, respectez les lois locales et les politiques des plateformes, et relisez tout avant envoi. Le schéma le plus sûr est de laisser OpenClaw rédiger et vous approuvez.

Documentation : [Sécurité](/gateway/security).

### Quels sont les avantages par rapport à Claude Code pour le développement web

OpenClaw est un **assistant personnel** et une couche de coordination, pas un remplacement d'IDE. Utilisez Claude Code ou Codex pour la boucle de codage direct la plus rapide dans un dépôt. Utilisez OpenClaw quand vous voulez de la mémoire durable, un accès multi-appareils et une orchestration d'outils.

Avantages :

- **Mémoire persistante + espace de travail** entre les sessions
- **Accès multi-plateforme** (WhatsApp, Telegram, TUI, WebChat)
- **Orchestration d'outils** (navigateur, fichiers, planification, hooks)
- **Gateway toujours actif** (exécutez sur un VPS, interagissez de n'importe où)
- **Nœuds** pour navigateur/écran/caméra/exec locaux

Vitrine : [https://openclaw.ai/showcase](https://openclaw.ai/showcase)

## Skills et automatisation

### Comment personnaliser les Skills sans salir le dépôt

Utilisez des surcharges gérées au lieu de modifier la copie du dépôt. Placez vos modifications dans `~/.openclaw/skills/<nom>/SKILL.md` (ou ajoutez un dossier via `skills.load.extraDirs` dans `~/.openclaw/openclaw.json`). L'ordre de priorité est `<workspace>/skills` > `~/.openclaw/skills` > inclus, donc les surcharges gérées l'emportent sans toucher à git. Seules les modifications dignes d'être intégrées en amont doivent rester dans le dépôt et être soumises comme PR.

### Puis-je charger des Skills depuis un dossier personnalisé

Oui. Ajoutez des répertoires supplémentaires via `skills.load.extraDirs` dans `~/.openclaw/openclaw.json` (priorité la plus basse). L'ordre de priorité par défaut reste : `<workspace>/skills` → `~/.openclaw/skills` → inclus → `skills.load.extraDirs`. `clawhub` installe dans `./skills` par défaut, qu'OpenClaw traite comme `<workspace>/skills`.

### Comment utiliser différents modèles pour différentes tâches

Aujourd'hui, les schémas supportés sont :

- **Jobs cron :** les jobs isolés peuvent définir une surcharge `model` par job.
- **Sous-agents :** routez les tâches vers des agents séparés avec différents modèles par défaut.
- **Changement à la demande :** utilisez `/model` pour changer le modèle de la session en cours à tout moment.

Voir [Jobs cron](/automation/cron-jobs), [Routage multi-agent](/concepts/multi-agent), et [Commandes slash](/tools/slash-commands).

### Le bot se fige pendant un travail lourd. Comment déléguer ça

Utilisez des **sous-agents** pour les tâches longues ou parallèles. Les sous-agents s'exécutent dans leur propre session, renvoient un résumé et gardent votre chat principal réactif.

Demandez à votre bot de « lancer un sous-agent pour cette tâche » ou utilisez `/subagents`.
Utilisez `/status` dans le chat pour voir ce que le Gateway fait en ce moment (et s'il est occupé).

Astuce tokens : les tâches longues et les sous-agents consomment tous les deux des tokens. Si le coût est un souci, définissez un modèle moins cher pour les sous-agents via `agents.defaults.subagents.model`.

Documentation : [Sous-agents](/tools/subagents).

### Comment fonctionnent les sessions de sous-agents liées aux threads sur Discord

Utilisez les liaisons de threads. Vous pouvez lier un thread Discord à un sous-agent ou une cible de session pour que les messages de suivi dans ce thread restent sur cette session liée.

Flux de base :

- Lancez avec `sessions_spawn` en utilisant `thread: true` (et optionnellement `mode: "session"` pour le suivi persistant).
- Ou liez manuellement avec `/focus <cible>`.
- Utilisez `/agents` pour inspecter l'état de la liaison.
- Utilisez `/session ttl <durée|off>` pour contrôler le détachement automatique.
- Utilisez `/unfocus` pour détacher le thread.

Configuration requise :

- Défauts globaux : `session.threadBindings.enabled`, `session.threadBindings.ttlHours`.
- Surcharges Discord : `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.ttlHours`.
- Liaison automatique au lancement : définissez `channels.discord.threadBindings.spawnSubagentSessions: true`.

Documentation : [Sous-agents](/tools/subagents), [Discord](/channels/discord), [Référence de configuration](/gateway/configuration-reference), [Commandes slash](/tools/slash-commands).

### Le cron ou les rappels ne se déclenchent pas. Que vérifier

Le cron s'exécute à l'intérieur du processus Gateway. Si le Gateway ne tourne pas en continu, les jobs planifiés ne s'exécuteront pas.

Checklist :

- Confirmez que le cron est activé (`cron.enabled`) et que `OPENCLAW_SKIP_CRON` n'est pas défini.
- Vérifiez que le Gateway tourne 24/7 (pas de mise en veille/redémarrages).
- Vérifiez les paramètres de fuseau horaire du job (`--tz` vs le fuseau de l'hôte).

Débogage :

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

Documentation : [Jobs cron](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat).

### Comment installer des Skills sur Linux

Utilisez **ClawHub** (CLI) ou déposez les Skills dans votre espace de travail. L'interface Skills de macOS n'est pas disponible sur Linux.
Parcourez les Skills sur [https://clawhub.com](https://clawhub.com).

Installez la CLI ClawHub (choisissez un gestionnaire de paquets) :

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

### OpenClaw peut-il exécuter des tâches selon un planning ou en continu en arrière-plan

Oui. Utilisez le planificateur du Gateway :

- **Jobs cron** pour les tâches planifiées ou récurrentes (persistent après les redémarrages).
- **Heartbeat** pour les vérifications périodiques de la « session principale ».
- **Jobs isolés** pour les agents autonomes qui publient des résumés ou livrent dans les chats.

Documentation : [Jobs cron](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat),
[Heartbeat](/gateway/heartbeat).

### Puis-je exécuter des Skills macOS uniquement depuis Linux ?

Pas directement. Les Skills macOS sont filtrées par `metadata.openclaw.os` plus les binaires requis, et les Skills n'apparaissent dans le prompt système que lorsqu'elles sont éligibles sur l'**hôte du Gateway**. Sous Linux, les Skills `darwin`-only (comme `apple-notes`, `apple-reminders`, `things-mac`) ne se chargeront pas sauf si vous contournez le filtrage.

Vous avez trois approches supportées :

**Option A : exécuter le Gateway sur un Mac (le plus simple).**
Exécutez le Gateway là où les binaires macOS existent, puis connectez-vous depuis Linux en [mode distant](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) ou via Tailscale. Les Skills se chargent normalement car l'hôte du Gateway est macOS.

**Option B : utiliser un nœud macOS (sans SSH).**
Exécutez le Gateway sur Linux, appairez un nœud macOS (app barre de menus), et définissez **Node Run Commands** sur « Always Ask » ou « Always Allow » sur le Mac. OpenClaw peut traiter les Skills macOS-only comme éligibles quand les binaires requis existent sur le nœud. L'agent exécute ces Skills via l'outil `nodes`. Si vous choisissez « Always Ask », approuver « Always Allow » dans l'invite ajouté cette commande à la liste autorisée.

**Option C : rediriger les binaires macOS via un proxy SSH (avancé).**
Gardez le Gateway sur Linux, mais faites en sorte que les binaires CLI requis pointent vers des wrappers SSH qui s'exécutent sur un Mac. Puis surchargez la Skill pour autoriser Linux afin qu'elle reste éligible.

1. Créez un wrapper SSH pour le binaire (exemple : `memo` pour Apple Notes) :

   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/memo "$@"
   ```

2. Placez le wrapper dans le `PATH` sur l'hôte Linux (par exemple `~/bin/memo`).
3. Surchargez les métadonnées de la Skill (workspace ou `~/.openclaw/skills`) pour autoriser Linux :

   ```markdown
   ---
   name: apple-notes
   description: Manage Apple Notes via the memo CLI on macOS.
   metadata: { "openclaw": { "os": ["darwin", "linux"], "requires": { "bins": ["memo"] } } }
   ---
   ```

4. Démarrez une nouvelle session pour que le snapshot des Skills se rafraîchisse.

### Avez-vous une intégration Notion ou HeyGen

Pas incluse nativement aujourd'hui.

Options :

- **Skill / plugin personnalisé :** idéal pour un accès fiable aux API (Notion et HeyGen ont tous deux des API).
- **Automatisation du navigateur :** fonctionne sans code mais est plus lent et plus fragile.

Si vous voulez conserver le contexte par client (workflows d'agence), un schéma simple est :

- Une page Notion par client (contexte + préférences + travail en cours).
- Demandez à l'agent de récupérer cette page au début d'une session.

Si vous voulez une intégration native, ouvrez une demande de fonctionnalité ou créez une Skill ciblant ces API.

Installation de Skills :

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub installe dans `./skills` sous votre répertoire courant (ou se rabat sur votre espace de travail OpenClaw configure) ; OpenClaw traite cela comme `<workspace>/skills` à la prochaine session. Pour les Skills partagées entre agents, placez-les dans `~/.openclaw/skills/<nom>/SKILL.md`. Certaines Skills nécessitent des binaires installés via Homebrew ; sous Linux, cela signifie Linuxbrew (voir l'entrée FAQ Homebrew Linux ci-dessus). Voir [Skills](/tools/skills) et [ClawHub](/tools/clawhub).

### Comment installer l'extension Chrome pour la prise de contrôle du navigateur

Utilisez l'installeur intégré, puis chargez l'extension non empaquetée dans Chrome :

```bash
openclaw browser extension install
openclaw browser extension path
```

Puis Chrome → `chrome://extensions` → activez le « Mode développeur » → « Charger l'extension non empaquetée » → choisissez ce dossier.

Guide complet (y compris Gateway distant + notes de sécurité) : [Extension Chrome](/tools/chrome-extension)

Si le Gateway tourne sur la même machine que Chrome (configuration par défaut), vous n'avez généralement **pas** besoin de quoi que ce soit de plus.
Si le Gateway tourne ailleurs, exécutez un hôte de nœud sur la machine du navigateur pour que le Gateway puisse rediriger les actions via un proxy du navigateur.
Vous devez quand même cliquer sur le bouton de l'extension dans l'onglet que vous voulez contrôler (elle ne s'attache pas automatiquement).## Bac à sable et mémoire

### Existe-t-il une documentation dédiée au bac à sable {#is-there-a-dedicated-sandboxing-doc}

Oui. Voir [Sandboxing](/gateway/sandboxing). Pour la configuration spécifique à Docker (Gateway complet dans Docker ou images de bac à sable), voir [Docker](/install/docker).

### Docker semble limité. Comment activer toutes les fonctionnalités {#docker-feels-limited-how-do-i-enable-full-features}

L'image par défaut privilégie la sécurité et s'exécute sous l'utilisateur `node`, elle n'inclut donc pas les paquets système, Homebrew ni les navigateurs intégrés. Pour une configuration plus complète :

- Persistez `/home/node` avec `OPENCLAW_HOME_VOLUME` pour que les caches survivent.
- Intégrez les dépendances système dans l'image avec `OPENCLAW_DOCKER_APT_PACKAGES`.
- Installez les navigateurs Playwright via le CLI intégré :
  `node /app/node_modules/playwright-core/cli.js install chromium`
- Définissez `PLAYWRIGHT_BROWSERS_PATH` et assurez-vous que le chemin est persisté.

Docs : [Docker](/install/docker), [Browser](/tools/browser).

**Puis-je garder les DM privés mais rendre les groupes publics dans un bac à sable avec un seul agent**

Oui, si votre trafic privé passe par les **DM** et votre trafic public par les **groupes**.

Utilisez `agents.defaults.sandbox.mode: "non-main"` pour que les sessions de groupe/canal (clés non-main) s'exécutent dans Docker, tandis que la session DM principale reste sur l'hôte. Ensuite, restreignez les outils disponibles dans les sessions en bac à sable via `tools.sandbox.tools`.

Guide de configuration + exemple : [Groups: personal DMs + public groups](/channels/groups#pattern-personal-dms-public-groups-single-agent)

Référence de configuration clé : [Gateway configuration](/gateway/configuration#agentsdefaultssandbox)

### Comment monter un dossier hôte dans le bac à sable {#how-do-i-bind-a-host-folder-into-the-sandbox}

Définissez `agents.defaults.sandbox.docker.binds` sur `["host:path:mode"]` (par exemple, `"/home/user/src:/src:ro"`). Les binds globaux et par agent sont fusionnés ; les binds par agent sont ignorés lorsque `scope: "shared"`. Utilisez `:ro` pour tout ce qui est sensible et rappelez-vous que les binds contournent les barrières du système de fichiers du bac à sable. Voir [Sandboxing](/gateway/sandboxing#custom-bind-mounts) et [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check) pour des exemples et des notes de sécurité.

### Comment fonctionne la mémoire {#how-does-memory-work}

La mémoire d'OpenClaw repose sur de simples fichiers Markdown dans l'espace de travail de l'agent :

- Notes quotidiennes dans `memory/YYYY-MM-DD.md`
- Notes long terme curées dans `MEMORY.md` (sessions main/privées uniquement)

OpenClaw exécute également un **flush de mémoire silencieux pré-compaction** pour rappeler au modèle d'écrire des notes durables avant l'auto-compaction. Cela ne fonctionne que lorsque l'espace de travail est inscriptible (les bacs à sable en lecture seule sautent cette étape). Voir [Memory](/concepts/memory).

### La mémoire oublie sans cesse. Comment la rendre persistante {#memory-keeps-forgetting-things-how-do-i-make-it-stick}

Demandez au bot d'**écrire le fait en mémoire**. Les notes long terme appartiennent à `MEMORY.md`, le contexte à court terme va dans `memory/YYYY-MM-DD.md`.

C'est un domaine que nous continuons d'améliorer. Il est utile de rappeler au modèle de stocker les souvenirs ; il saura quoi faire. Si le problème persiste, vérifiez que le Gateway utilise le même espace de travail à chaque exécution.

Docs : [Memory](/concepts/memory), [Agent workspace](/concepts/agent-workspace).

### La recherche sémantique de mémoire nécessite-t-elle une clé API OpenAI {#does-semantic-memory-search-require-an-openai-api-key}

Uniquement si vous utilisez les **embeddings OpenAI**. Codex OAuth couvre chat/completions et n'accorde **pas** l'accès aux embeddings, donc **se connecter avec Codex (OAuth ou le login Codex CLI)** ne fonctionne pas pour la recherche sémantique de mémoire. Les embeddings OpenAI nécessitent toujours une vraie clé API (`OPENAI_API_KEY` ou `models.providers.openai.apiKey`).

Si vous ne définissez pas de fournisseur explicitement, OpenClaw sélectionne automatiquement un fournisseur lorsqu'il peut résoudre une clé API (profils d'authentification, `models.providers.*.apiKey`, ou variables d'environnement). Il préfère OpenAI si une clé OpenAI est résolue, sinon Gemini si une clé Gemini est résolue, puis Voyage, puis Mistral. Si aucune clé distante n'est disponible, la recherche mémoire reste désactivée jusqu'à ce que vous la configuriez. Si vous avez un chemin de modèle local configuré et présent, OpenClaw préfère `local`.

Si vous préférez rester en local, définissez `memorySearch.provider = "local"` (et optionnellement `memorySearch.fallback = "none"`). Si vous voulez les embeddings Gemini, définissez `memorySearch.provider = "gemini"` et fournissez `GEMINI_API_KEY` (ou `memorySearch.remote.apiKey`). Nous supportons les modèles d'embedding **OpenAI, Gemini, Voyage, Mistral ou local** ; voir [Memory](/concepts/memory) pour les détails de configuration.

### La mémoire persiste-t-elle indéfiniment ? Quelles sont les limités {#does-memory-persist-forever-what-are-the-limits}

Les fichiers de mémoire sont sur le disque et persistent jusqu'à ce que vous les supprimiez. La limité est votre espace de stockage, pas le modèle. Le **contexte de session** est toujours limité par la fenêtre de contexte du modèle, donc les longues conversations peuvent être compactées ou tronquées. C'est pourquoi la recherche mémoire existe : elle ne ramène que les parties pertinentes dans le contexte.

Docs : [Memory](/concepts/memory), [Context](/concepts/context).

## Où les données sont stockées sur le disque

### Toutes les données utilisées avec OpenClaw sont-elles sauvegardées localement {#is-all-data-used-with-openclaw-saved-locally}

Non. **L'état d'OpenClaw est local**, mais **les services externes voient toujours ce que vous leur envoyez**.

- **Local par défaut :** sessions, fichiers mémoire, configuration et espace de travail résident sur l'hôte du Gateway (`~/.openclaw` + votre répertoire d'espace de travail).
- **Distant par nécessité :** les messages que vous envoyez aux fournisseurs de modèles (Anthropic/OpenAI/etc.) transitent par leurs API, et les plateformes de chat (WhatsApp/Telegram/Slack/etc.) stockent les données de messages sur leurs serveurs.
- **Vous contrôlez l'empreinte :** utiliser des modèles locaux garde les prompts sur votre machine, mais le trafic des canaux passe toujours par les serveurs du canal.

Voir aussi : [Agent workspace](/concepts/agent-workspace), [Memory](/concepts/memory).

### Où OpenClaw stocke-t-il ses données {#where-does-openclaw-store-its-data}

Tout se trouve sous `$OPENCLAW_STATE_DIR` (défaut : `~/.openclaw`) :

| Chemin                                                          | Fonction                                                           |
| --------------------------------------------------------------- | ------------------------------------------------------------------ |
| `$OPENCLAW_STATE_DIR/openclaw.json`                             | Configuration principale (JSON5)                                   |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json`                    | Import OAuth legacy (copié dans les profils d'auth au premier usage) |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | Profils d'auth (OAuth, clés API, et optionnel `keyRef`/`tokenRef`) |
| `$OPENCLAW_STATE_DIR/secrets.json`                              | Payload de secrets optionnel pour les fournisseurs SecretRef `file` |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json`          | Fichier de compatibilité legacy (entrées `api_key` statiques supprimées) |
| `$OPENCLAW_STATE_DIR/credentials/`                              | État des fournisseurs (ex. `whatsapp/<accountId>/creds.json`)      |
| `$OPENCLAW_STATE_DIR/agents/`                                   | État par agent (agentDir + sessions)                               |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`                | Historique et état des conversations (par agent)                   |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json`   | Métadonnées de session (par agent)                                 |

Chemin legacy mono-agent : `~/.openclaw/agent/*` (migré par `openclaw doctor`).

Votre **espace de travail** (AGENTS.md, fichiers mémoire, Skills, etc.) est séparé et configure via `agents.defaults.workspace` (défaut : `~/.openclaw/workspace`).

### Où doivent se trouver AGENTS.md / SOUL.md / USER.md / MEMORY.md {#where-should-agentsmd-soulmd-usermd-memorymd-live}

Ces fichiers se trouvent dans l'**espace de travail de l'agent**, pas dans `~/.openclaw`.

- **Espace de travail (par agent)** : `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (ou `memory.md`), `memory/YYYY-MM-DD.md`, optionnel `HEARTBEAT.md`.
- **Répertoire d'état (`~/.openclaw`)** : configuration, identifiants, profils d'authentification, sessions, logs,
  et Skills partagés (`~/.openclaw/skills`).

L'espace de travail par défaut est `~/.openclaw/workspace`, configurable via :

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

Si le bot « oublie » après un redémarrage, confirmez que le Gateway utilise le même espace de travail à chaque lancement (et rappelez-vous : le mode distant utilisé l'espace de travail de l'**hôte du Gateway**, pas celui de votre laptop local).

Astuce : si vous souhaitez un comportement ou une préférence durable, demandez au bot de **l'écrire dans AGENTS.md ou MEMORY.md** plutôt que de compter sur l'historique du chat.

Voir [Agent workspace](/concepts/agent-workspace) et [Memory](/concepts/memory).

### Quelle est la stratégie de sauvegarde recommandée {#whats-the-recommended-backup-strategy}

Placez votre **espace de travail de l'agent** dans un dépôt **privé** git et sauvegardez-le quelque part de privé (par exemple GitHub privé). Cela capture la mémoire + les fichiers AGENTS/SOUL/USER, et vous permet de restaurer l'« esprit » de l'assistant plus tard.

Ne commitez **jamais** quoi que ce soit sous `~/.openclaw` (identifiants, sessions, jetons ou payloads de secrets chiffrés). Si vous avez besoin d'une restauration complète, sauvegardez l'espace de travail et le répertoire d'état séparément (voir la question sur la migration ci-dessus).

Docs : [Agent workspace](/concepts/agent-workspace).

### Comment désinstaller complètement OpenClaw {#how-do-i-completely-uninstall-openclaw}

Voir le guide dédié : [Uninstall](/install/uninstall).

### Les agents peuvent-ils travailler en dehors de l'espace de travail {#can-agents-work-outside-the-workspace}

Oui. L'espace de travail est le **répertoire de travail par défaut** et l'ancrage mémoire, pas un bac à sable strict. Les chemins relatifs se résolvent à l'intérieur de l'espace de travail, mais les chemins absolus peuvent accéder à d'autres emplacements de l'hôte sauf si le bac à sable est activé. Si vous avez besoin d'isolation, utilisez [`agents.defaults.sandbox`](/gateway/sandboxing) ou les paramètres de bac à sable par agent. Si vous voulez qu'un dépôt soit le répertoire de travail par défaut, pointez le `workspace` de cet agent vers la racine du dépôt. Le dépôt OpenClaw n'est que du code source ; gardez l'espace de travail séparé sauf si vous voulez intentionnellement que l'agent travaille dedans.

Exemple (dépôt comme répertoire de travail par défaut) :

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo",
    },
  },
}
```

### Je suis en mode distant, où se trouve le magasin de sessions {#im-in-remote-mode-where-is-the-session-store}

L'état de session appartient à l'**hôte du Gateway**. Si vous êtes en mode distant, le magasin de sessions qui vous intéresse se trouve sur la machine distante, pas sur votre laptop local. Voir [Session management](/concepts/session).

## Bases de la configuration

### Quel est le format de la configuration ? Où est-elle {#what-format-is-the-config-where-is-it}

OpenClaw lit une configuration optionnelle **JSON5** depuis `$OPENCLAW_CONFIG_PATH` (défaut : `~/.openclaw/openclaw.json`) :

```
$OPENCLAW_CONFIG_PATH
```

Si le fichier est absent, il utilise des valeurs par défaut raisonnables (y compris un espace de travail par défaut de `~/.openclaw/workspace`).

### J'ai défini gateway.bind sur lan ou tailnet et maintenant rien n'écoute / l'interface dit unauthorized {#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized}

Les binds non-loopback **nécessitent une authentification**. Configurez `gateway.auth.mode` + `gateway.auth.token` (ou utilisez `OPENCLAW_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me",
    },
  },
}
```

Notes :

- `gateway.remote.token` / `.password` n'activent **pas** l'auth du Gateway local par eux-mêmes.
- Les chemins d'appel locaux peuvent utiliser `gateway.remote.*` comme fallback lorsque `gateway.auth.*` n'est pas défini.
- L'interface Control UI s'authentifié via `connect.params.auth.token` (stocké dans les paramètres de l'app/UI). Évitez de mettre les jetons dans les URL.

### Pourquoi ai-je besoin d'un jeton sur localhost maintenant {#why-do-i-need-a-token-on-localhost-now}

OpenClaw impose l'authentification par jeton par défaut, y compris en loopback. Si aucun jeton n'est configuré, le démarrage du Gateway en génère un automatiquement et le sauvegarde dans `gateway.auth.token`, donc **les clients WS locaux doivent s'authentifier**. Cela empêche d'autres processus locaux d'appeler le Gateway.

Si vous voulez **vraiment** un loopback ouvert, définissez `gateway.auth.mode: "none"` explicitement dans votre configuration. Doctor peut générer un jeton pour vous à tout moment : `openclaw doctor --generate-gateway-token`.

### Dois-je redémarrer après avoir modifié la configuration {#do-i-have-to-restart-after-changing-config}

Le Gateway surveille la configuration et supporte le rechargement à chaud :

- `gateway.reload.mode: "hybrid"` (défaut) : applique les changements sûrs à chaud, redémarre pour les changements critiques
- `hot`, `restart`, `off` sont également supportés

### Comment activer la recherche web et le web fetch {#how-do-i-enable-web-search-and-web-fetch}

`web_fetch` fonctionne sans clé API. `web_search` nécessite une clé API Brave Search. **Recommandé :** exécutez `openclaw configuré --section web` pour la stocker dans `tools.web.search.apiKey`. Alternative par variable d'environnement : définissez `BRAVE_API_KEY` pour le processus Gateway.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
      },
      fetch: {
        enabled: true,
      },
    },
  },
}
```

Notes :

- Si vous utilisez des listes d'autorisation, ajoutez `web_search`/`web_fetch` ou `group:web`.
- `web_fetch` est activé par défaut (sauf si explicitement désactivé).
- Les daemons lisent les variables d'environnement depuis `~/.openclaw/.env` (ou l'environnement du service).

Docs : [Web tools](/tools/web).

### Comment exécuter un Gateway central avec des workers spécialisés sur différents appareils {#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices}

Le schéma classique est **un Gateway** (par exemple un Raspberry Pi) plus des **nœuds** et des **agents** :

- **Gateway (central) :** gère les canaux (Signal/WhatsApp), le routage et les sessions.
- **Nœuds (appareils) :** les Mac/iOS/Android se connectent comme périphériques et exposent les outils locaux (`system.run`, `canvas`, `camera`).
- **Agents (workers) :** des cerveaux/espaces de travail séparés pour des rôles spéciaux (par ex. « Ops Hetzner », « Données personnelles »).
- **Sous-agents :** lancent du travail en arrière-plan depuis un agent principal quand vous voulez du parallélisme.
- **TUI :** se connecte au Gateway et permet de basculer entre agents/sessions.

Docs : [Nodes](/nodes), [Remote access](/gateway/remote), [Multi-Agent Routing](/concepts/multi-agent), [Sub-agents](/tools/subagents), [TUI](/web/tui).

### Le navigateur OpenClaw peut-il fonctionner en headless {#can-the-openclaw-browser-run-headless}

Oui. C'est une option de configuration :

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } },
    },
  },
}
```

La valeur par défaut est `false` (avec interface graphique). Le mode headless est plus susceptible de déclencher des vérifications anti-bot sur certains sites. Voir [Browser](/tools/browser).

Le mode headless utilisé le **même moteur Chromium** et fonctionne pour la plupart des automatisations (formulaires, clics, scraping, connexions). Les principales différences :

- Pas de fenêtre de navigateur visible (utilisez les captures d'écran si vous avez besoin de visuels).
- Certains sites sont plus stricts avec l'automatisation en mode headless (CAPTCHAs, anti-bot).
  Par exemple, X/Twitter bloque souvent les sessions headless.

### Comment utiliser Brave pour le contrôle du navigateur {#how-do-i-use-brave-for-browser-control}

Définissez `browser.executablePath` sur le binaire de votre navigateur Brave (ou tout navigateur basé sur Chromium) et redémarrez le Gateway.
Voir les exemples de configuration complets dans [Browser](/tools/browser#use-brave-or-another-chromium-based-browser).

## Gateways distants et nœuds

### Comment les commandes se propagent-elles entre Telegram, le Gateway et les nœuds {#how-do-commands-propagate-between-telegram-the-gateway-and-nodes}

Les messages Telegram sont traités par le **Gateway**. Le Gateway exécute l'agent et n'appelle les nœuds via le **WebSocket du Gateway** que lorsqu'un outil de nœud est nécessaire :

Telegram → Gateway → Agent → `node.*` → Nœud → Gateway → Telegram

Les nœuds ne voient pas le trafic entrant du fournisseur ; ils ne reçoivent que des appels RPC de nœud.

### Comment mon agent peut-il accéder à mon ordinateur si le Gateway est hébergé à distance {#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely}

Réponse courte : **appairez votre ordinateur comme nœud**. Le Gateway tourne ailleurs, mais il peut appeler les outils `node.*` (écran, caméra, système) sur votre machine locale via le WebSocket du Gateway.

Configuration typique :

1. Exécutez le Gateway sur l'hôte toujours allumé (VPS/serveur domestique).
2. Mettez l'hôte du Gateway + votre ordinateur sur le même tailnet.
3. Assurez-vous que le WS du Gateway est accessible (bind tailnet ou tunnel SSH).
4. Ouvrez l'app macOS en local et connectez-vous en mode **Remote over SSH** (ou tailnet direct) pour qu'elle puisse s'enregistrer comme nœud.
5. Approuvez le nœud sur le Gateway :

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Aucun pont TCP séparé n'est nécessaire ; les nœuds se connectent via le WebSocket du Gateway.

Rappel de sécurité : appairer un nœud macOS autorise `system.run` sur cette machine. N'appairez que des appareils de confiance, et consultez [Security](/gateway/security).

Docs : [Nodes](/nodes), [Gateway protocol](/gateway/protocol), [macOS remote mode](/platforms/mac/remote), [Security](/gateway/security).

### Tailscale est connecté mais je ne reçois pas de réponses. Que faire {#tailscale-is-connected-but-i-get-no-replies-what-now}

Vérifiez les bases :

- Le Gateway tourne : `openclaw gateway status`
- Santé du Gateway : `openclaw status`
- Santé des canaux : `openclaw channels status`

Puis vérifiez l'auth et le routage :

- Si vous utilisez Tailscale Serve, assurez-vous que `gateway.auth.allowTailscale` est correctement défini.
- Si vous vous connectez via un tunnel SSH, confirmez que le tunnel local est actif et pointe vers le bon port.
- Confirmez que vos listes d'autorisation (DM ou groupe) incluent votre compte.

Docs : [Tailscale](/gateway/tailscale), [Remote access](/gateway/remote), [Channels](/channels).

### Deux instances OpenClaw peuvent-elles communiquer entre elles (local + VPS) {#can-two-openclaw-instances-talk-to-each-other-local-vps}

Oui. Il n'y a pas de pont « bot-à-bot » intégré, mais vous pouvez le mettre en place de plusieurs façons fiables :

**Le plus simple :** utilisez un canal de chat normal auquel les deux bots ont accès (Telegram/Slack/WhatsApp). Faites envoyer un message du Bot A au Bot B, puis laissez le Bot B répondre normalement.

**Pont CLI (générique) :** exécutez un script qui appelle l'autre Gateway avec `openclaw agent --message ... --deliver`, ciblant un chat où l'autre bot écoute. Si un bot est sur un VPS distant, pointez votre CLI vers ce Gateway distant via SSH/Tailscale (voir [Remote access](/gateway/remote)).

Exemple de schéma (exécuté depuis une machine qui peut atteindre le Gateway cible) :

```bash
openclaw agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

Astuce : ajoutez une garde-fou pour que les deux bots ne bouclent pas indéfiniment (mention uniquement, listes d'autorisation de canal, ou une règle « ne pas répondre aux messages de bot »).

Docs : [Remote access](/gateway/remote), [Agent CLI](/cli/agent), [Agent send](/tools/agent-send).

### Ai-je besoin de VPS séparés pour plusieurs agents {#do-i-need-separate-vpses-for-multiple-agents}

Non. Un seul Gateway peut héberger plusieurs agents, chacun avec son propre espace de travail, ses modèles par défaut et son routage. C'est la configuration normale et c'est bien moins cher et plus simple que d'exécuter un VPS par agent.

Utilisez des VPS séparés uniquement quand vous avez besoin d'une isolation stricte (frontières de sécurité) ou de configurations très différentes que vous ne voulez pas partager. Sinon, gardez un seul Gateway et utilisez plusieurs agents ou sous-agents.

### Y a-t-il un avantage à utiliser un nœud sur mon laptop personnel plutôt que SSH depuis un VPS {#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps}

Oui. Les nœuds sont le moyen privilégié pour atteindre votre laptop depuis un Gateway distant, et ils offrent plus que le simple accès shell. Le Gateway tourne sur macOS/Linux (Windows via WSL2) et est léger (un petit VPS ou un boîtier type Raspberry Pi suffit ; 4 Go de RAM suffisent), donc une configuration courante est un hôte toujours allumé plus votre laptop comme nœud.

- **Pas de SSH entrant requis.** Les nœuds se connectent en sortie au WebSocket du Gateway et utilisent l'appairage d'appareil.
- **Contrôles d'exécution plus sûrs.** `system.run` est restreint par les listes d'autorisation/approbations de nœud sur ce laptop.
- **Plus d'outils d'appareil.** Les nœuds exposent `canvas`, `camera` et `screen` en plus de `system.run`.
- **Automatisation de navigateur local.** Gardez le Gateway sur un VPS, mais exécutez Chrome en local et relayez le contrôle avec l'extension Chrome + un hôte de nœud sur le laptop.

SSH convient pour un accès shell ponctuel, mais les nœuds sont plus simples pour les workflows d'agent continus et l'automatisation d'appareils.

Docs : [Nodes](/nodes), [Nodes CLI](/cli/nodes), [Chrome extension](/tools/chrome-extension).

### Dois-je installer sur un second laptop ou simplement ajouter un nœud {#should-i-install-on-a-second-laptop-or-just-add-a-node}

Si vous n'avez besoin que d'**outils locaux** (écran/caméra/exec) sur le second laptop, ajoutez-le comme **nœud**. Cela maintient un seul Gateway et évite les configurations dupliquées. Les outils de nœud locaux sont actuellement réservés à macOS, mais nous prévoyons de les étendre à d'autres OS.

Installez un second Gateway uniquement quand vous avez besoin d'une **isolation stricte** ou de deux bots entièrement séparés.

Docs : [Nodes](/nodes), [Nodes CLI](/cli/nodes), [Multiple gateways](/gateway/multiple-gateways).

### Les nœuds exécutent-ils un service Gateway {#do-nodes-run-a-gateway-service}

Non. Un seul **Gateway** devrait tourner par hôte, sauf si vous exécutez intentionnellement des profils isolés (voir [Multiple gateways](/gateway/multiple-gateways)). Les nœuds sont des périphériques qui se connectent au Gateway (nœuds iOS/Android, ou macOS en « mode nœud » dans l'app de la barre de menus). Pour les hôtes de nœud headless et le contrôle CLI, voir [Node host CLI](/cli/node).

Un redémarrage complet est requis pour les changements de `gateway`, `discovery` et `canvasHost`.

### Existe-t-il une API / RPC pour appliquer la configuration {#is-there-an-api-rpc-way-to-apply-config}

Oui. `config.apply` valide + écrit la configuration complète et redémarre le Gateway dans le cadre de l'opération.

### config.apply a effacé ma configuration. Comment récupérer et éviter ça {#configapply-wiped-my-config-how-do-i-recover-and-avoid-this}

`config.apply` remplace la configuration **entière**. Si vous envoyez un objet partiel, tout le reste est supprimé.

Récupération :

- Restaurez depuis une sauvegarde (git ou une copie de `~/.openclaw/openclaw.json`).
- Si vous n'avez pas de sauvegarde, relancez `openclaw doctor` et reconfigurez les canaux/modèles.
- Si c'était inattendu, signalez un bug et incluez votre dernière configuration connue ou toute sauvegarde.
- Un agent de codage local peut souvent reconstruire une configuration fonctionnelle à partir des logs ou de l'historique.

Prévention :

- Utilisez `openclaw config set` pour les petits changements.
- Utilisez `openclaw configuré` pour les modifications interactives.

Docs : [Config](/cli/config), [Configuré](/cli/configure), [Doctor](/gateway/doctor).

### Quelle est la configuration minimale « saine » pour une première installation {#whats-a-minimal-sane-config-for-a-first-install}

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Cela définit votre espace de travail et restreint qui peut déclencher le bot.

### Comment configurer Tailscale sur un VPS et se connecter depuis mon Mac {#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac}

Étapes minimales :

1. **Installer + se connecter sur le VPS**

   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```

2. **Installer + se connecter sur votre Mac**
   - Utilisez l'app Tailscale et connectez-vous au même tailnet.
3. **Activer MagicDNS (recommandé)**
   - Dans la console d'administration Tailscale, activez MagicDNS pour que le VPS ait un nom stable.
4. **Utiliser le hostname du tailnet**
   - SSH : `ssh user@your-vps.tailnet-xxxx.ts.net`
   - Gateway WS : `ws://your-vps.tailnet-xxxx.ts.net:18789`

Si vous voulez l'interface Control UI sans SSH, utilisez Tailscale Serve sur le VPS :

```bash
openclaw gateway --tailscale serve
```

Cela garde le Gateway en bind sur loopback et expose du HTTPS via Tailscale. Voir [Tailscale](/gateway/tailscale).

### Comment connecter un nœud Mac à un Gateway distant (Tailscale Serve) {#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve}

Serve expose l'**interface Control UI + WS du Gateway**. Les nœuds se connectent via le même endpoint WS du Gateway.

Configuration recommandée :

1. **Assurez-vous que le VPS + Mac sont sur le même tailnet**.
2. **Utilisez l'app macOS en mode Remote** (la cible SSH peut être le hostname du tailnet).
   L'app ouvrira un tunnel vers le port du Gateway et se connectera comme nœud.
3. **Approuvez le nœud** sur le Gateway :

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Docs : [Gateway protocol](/gateway/protocol), [Discovery](/gateway/discovery), [macOS remote mode](/platforms/mac/remote).

## Variables d'environnement et chargement .env

### Comment OpenClaw charge-t-il les variables d'environnement {#how-does-openclaw-load-environment-variables}

OpenClaw lit les variables d'environnement depuis le processus parent (shell, launchd/systemd, CI, etc.) et charge en plus :

- `.env` depuis le répertoire de travail courant
- un `.env` global de secours depuis `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Aucun fichier `.env` ne remplace les variables d'environnement existantes.

Vous pouvez également définir des variables d'environnement en ligne dans la configuration (appliquées uniquement si absentes de l'environnement du processus) :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

Voir [/environment](/help/environment) pour la précédence complète et les sources.

### J'ai démarré le Gateway via le service et mes variables d'environnement ont disparu. Que faire {#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now}

Deux correctifs courants :

1. Placez les clés manquantes dans `~/.openclaw/.env` pour qu'elles soient récupérées même quand le service n'hérite pas de votre environnement shell.
2. Activez l'import shell (commodité opt-in) :

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Cela lance votre shell de connexion et importe uniquement les clés attendues manquantes (ne remplace jamais). Équivalents en variables d'environnement :
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

### J'ai défini COPILOT_GITHUB_TOKEN mais models status affiche « Shell env: off ». Pourquoi {#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why}

`openclaw models status` indique si l'**import de l'environnement shell** est activé. « Shell env: off » ne signifie **pas** que vos variables d'environnement sont absentes : cela signifie simplement qu'OpenClaw ne chargera pas automatiquement votre shell de connexion.

Si le Gateway tourne comme service (launchd/systemd), il n'héritera pas de votre environnement shell. Corrigez en faisant l'une de ces actions :

1. Placez le jeton dans `~/.openclaw/.env` :

   ```
   COPILOT_GITHUB_TOKEN=...
   ```

2. Ou activez l'import shell (`env.shellEnv.enabled: true`).
3. Ou ajoutez-le à votre bloc de configuration `env` (ne s'applique que si absent).

Puis redémarrez le Gateway et vérifiez à nouveau :

```bash
openclaw models status
```

Les jetons Copilot sont lus depuis `COPILOT_GITHUB_TOKEN` (aussi `GH_TOKEN` / `GITHUB_TOKEN`).
Voir [/concepts/model-providers](/concepts/model-providers) et [/environment](/help/environment).

## Sessions et conversations multiples

### Comment démarrer une nouvelle conversation {#how-do-i-start-a-fresh-conversation}

Envoyez `/new` ou `/reset` comme message autonome. Voir [Session management](/concepts/session).

### Les sessions se réinitialisent-elles automatiquement si je n'envoie jamais /new {#do-sessions-reset-automatically-if-i-never-send-new}

Oui. Les sessions expirent après `session.idleMinutes` (défaut **60**). Le **prochain** message démarre un nouvel identifiant de session pour cette clé de chat. Cela ne supprime pas les transcriptions : cela démarre simplement une nouvelle session.

```json5
{
  session: {
    idleMinutes: 240,
  },
}
```

### Existe-t-il un moyen de créer une équipe d'instances OpenClaw : un CEO et plusieurs agents {#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents}

Oui, via le **routage multi-agent** et les **sous-agents**. Vous pouvez créer un agent coordinateur et plusieurs agents workers avec leurs propres espaces de travail et modèles.

Cela dit, considérez cela comme une **expérience amusante**. C'est coûteux en jetons et souvent moins efficace qu'utiliser un seul bot avec des sessions séparées. Le modèle typique que nous envisageons est un bot à qui vous parlez, avec différentes sessions pour le travail en parallèle. Ce bot peut aussi lancer des sous-agents selon les besoins.

Docs : [Multi-agent routing](/concepts/multi-agent), [Sub-agents](/tools/subagents), [Agents CLI](/cli/agents).

### Pourquoi le contexte a-t-il été tronqué en pleine tâche ? Comment l'empêcher {#why-did-context-get-truncated-midtask-how-do-i-prevent-it}

Le contexte de session est limité par la fenêtre du modèle. Les longs chats, les sorties d'outils volumineuses ou les nombreux fichiers peuvent déclencher une compaction ou une troncation.

Ce qui aide :

- Demandez au bot de résumer l'état actuel et de l'écrire dans un fichier.
- Utilisez `/compact` avant les tâches longues, et `/new` quand vous changez de sujet.
- Gardez le contexte important dans l'espace de travail et demandez au bot de le relire.
- Utilisez des sous-agents pour le travail long ou parallèle afin que le chat principal reste plus petit.
- Choisissez un modèle avec une fenêtre de contexte plus grande si cela arrive souvent.

### Comment réinitialiser complètement OpenClaw tout en le gardant installé {#how-do-i-completely-reset-openclaw-but-keep-it-installed}

Utilisez la commande de réinitialisation :

```bash
openclaw reset
```

Réinitialisation complète non-interactive :

```bash
openclaw reset --scope full --yes --non-interactive
```

Puis relancez la configuration initiale :

```bash
openclaw onboard --install-daemon
```

Notes :

- L'assistant de configuration initiale propose aussi **Reset** s'il détecte une configuration existante. Voir [Wizard](/start/wizard).
- Si vous avez utilisé des profils (`--profile` / `OPENCLAW_PROFILE`), réinitialisez chaque répertoire d'état (les défauts sont `~/.openclaw-<profile>`).
- Réinitialisation dev : `openclaw gateway --dev --reset` (dev uniquement ; efface la configuration dev + identifiants + sessions + espace de travail).

### J'obtiens des erreurs « context too large ». Comment réinitialiser ou compacter {#im-getting-context-too-large-errors-how-do-i-reset-or-compact}

Utilisez l'une de ces méthodes :

- **Compacter** (garde la conversation mais résume les anciens tours) :

  ```
  /compact
  ```

  ou `/compact <instructions>` pour guider le résumé.

- **Réinitialiser** (nouvel identifiant de session pour la même clé de chat) :

  ```
  /new
  /reset
  ```

Si le problème persiste :

- Activez ou ajustez l'**élagage de session** (`agents.defaults.contextPruning`) pour supprimer les anciennes sorties d'outils.
- Utilisez un modèle avec une fenêtre de contexte plus grande.

Docs : [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning), [Session management](/concepts/session).

### Pourquoi est-ce que je vois « LLM request rejected: messages.content.tool_use.input field required » {#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required}

C'est une erreur de validation du fournisseur : le modèle a émis un bloc `tool_use` sans le champ `input` requis. Cela signifie généralement que l'historique de session est périmé ou corrompu (souvent après de longs fils de discussion ou un changement de schéma d'outil).

Correction : démarrez une nouvelle session avec `/new` (message autonome).

### Pourquoi est-ce que je reçois des messages heartbeat toutes les 30 minutes {#why-am-i-getting-heartbeat-messages-every-30-minutes}

Les heartbeats s'exécutent toutes les **30 minutes** par défaut. Ajustez ou désactivez-les :

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h", // ou "0m" pour désactiver
      },
    },
  },
}
```

Si `HEARTBEAT.md` existe mais est effectivement vide (uniquement des lignes vides et des en-têtes Markdown comme `# Heading`), OpenClaw saute l'exécution du heartbeat pour économiser les appels API. Si le fichier est absent, le heartbeat s'exécute quand même et le modèle décide quoi faire.

Les remplacements par agent utilisent `agents.list[].heartbeat`. Docs : [Heartbeat](/gateway/heartbeat).

### Dois-je ajouter un « compte bot » à un groupe WhatsApp {#do-i-need-to-add-a-bot-account-to-a-whatsapp-group}

Non. OpenClaw fonctionne sur **votre propre compte**, donc si vous êtes dans le groupe, OpenClaw peut le voir. Par défaut, les réponses de groupe sont bloquées tant que vous n'autorisez pas les expéditeurs (`groupPolicy: "allowlist"`).

Si vous voulez que seul **vous** puissiez déclencher les réponses de groupe :

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### Comment obtenir le JID d'un groupe WhatsApp {#how-do-i-get-the-jid-of-a-whatsapp-group}

Option 1 (la plus rapide) : suivez les logs et envoyez un message test dans le groupe :

```bash
openclaw logs --follow --json
```

Cherchez `chatId` (ou `from`) se terminant par `@g.us`, comme :
`1234567890-1234567890@g.us`.

Option 2 (si déjà configuré/autorisé) : listez les groupes depuis la configuration :

```bash
openclaw directory groups list --channel whatsapp
```

Docs : [WhatsApp](/channels/whatsapp), [Directory](/cli/directory), [Logs](/cli/logs).

### Pourquoi OpenClaw ne répond-il pas dans un groupe {#why-doesnt-openclaw-reply-in-a-group}

Deux causes fréquentes :

- La restriction par mention est activé (par défaut). Vous devez @mentionner le bot (ou correspondre à `mentionPatterns`).
- Vous avez configuré `channels.whatsapp.groups` sans `"*"` et le groupe n'est pas dans la liste d'autorisation.

Voir [Groups](/channels/groups) et [Group messages](/channels/group-messages).

### Les groupes/threads partagent-ils le contexte avec les DM {#do-groupsthreads-share-context-with-dms}

Les chats directs se replient sur la session principale par défaut. Les groupes/canaux ont leurs propres clés de session, et les topics Telegram / threads Discord sont des sessions séparées. Voir [Groups](/channels/groups) et [Group messages](/channels/group-messages).

### Combien d'espaces de travail et d'agents puis-je créer {#how-many-workspaces-and-agents-can-i-create}

Pas de limités strictes. Des dizaines (voire des centaines) fonctionnent sans problème, mais surveillez :

- **Croissance du disque :** sessions + transcriptions résident sous `~/.openclaw/agents/<agentId>/sessions/`.
- **Coût en jetons :** plus d'agents signifie plus d'utilisation concurrente des modèles.
- **Charge opérationnelle :** profils d'authentification par agent, espaces de travail et routage des canaux.

Conseils :

- Gardez un **seul** espace de travail actif par agent (`agents.defaults.workspace`).
- Purgez les anciennes sessions (supprimez les JSONL ou les entrées du store) si le disque grossit.
- Utilisez `openclaw doctor` pour repérer les espaces de travail orphelins et les incohérences de profil.

### Puis-je exécuter plusieurs bots ou chats en même temps (Slack) et comment configurer cela {#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up}

Oui. Utilisez le **routage multi-agent** pour exécuter plusieurs agents isolés et router les messages entrants par canal/compte/pair. Slack est supporté comme canal et peut être lié à des agents spécifiques.

L'accès au navigateur est puissant mais ne permet pas de « faire tout ce qu'un humain peut faire » : l'anti-bot, les CAPTCHAs et le MFA peuvent toujours bloquer l'automatisation. Pour le contrôle de navigateur le plus fiable, utilisez le relais de l'extension Chrome sur la machine qui exécute le navigateur (et gardez le Gateway n'importe où).

Configuration recommandée :

- Hôte Gateway toujours allumé (VPS/Mac mini).
- Un agent par rôle (bindings).
- Canal(aux) Slack liés à ces agents.
- Navigateur local via le relais d'extension (ou un nœud) si nécessaire.

Docs : [Multi-Agent Routing](/concepts/multi-agent), [Slack](/channels/slack),
[Browser](/tools/browser), [Chrome extension](/tools/chrome-extension), [Nodes](/nodes).

## Modèles : défauts, sélection, alias, changement

### Quel est le modèle par défaut

Le modèle par défaut d'OpenClaw est celui que vous définissez dans :

```
agents.defaults.model.primary
```

Les modèles sont référencés au format `provider/model` (exemple : `anthropic/claude-opus-4-6`). Si vous omettez le fournisseur, OpenClaw utilise temporairement `anthropic` comme solution de repli dépréciée, mais vous devriez toujours définir **explicitement** `provider/model`.

### Quel modèle recommandez-vous

**Défaut recommandé :** `anthropic/claude-opus-4-6`.
**Bonne alternative :** `anthropic/claude-sonnet-4-5`.
**Fiable (moins de personnalité) :** `openai/gpt-5.2` : presque aussi bon qu'Opus, juste moins de caractère.
**Économique :** `zai/glm-4.7`.

MiniMax M2.1 a sa propre documentation : [MiniMax](/providers/minimax) et
[Modèles locaux](/gateway/local-models).

Règle générale : utilisez le **meilleur modèle que vous pouvez vous permettre** pour les tâches critiques, et un modèle moins cher pour le chat courant ou les résumés. Vous pouvez router les modèles par agent et utiliser des sous-agents pour paralléliser les tâches longues (chaque sous-agent consomme des tokens). Voir [Modèles](/concepts/models) et [Sous-agents](/tools/subagents).

Avertissement important : les modèles plus faibles ou sur-quantifiés sont plus vulnérables à l'injection de prompts et aux comportements non sécurisés. Voir [Sécurité](/gateway/security).

Plus de détails : [Modèles](/concepts/models).

### Puis-je utiliser des modèles auto-hébergés (llama.cpp, vLLM, Ollama)

Oui. Si votre serveur local expose une API compatible OpenAI, vous pouvez configurer un fournisseur personnalisé pour y pointer. Ollama est supporté directement et constitue le chemin le plus simple.

Note de sécurité : les modèles plus petits ou fortement quantifiés sont plus vulnérables à l'injection de prompts. Nous recommandons fortement les **grands modèles** pour tout bot ayant accès à des outils. Si vous souhaitez quand même utiliser des petits modèles, activez le bac à sable et des listes d'outils autorisés strictes.

Documentation : [Ollama](/providers/ollama), [Modèles locaux](/gateway/local-models),
[Fournisseurs de modèles](/concepts/model-providers), [Sécurité](/gateway/security),
[Bac à sable](/gateway/sandboxing).

### Comment changer de modèle sans écraser ma configuration

Utilisez les **commandes de modèle** ou modifiez uniquement les champs **model**. Évitez les remplacements complets de configuration.

Options sûres :

- `/model` dans le chat (rapide, par session)
- `openclaw models set ...` (met à jour uniquement la configuration du modèle)
- `openclaw configuré --section model` (interactif)
- Modifier `agents.defaults.model` dans `~/.openclaw/openclaw.json`

Évitez `config.apply` avec un objet partiel sauf si vous avez l'intention de remplacer toute la configuration. Si vous avez écrasé la configuration, restaurez depuis une sauvegarde ou relancez `openclaw doctor` pour réparer.

Documentation : [Modèles](/concepts/models), [Configuré](/cli/configure), [Config](/cli/config), [Doctor](/gateway/doctor).

### Quels modèles OpenClaw, Flawd et Krill utilisent-ils

- **OpenClaw + Flawd :** Anthropic Opus (`anthropic/claude-opus-4-6`) : voir [Anthropic](/providers/anthropic).
- **Krill :** MiniMax M2.1 (`minimax/MiniMax-M2.1`) : voir [MiniMax](/providers/minimax).

### Comment changer de modèle à la volée sans redémarrer

Utilisez la commande `/model` comme message autonome :

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

Vous pouvez lister les modèles disponibles avec `/model`, `/model list` ou `/model status`.

`/model` (et `/model list`) affiche un sélecteur compact numéroté. Sélectionnez par numéro :

```
/model 3
```

Vous pouvez aussi forcer un profil d'authentification spécifique pour le fournisseur (par session) :

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Astuce : `/model status` montre quel agent est actif, quel fichier `auth-profiles.json` est utilisé et quel profil d'authentification sera essayé ensuite.
Il affiche aussi le point de terminaison du fournisseur configuré (`baseUrl`) et le mode API (`api`) lorsqu'ils sont disponibles.

**Comment détacher un profil défini avec profile**

Relancez `/model` **sans** le suffixe `@profile` :

```
/model anthropic/claude-opus-4-6
```

Si vous voulez revenir au défaut, choisissez-le depuis `/model` (ou envoyez `/model <provider/model par défaut>`).
Utilisez `/model status` pour confirmer quel profil d'authentification est actif.

### Puis-je utiliser GPT 5.2 pour les tâches courantes et Codex 5.3 pour le code

Oui. Définissez l'un comme défaut et changez selon les besoins :

- **Changement rapide (par session) :** `/model gpt-5.2` pour les tâches courantes, `/model gpt-5.3-codex` pour le code.
- **Défaut + changement :** définissez `agents.defaults.model.primary` sur `openai/gpt-5.2`, puis basculez vers `openai-codex/gpt-5.3-codex` pour le code (ou inversement).
- **Sous-agents :** routez les tâches de code vers des sous-agents avec un modèle par défaut différent.

Voir [Modèles](/concepts/models) et [Commandes slash](/tools/slash-commands).

### Pourquoi est-ce que je vois « Model is not allowed » suivi d'aucune réponse

Si `agents.defaults.models` est défini, il devient la **liste autorisée** pour `/model` et tout remplacement de session. Choisir un modèle qui n'est pas dans cette liste retourne :

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Cette erreur est retournée **à la place** d'une réponse normale. Correction : ajoutez le modèle à `agents.defaults.models`, supprimez la liste autorisée, ou choisissez un modèle depuis `/model list`.

### Pourquoi est-ce que je vois « Unknown model minimaxMiniMaxM21 »

Cela signifie que le **fournisseur n'est pas configuré** (aucune configuration de fournisseur MiniMax ou profil d'authentification n'a été trouvé), donc le modèle ne peut pas être résolu. Un correctif pour cette détection est prévu dans **2026.1.12** (non publié au moment de la rédaction).

Checklist de correction :

1. Mettez à jour vers **2026.1.12** (ou lancez depuis la branche source `main`), puis redémarrez le Gateway.
2. Assurez-vous que MiniMax est configuré (assistant ou JSON), ou qu'une clé API MiniMax existe dans les variables d'environnement ou les profils d'authentification pour que le fournisseur puisse être injecté.
3. Utilisez l'identifiant de modèle exact (sensible à la casse) : `minimax/MiniMax-M2.1` ou `minimax/MiniMax-M2.1-lightning`.
4. Exécutez :

   ```bash
   openclaw models list
   ```

   et choisissez dans la liste (ou `/model list` dans le chat).

Voir [MiniMax](/providers/minimax) et [Modèles](/concepts/models).

### Puis-je utiliser MiniMax par défaut et OpenAI pour les tâches complexes

Oui. Utilisez **MiniMax comme défaut** et changez de modèle **par session** selon les besoins.
Les solutions de repli sont pour les **erreurs**, pas les « tâches difficiles », donc utilisez `/model` ou un agent séparé.

**Option A : changement par session**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" },
      },
    },
  },
}
```

Puis :

```
/model gpt
```

**Option B : agents séparés**

- Agent A par défaut : MiniMax
- Agent B par défaut : OpenAI
- Routez par agent ou utilisez `/agent` pour changer

Documentation : [Modèles](/concepts/models), [Routage multi-agent](/concepts/multi-agent), [MiniMax](/providers/minimax), [OpenAI](/providers/openai).

### Est-ce que opus, sonnet, gpt sont des raccourcis intégrés

Oui. OpenClaw fournit quelques raccourcis par défaut (appliqués uniquement quand le modèle existe dans `agents.defaults.models`) :

- `opus` → `anthropic/claude-opus-4-6`
- `sonnet` → `anthropic/claude-sonnet-4-5`
- `gpt` → `openai/gpt-5.2`
- `gpt-mini` → `openai/gpt-5-mini`
- `gemini` → `google/gemini-3-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`

Si vous définissez votre propre alias avec le même nom, votre valeur prévaut.

### Comment définir ou remplacer les raccourcis (alias) de modèles

Les alias proviennent de `agents.defaults.models.<modelId>.alias`. Exemple :

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" },
      },
    },
  },
}
```

Ensuite `/model sonnet` (ou `/<alias>` quand c'est supporté) résout vers cet identifiant de modèle.

### Comment ajouter des modèles d'autres fournisseurs comme OpenRouter ou ZAI

OpenRouter (paiement par token ; nombreux modèles) :

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} },
    },
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." },
}
```

Z.AI (modèles GLM) :

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
  env: { ZAI_API_KEY: "..." },
}
```

Si vous référencez un provider/model sans que la clé de fournisseur requise ne soit présente, vous obtiendrez une erreur d'authentification à l'exécution (ex : `No API key found for provider "zai"`).

**« No API key found for provider » après l'ajout d'un nouvel agent**

Cela signifie généralement que le **nouvel agent** a un magasin d'authentification vide. L'authentification est par agent et stockée dans :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Options de correction :

- Lancez `openclaw agents add <id>` et configurez l'authentification pendant l'assistant.
- Ou copiez `auth-profiles.json` du `agentDir` de l'agent principal vers le `agentDir` du nouvel agent.

Ne réutilisez **pas** le même `agentDir` entre agents : cela provoque des collisions d'authentification et de sessions.

## Basculement de modèle et « All models failed »

### Comment fonctionne le basculement

Le basculement se fait en deux étapes :

1. **Rotation de profil d'authentification** au sein du même fournisseur.
2. **Repli de modèle** vers le modèle suivant dans `agents.defaults.model.fallbacks`.

Des temps de refroidissement s'appliquent aux profils en échec (backoff exponentiel), permettant à OpenClaw de continuer à répondre même quand un fournisseur est limité en débit ou temporairement défaillant.

### Que signifie cette erreur

```
No credentials found for profile "anthropic:default"
```

Cela signifie que le système a tenté d'utiliser le profil d'authentification `anthropic:default`, mais n'a pas trouvé d'identifiants pour celui-ci dans le magasin d'authentification attendu.

### Checklist de correction pour « No credentials found for profile anthropic:default »

- **Confirmez où se trouvent les profils d'authentification** (nouveaux vs anciens chemins)
  - Actuel : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  - Ancien : `~/.openclaw/agent/*` (migré par `openclaw doctor`)
- **Confirmez que votre variable d'environnement est chargée par le Gateway**
  - Si vous définissez `ANTHROPIC_API_KEY` dans votre shell mais lancez le Gateway via systemd/launchd, il pourrait ne pas en hériter. Mettez-la dans `~/.openclaw/.env` ou activez `env.shellEnv`.
- **Assurez-vous de modifier le bon agent**
  - Les configurations multi-agents impliquent plusieurs fichiers `auth-profiles.json`.
- **Vérifiez le statut modèle/authentification**
  - Utilisez `openclaw models status` pour voir les modèles configurés et si les fournisseurs sont authentifiés.

**Checklist de correction pour « No credentials found for profile anthropic »**

Cela signifie que l'exécution est verrouillée sur un profil d'authentification Anthropic, mais le Gateway ne le trouve pas dans son magasin d'authentification.

- **Utilisez un setup-token**
  - Lancez `claude setup-token`, puis collez-le avec `openclaw models auth setup-token --provider anthropic`.
  - Si le token a été créé sur une autre machine, utilisez `openclaw models auth paste-token --provider anthropic`.
- **Si vous voulez utiliser une clé API à la place**
  - Mettez `ANTHROPIC_API_KEY` dans `~/.openclaw/.env` sur l'**hôte du Gateway**.
  - Effacez tout ordre épinglé qui force un profil manquant :

    ```bash
    openclaw models auth order clear --provider anthropic
    ```

- **Confirmez que vous exécutez les commandes sur l'hôte du Gateway**
  - En mode distant, les profils d'authentification se trouvent sur la machine du Gateway, pas sur votre laptop.

### Pourquoi a-t-il aussi essayé Google Gemini et échoué

Si votre configuration de modèle inclut Google Gemini comme repli (ou si vous avez basculé vers un raccourci Gemini), OpenClaw l'essaiera lors du repli de modèle. Si vous n'avez pas configuré les identifiants Google, vous verrez `No API key found for provider "google"`.

Correction : soit fournissez l'authentification Google, soit supprimez ou évitez les modèles Google dans `agents.defaults.model.fallbacks` / les alias pour que le repli ne route pas vers eux.

**Message « LLM request rejected : thinking signature required » (Google Antigravity)**

Cause : l'historique de session contient des **blocs de réflexion sans signatures** (souvent issus d'un flux interrompu ou partiel). Google Antigravity exige des signatures pour les blocs de réflexion.

Correction : OpenClaw supprime désormais les blocs de réflexion non signés pour Google Antigravity Claude. Si le problème persiste, démarrez une **nouvelle session** ou désactivez `/thinking off` pour cet agent.

## Profils d'authentification : définition et gestion

Voir aussi : [/concepts/oauth](/concepts/oauth) (flux OAuth, stockage de tokens, patterns multi-comptes)

### Qu'est-ce qu'un profil d'authentification

Un profil d'authentification est un enregistrement d'identifiant nommé (OAuth ou clé API) lié à un fournisseur. Les profils se trouvent dans :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

### Quels sont les identifiants de profil typiques

OpenClaw utilise des identifiants préfixés par le fournisseur, comme :

- `anthropic:default` (courant quand aucune identité e-mail n'existe)
- `anthropic:<email>` pour les identités OAuth
- Des identifiants personnalisés de votre choix (ex : `anthropic:work`)

### Puis-je contrôler quel profil d'authentification est essayé en premier

Oui. La configuration supporte des métadonnées optionnelles pour les profils et un ordre par fournisseur (`auth.order.<provider>`). Ceci ne stocke **pas** de secrets ; il associé des identifiants au fournisseur/mode et définit l'ordre de rotation.

OpenClaw peut temporairement ignorer un profil s'il est en **refroidissement** court (limités de débit/timeouts/échecs d'authentification) ou en état **désactivé** prolongé (facturation/crédits insuffisants). Pour inspecter cela, lancez `openclaw models status --json` et vérifiez `auth.unusableProfiles`. Réglage : `auth.cooldowns.billingBackoffHours*`.

Vous pouvez aussi définir un **ordre de remplacement par agent** (stocké dans le `auth-profiles.json` de cet agent) via le CLI :

```bash
# Par défaut, utilise l'agent par défaut configuré (omettez --agent)
openclaw models auth order get --provider anthropic

# Verrouiller la rotation sur un seul profil (essayer uniquement celui-ci)
openclaw models auth order set --provider anthropic anthropic:default

# Ou définir un ordre explicite (repli au sein du fournisseur)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# Effacer le remplacement (revenir à config auth.order / round-robin)
openclaw models auth order clear --provider anthropic
```

Pour cibler un agent spécifique :

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

### OAuth vs clé API : quelle est la différence

OpenClaw supporte les deux :

- **OAuth** exploite souvent l'accès par abonnement (quand applicable).
- **Les clés API** utilisent la facturation au token.

L'assistant supporte explicitement le setup-token Anthropic et l'OAuth Codex OpenAI, et peut stocker des clés API pour vous.

## Gateway : ports, « already running » et mode distant

### Quel port le Gateway utilise-t-il

`gateway.port` contrôle le port unique multiplexé pour WebSocket + HTTP (Interface de contrôle, hooks, etc.).

Ordre de priorité :

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > défaut 18789
```

### Pourquoi openclaw gateway status dit « Runtime running » mais « RPC probe failed »

Parce que « running » est la vue du **superviseur** (launchd/systemd/schtasks). La sonde RPC est le CLI qui se connecte réellement au WebSocket du Gateway et appelle `status`.

Utilisez `openclaw gateway status` et fiez-vous à ces lignes :

- `Probe target:` (l'URL que la sonde a réellement utilisée)
- `Listening:` (ce qui est effectivement lié sur le port)
- `Last gateway error:` (cause racine courante quand le processus est actif mais le port n'écoute pas)

### Pourquoi openclaw gateway status affiche « Config cli » et « Config service » différents

Vous modifiez un fichier de configuration pendant que le service en utilisé un autre (souvent un décalage `--profile` / `OPENCLAW_STATE_DIR`).

Correction :

```bash
openclaw gateway install --force
```

Lancez cela depuis le même `--profile` / environnement que vous voulez que le service utilise.

### Que signifie « another gateway instance is already listening »

OpenClaw impose un verrou d'exécution en liant l'écouteur WebSocket immédiatement au démarrage (par défaut `ws://127.0.0.1:18789`). Si la liaison échoue avec `EADDRINUSE`, il lève une `GatewayLockError` indiquant qu'une autre instance écoute déjà.

Correction : arrêtez l'autre instance, libérez le port, ou lancez avec `openclaw gateway --port <port>`.

### Comment exécuter OpenClaw en mode distant (le client se connecte à un Gateway distant)

Définissez `gateway.mode: "remote"` et pointez vers une URL WebSocket distante, optionnellement avec un token/mot de passe :

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Notes :

- `openclaw gateway` ne démarre que quand `gateway.mode` est `local` (ou si vous passez le flag de remplacement).
- L'application macOS surveille le fichier de configuration et change de mode en direct quand ces valeurs changent.

### L'interface de contrôle dit « unauthorized » ou se reconnecte en boucle. Que faire

Votre Gateway fonctionne avec l'authentification activée (`gateway.auth.*`), mais l'interface n'envoie pas le token/mot de passe correspondant.

Détails (issus du code) :

- L'interface de contrôle stocke le token dans le localStorage du navigateur sous la clé `openclaw.control.settings.v1`.

Correction :

- Le plus rapide : `openclaw dashboard` (affiche + copie l'URL du tableau de bord, tente de l'ouvrir ; affiche une astuce SSH si headless).
- Si vous n'avez pas encore de token : `openclaw doctor --generate-gateway-token`.
- Si distant, créez d'abord un tunnel : `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/`.
- Définissez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) sur l'hôte du Gateway.
- Dans les paramètres de l'interface de contrôle, collez le même token.
- Toujours bloqué ? Lancez `openclaw status --all` et suivez [Dépannage](/gateway/troubleshooting). Voir [Tableau de bord](/web/dashboard) pour les détails d'authentification.

### J'ai défini gateway.bind sur « tailnet » mais rien ne peut se lier, rien n'écoute

Le bind `tailnet` sélectionne une IP Tailscale depuis vos interfaces réseau (100.64.0.0/10). Si la machine n'est pas sur Tailscale (ou si l'interface est désactivée), il n'y a rien sur quoi se lier.

Correction :

- Démarrez Tailscale sur cet hôte (pour qu'il ait une adresse 100.x), ou
- Basculez vers `gateway.bind: "loopback"` / `"lan"`.

Note : `tailnet` est explicite. `auto` préfère le loopback ; utilisez `gateway.bind: "tailnet"` quand vous voulez un bind exclusivement tailnet.

### Puis-je exécuter plusieurs Gateways sur le même hôte

Généralement non : un seul Gateway peut gérer plusieurs canaux de messagerie et agents. N'utilisez plusieurs Gateways que si vous avez besoin de redondance (ex : bot de secours) ou d'isolation stricte.

Oui, mais vous devez isoler :

- `OPENCLAW_CONFIG_PATH` (configuration par instance)
- `OPENCLAW_STATE_DIR` (état par instance)
- `agents.defaults.workspace` (isolation de l'espace de travail)
- `gateway.port` (ports uniques)

Configuration rapide (recommandée) :

- Utilisez `openclaw --profile <name> ...` par instance (crée automatiquement `~/.openclaw-<name>`).
- Définissez un `gateway.port` unique dans chaque configuration de profil (ou passez `--port` pour les exécutions manuelles).
- Installez un service par profil : `openclaw --profile <name> gateway install`.

Les profils ajoutent aussi un suffixe aux noms de service (`ai.openclaw.<profile>` ; ancien format `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
Guide complet : [Gateways multiples](/gateway/multiple-gateways).

### Que signifie « invalid handshake code 1008 »

Le Gateway est un **serveur WebSocket**, et il attend que le tout premier message soit une trame `connect`. S'il reçoit autre chose, il ferme la connexion avec le **code 1008** (violation de politique).

Causes courantes :

- Vous avez ouvert l'URL **HTTP** dans un navigateur (`http://...`) au lieu d'un client WS.
- Vous avez utilisé le mauvais port ou chemin.
- Un proxy ou tunnel a retiré les en-têtes d'authentification ou envoyé une requête non-Gateway.

Corrections rapides :

1. Utilisez l'URL WS : `ws://<host>:18789` (ou `wss://...` si HTTPS).
2. N'ouvrez pas le port WS dans un onglet de navigateur normal.
3. Si l'authentification est activée, incluez le token/mot de passe dans la trame `connect`.

Si vous utilisez le CLI ou le TUI, l'URL devrait ressembler à :

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Détails du protocole : [Protocole du Gateway](/gateway/protocol).

## Journalisation et débogage

### Où se trouvent les logs

Logs fichiers (structurés) :

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Vous pouvez définir un chemin stable via `logging.file`. Le niveau de log fichier est contrôlé par `logging.level`. La verbosité de la console est contrôlée par `--verbose` et `logging.consoleLevel`.

Suivi de logs le plus rapide :

```bash
openclaw logs --follow
```

Logs du service/superviseur (quand le Gateway fonctionne via launchd/systemd) :

- macOS : `$OPENCLAW_STATE_DIR/logs/gateway.log` et `gateway.err.log` (par défaut : `~/.openclaw/logs/...` ; les profils utilisent `~/.openclaw-<profile>/logs/...`)
- Linux : `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Windows : `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

Voir [Dépannage](/gateway/troubleshooting#log-locations) pour plus de détails.

### Comment démarrer/arrêter/redémarrer le service Gateway

Utilisez les commandes d'aide du Gateway :

```bash
openclaw gateway status
openclaw gateway restart
```

Si vous exécutez le Gateway manuellement, `openclaw gateway --force` peut récupérer le port. Voir [Gateway](/gateway).

### J'ai fermé mon terminal sous Windows, comment redémarrer OpenClaw

Il y a **deux modes d'installation Windows** :

**1) WSL2 (recommandé) :** le Gateway fonctionne dans Linux.

Ouvrez PowerShell, entrez dans WSL, puis redémarrez :

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

Si vous n'avez jamais installé le service, démarrez-le en premier plan :

```bash
openclaw gateway run
```

**2) Windows natif (non recommandé) :** le Gateway fonctionne directement sous Windows.

Ouvrez PowerShell et lancez :

```powershell
openclaw gateway status
openclaw gateway restart
```

Si vous l'exécutez manuellement (pas de service), utilisez :

```powershell
openclaw gateway run
```

Documentation : [Windows (WSL2)](/platforms/windows), [Guide opérationnel du service Gateway](/gateway).

### Le Gateway fonctionne mais les réponses n'arrivent jamais. Que vérifier

Commencez par un balayage de santé rapide :

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

Causes courantes :

- L'authentification du modèle n'est pas chargée sur l'**hôte du Gateway** (vérifiez `models status`).
- L'appairage/la liste autorisée du canal bloque les réponses (vérifiez la configuration du canal + les logs).
- Le WebChat/Tableau de bord est ouvert sans le bon token.

Si vous êtes en mode distant, confirmez que le tunnel/la connexion Tailscale est activé et que le WebSocket du Gateway est joignable.

Documentation : [Canaux](/channels), [Dépannage](/gateway/troubleshooting), [Accès distant](/gateway/remote).

### « Disconnected from gateway » sans raison. Que faire

Cela signifie généralement que l'interface a perdu la connexion WebSocket. Vérifiez :

1. Le Gateway est-il en cours d'exécution ? `openclaw gateway status`
2. Le Gateway est-il sain ? `openclaw status`
3. L'interface a-t-elle le bon token ? `openclaw dashboard`
4. Si distant, le lien tunnel/Tailscale est-il actif ?

Puis suivez les logs :

```bash
openclaw logs --follow
```

Documentation : [Tableau de bord](/web/dashboard), [Accès distant](/gateway/remote), [Dépannage](/gateway/troubleshooting).

### setMyCommands Telegram échoue avec des erreurs réseau. Que vérifier

Commencez par les logs et le statut du canal :

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

Si vous êtes sur un VPS ou derrière un proxy, confirmez que le HTTPS sortant est autorisé et que le DNS fonctionne.
Si le Gateway est distant, assurez-vous de consulter les logs sur l'hôte du Gateway.

Documentation : [Telegram](/channels/telegram), [Dépannage des canaux](/channels/troubleshooting).

### Le TUI n'affiche rien. Que vérifier

Commencez par confirmer que le Gateway est joignable et que l'agent peut fonctionner :

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

Dans le TUI, utilisez `/status` pour voir l'état actuel. Si vous attendez des réponses dans un canal de chat, assurez-vous que la livraison est activée (`/deliver on`).

Documentation : [TUI](/web/tui), [Commandes slash](/tools/slash-commands).

### Comment arrêter complètement puis relancer le Gateway

Si vous avez installé le service :

```bash
openclaw gateway stop
openclaw gateway start
```

Ceci arrête/démarre le **service supervisé** (launchd sur macOS, systemd sur Linux).
Utilisez cela quand le Gateway fonctionne en arrière-plan comme démon.

Si vous l'exécutez en premier plan, arrêtez avec Ctrl-C, puis :

```bash
openclaw gateway run
```

Documentation : [Guide opérationnel du service Gateway](/gateway).

### Explication simple : openclaw gateway restart vs openclaw gateway

- `openclaw gateway restart` : redémarre le **service en arrière-plan** (launchd/systemd).
- `openclaw gateway` : exécute le Gateway **en premier plan** pour cette session de terminal.

Si vous avez installé le service, utilisez les commandes Gateway. Utilisez `openclaw gateway` quand vous voulez une exécution ponctuelle en premier plan.

### Quel est le moyen le plus rapide d'obtenir plus de détails quand quelque chose échoue

Démarrez le Gateway avec `--verbose` pour plus de détails dans la console. Puis inspectez le fichier de log pour les erreurs d'authentification de canal, de routage de modèle et de RPC.

## Médias et pièces jointes

### Mon Skill a généré une image/un PDF mais rien n'a été envoyé

Les pièces jointes sortantes de l'agent doivent inclure une ligne `MEDIA:<chemin-ou-url>` (sur sa propre ligne). Voir [Configuration de l'assistant OpenClaw](/start/openclaw) et [Envoi par agent](/tools/agent-send).

Envoi via CLI :

```bash
openclaw message send --target +15555550123 --message "Here you go" --media /path/to/file.png
```

Vérifiez aussi :

- Le canal cible supporte les médias sortants et n'est pas bloqué par des listes autorisées.
- Le fichier respecte les limités de taille du fournisseur (les images sont redimensionnées à 2048px maximum).

Voir [Images](/nodes/images).

## Sécurité et contrôle d'accès

### Est-il sûr d'exposer OpenClaw aux messages privés entrants

Traitez les messages privés entrants comme des entrées non fiables. Les paramètres par défaut sont conçus pour réduire les risques :

- Le comportement par défaut sur les canaux supportant les messages privés est l'**appairage** :
  - Les expéditeurs inconnus reçoivent un code d'appairage ; le bot ne traite pas leur message.
  - Approuvez avec : `openclaw pairing approve --channel <channel> [--account <id>] <code>`
  - Les demandes en attente sont plafonnées à **3 par canal** ; vérifiez `openclaw pairing list --channel <channel> [--account <id>]` si un code n'est pas arrivé.
- Ouvrir les messages privés publiquement nécessite un opt-in explicite (`dmPolicy: "open"` et liste autorisée `"*"`).

Lancez `openclaw doctor` pour détecter les politiques de messages privés risquées.

### L'injection de prompts ne concerne-t-elle que les bots publics

Non. L'injection de prompts concerne le **contenu non fiable**, pas seulement qui peut envoyer des messages privés au bot.
Si votre assistant lit du contenu externe (recherche/récupération web, pages du navigateur, e-mails, documents, pièces jointes, logs collés), ce contenu peut inclure des instructions qui tentent de détourner le modèle. Cela peut arriver même si **vous êtes le seul expéditeur**.

Le plus grand risque survient quand les outils sont activés : le modèle peut être trompé pour exfiltrer du contexte ou appeler des outils en votre nom. Réduisez le périmètre d'impact en :

- utilisant un agent « lecteur » en lecture seule ou sans outils pour résumer le contenu non fiable
- gardant `web_search` / `web_fetch` / `browser` désactivés pour les agents avec outils activés
- utilisant le bac à sable et des listes d'outils autorisés strictes

Détails : [Sécurité](/gateway/security).

### Mon bot devrait-il avoir son propre e-mail, compte GitHub ou numéro de téléphone

Oui, dans la plupart des configurations. Isoler le bot avec des comptes et numéros de téléphone séparés réduit le périmètre d'impact si quelque chose tourne mal. Cela facilite aussi la rotation des identifiants ou la révocation d'accès sans impacter vos comptes personnels.

Commencez petit. Donnez accès uniquement aux outils et comptes dont vous avez réellement besoin, et élargissez plus tard si nécessaire.

Documentation : [Sécurité](/gateway/security), [Appairage](/channels/pairing).

### Puis-je lui donner l'autonomie sur mes SMS et est-ce sûr

Nous **déconseillons** l'autonomie totale sur vos messages personnels. Le pattern le plus sûr est :

- Garder les messages privés en **mode appairage** ou avec une liste autorisée restreinte.
- Utiliser un **numéro ou compte séparé** si vous voulez qu'il envoie des messages en votre nom.
- Le laisser rédiger, puis **approuver avant l'envoi**.

Si vous voulez expérimenter, faites-le sur un compte dédié et gardez-le isolé. Voir [Sécurité](/gateway/security).

### Puis-je utiliser des modèles moins chers pour les tâches d'assistant personnel

Oui, **si** l'agent est uniquement en mode chat et que l'entrée est fiable. Les modèles de niveau inférieur sont plus susceptibles au détournement d'instructions, donc évitez-les pour les agents avec outils activés ou lors de la lecture de contenu non fiable. Si vous devez utiliser un modèle plus petit, verrouillez les outils et exécutez dans un bac à sable. Voir [Sécurité](/gateway/security).

### J'ai lancé /start dans Telegram mais je n'ai pas reçu de code d'appairage

Les codes d'appairage sont envoyés **uniquement** quand un expéditeur inconnu envoie un message au bot et que `dmPolicy: "pairing"` est activé. `/start` seul ne génère pas de code.

Vérifiez les demandes en attente :

```bash
openclaw pairing list telegram
```

Si vous voulez un accès immédiat, ajoutez votre identifiant d'expéditeur à la liste autorisée ou définissez `dmPolicy: "open"` pour ce compte.

### WhatsApp : va-t-il contacter mes contacts ? Comment fonctionne l'appairage

Non. La politique de messages privés WhatsApp par défaut est l'**appairage**. Les expéditeurs inconnus reçoivent uniquement un code d'appairage et leur message n'est **pas traité**. OpenClaw ne répond qu'aux conversations reçues ou aux envois explicites que vous déclenchez.

Approuvez l'appairage avec :

```bash
openclaw pairing approve whatsapp <code>
```

Listez les demandes en attente :

```bash
openclaw pairing list whatsapp
```

Question sur le numéro de téléphone de l'assistant : il sert à définir votre **liste autorisée/propriétaire** pour que vos propres messages privés soient autorisés. Il n'est pas utilisé pour l'envoi automatique. Si vous utilisez votre numéro WhatsApp personnel, utilisez ce numéro et activez `channels.whatsapp.selfChatMode`.

## Commandes de chat, interruption de tâches et « ça ne s'arrête pas »

### Comment empêcher les messages système internes d'apparaître dans le chat

La plupart des messages internes ou d'outils n'apparaissent que quand **verbose** ou **reasoning** est activé pour cette session.

Correction dans le chat concerné :

```
/verbose off
/reasoning off
```

Si c'est toujours bruyant, vérifiez les paramètres de session dans l'interface de contrôle et définissez verbose sur **inherit**. Confirmez aussi que vous n'utilisez pas un profil de bot avec `verboseDefault` défini sur `on` dans la configuration.

Documentation : [Réflexion et verbosité](/tools/thinking), [Sécurité](/gateway/security#reasoning--verbose-output-in-groups).

### Comment arrêter/annuler une tâche en cours

Envoyez n'importe lequel de ces messages **comme message autonome** (sans slash) :

```
stop
stop action
stop current action
stop run
stop current run
stop agent
stop the agent
stop openclaw
openclaw stop
stop don't do anything
stop do not do anything
stop doing anything
please stop
stop please
abort
esc
wait
exit
interrupt
```

Ce sont des déclencheurs d'interruption (pas des commandes slash).

Pour les processus en arrière-plan (lancés par l'outil exec), vous pouvez demander à l'agent d'exécuter :

```
process action:kill sessionId:XXX
```

Vue d'ensemble des commandes slash : voir [Commandes slash](/tools/slash-commands).

La plupart des commandes doivent être envoyées comme **message autonome** commençant par `/`, mais quelques raccourcis (comme `/status`) fonctionnent aussi en ligne pour les expéditeurs de la liste autorisée.

### Comment envoyer un message Discord depuis Telegram ? « Cross-context messaging denied »

OpenClaw bloque la messagerie **inter-fournisseur** par défaut. Si un appel d'outil est lié à Telegram, il n'enverra pas vers Discord sauf si vous l'autorisez explicitement.

Activez la messagerie inter-fournisseur pour l'agent :

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " },
          },
        },
      },
    },
  },
}
```

Redémarrez le Gateway après avoir modifié la configuration. Si vous ne voulez cela que pour un seul agent, définissez-le sous `agents.list[].tools.message` à la place.

### Pourquoi ai-je l'impression que le bot ignore les messages envoyés rapidement

Le mode de file d'attente contrôle comment les nouveaux messages interagissent avec une exécution en cours. Utilisez `/queue` pour changer de mode :

- `steer` : les nouveaux messages redirigent la tâche en cours
- `followup` : traiter les messages un par un
- `collect` : regrouper les messages et répondre une seule fois (défaut)
- `steer-backlog` : rediriger maintenant, puis traiter le backlog
- `interrupt` : interrompre l'exécution en cours et repartir de zéro

Vous pouvez ajouter des options comme `debounce:2s cap:25 drop:summarize` pour les modes followup.

## Répondre à la question exacte depuis la capture d'écran / le journal de chat

**Q : « Quel est le modèle par défaut pour Anthropic avec une clé API ? »**

**R :** Dans OpenClaw, les identifiants et la sélection de modèle sont séparés. Définir `ANTHROPIC_API_KEY` (ou stocker une clé API Anthropic dans les profils d'authentification) activé l'authentification, mais le modèle par défaut réel est celui que vous configurez dans `agents.defaults.model.primary` (par exemple, `anthropic/claude-sonnet-4-5` ou `anthropic/claude-opus-4-6`). Si vous voyez `No credentials found for profile "anthropic:default"`, cela signifie que le Gateway n'a pas trouvé d'identifiants Anthropic dans le `auth-profiles.json` attendu pour l'agent en cours d'exécution.

---

Toujours bloqué ? Posez votre question sur [Discord](https://discord.com/invite/clawd) ou ouvrez une [discussion GitHub](https://github.com/openclaw/openclaw/discussions).
