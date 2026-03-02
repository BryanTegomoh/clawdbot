---
summary: "Référence CLI OpenClaw pour les commandes `openclaw`, sous-commandes et options"
read_when:
  - Ajout ou modification de commandes ou options CLI
  - Documentation de nouvelles surfaces de commandes
title: "Référence CLI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/index.md
  workflow: manual
---

# Référence CLI

Cette page décrit le comportement actuel du CLI. Si les commandes changent, mettez à jour ce document.

## Pages de commandes

- [`setup`](/cli/setup)
- [`onboard`](/cli/onboard)
- [`configuré`](/cli/configure)
- [`config`](/cli/config)
- [`completion`](/cli/completion)
- [`doctor`](/cli/doctor)
- [`dashboard`](/cli/dashboard)
- [`reset`](/cli/reset)
- [`uninstall`](/cli/uninstall)
- [`update`](/cli/update)
- [`message`](/cli/message)
- [`agent`](/cli/agent)
- [`agents`](/cli/agents)
- [`acp`](/cli/acp)
- [`status`](/cli/status)
- [`health`](/cli/health)
- [`sessions`](/cli/sessions)
- [`gateway`](/cli/gateway)
- [`logs`](/cli/logs)
- [`system`](/cli/system)
- [`models`](/cli/models)
- [`memory`](/cli/memory)
- [`directory`](/cli/directory)
- [`nodes`](/cli/nodes)
- [`devices`](/cli/devices)
- [`node`](/cli/node)
- [`approvals`](/cli/approvals)
- [`sandbox`](/cli/sandbox)
- [`tui`](/cli/tui)
- [`browser`](/cli/browser)
- [`cron`](/cli/cron)
- [`dns`](/cli/dns)
- [`docs`](/cli/docs)
- [`hooks`](/cli/hooks)
- [`webhooks`](/cli/webhooks)
- [`pairing`](/cli/pairing)
- [`qr`](/cli/qr)
- [`plugins`](/cli/plugins) (commandes de plugins)
- [`channels`](/cli/channels)
- [`security`](/cli/security)
- [`secrets`](/cli/secrets)
- [`skills`](/cli/skills)
- [`daemon`](/cli/daemon) (alias historique pour les commandes du service Gateway)
- [`clawbot`](/cli/clawbot) (espace de noms d'alias historique)
- [`voicecall`](/cli/voicecall) (plugin ; si installé)

## Options globales

- `--dev` : isole l'état sous `~/.openclaw-dev` et décale les ports par défaut.
- `--profile <name>` : isole l'état sous `~/.openclaw-<name>`.
- `--no-color` : désactivé les couleurs ANSI.
- `--update` : raccourci pour `openclaw update` (installations depuis les sources uniquement).
- `-V`, `--version`, `-v` : affiche la version et quitte.

## Style de sortie

- Les couleurs ANSI et les indicateurs de progression ne s'affichent que dans les sessions TTY.
- Les hyperliens OSC-8 s'affichent comme des liens cliquables dans les terminaux compatibles ; sinon, les URL en texte brut sont utilisées.
- `--json` (et `--plain` lorsque supporté) désactive le style pour une sortie propre.
- `--no-color` désactive le style ANSI ; `NO_COLOR=1` est également respecté.
- Les commandes longues affichent un indicateur de progression (OSC 9;4 lorsque supporté).

## Palette de couleurs

OpenClaw utilise une palette lobster pour la sortie CLI.

- `accent` (#FF5A2D) : titres, étiquettes, surbrillances principales.
- `accentBright` (#FF7A3D) : noms de commandes, emphase.
- `accentDim` (#D14A22) : texte de surbrillancé secondaire.
- `info` (#FF8A5B) : valeurs informatives.
- `success` (#2FBF71) : états de succès.
- `warn` (#FFB020) : avertissements, replis, attention.
- `error` (#E23D2D) : erreurs, échecs.
- `muted` (#8B7F77) : attenuation, metadonnées.

Source de verite de la palette : `src/terminal/palette.ts` (alias "lobster seam").

## Arbre des commandes

```
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

Note : les plugins peuvent ajouter des commandes supplémentaires de niveau supérieur (par exemple `openclaw voicecall`).

## Sécurité

- `openclaw security audit` : audite la configuration et l'état local pour les erreurs de sécurité courantes.
- `openclaw security audit --deep` : sonde le Gateway en direct (au mieux).
- `openclaw security audit --fix` : renforce les paramètres par défaut et ajuste les permissions chmod de l'état/configuration.

## Secrets

- `openclaw secrets reload` : re-résout les refs et échange atomiquement l'instantané d'exécution.
- `openclaw secrets audit` : recherche les résidus en clair, les refs non résolues et les dérives de priorité.
- `openclaw secrets configuré` : assistant interactif pour la configuration des fournisseurs + le mapping SecretRef + preflight/apply.
- `openclaw secrets apply --from <plan.json>` : applique un plan précédemment génère (`--dry-run` supporté).

## Plugins

Gérez les extensions et leur configuration :

- `openclaw plugins list` : découvrir les plugins (utilisez `--json` pour une sortie machine).
- `openclaw plugins info <id>` : afficher les détails d'un plugin.
- `openclaw plugins install <path|.tgz|npm-spec>` : installer un plugin (ou ajouter un chemin de plugin a `plugins.load.paths`).
- `openclaw plugins enable <id>` / `disable <id>` : basculer `plugins.entries.<id>.enabled`.
- `openclaw plugins doctor` : signaler les erreurs de chargement des plugins.

La plupart des modifications de plugins nécessitent un redémarrage du Gateway. Voir [/plugin](/tools/plugin).

## Mémoire

Recherche vectorielle sur `MEMORY.md` + `memory/*.md` :

- `openclaw memory status` : afficher les statistiques de l'index.
- `openclaw memory index` : reindexer les fichiers mémoire.
- `openclaw memory search "<query>"` (ou `--query "<query>"`) : recherche semantique dans la mémoire.

## Commandes slash du chat

Les messages du chat supportent les commandes `/...` (texte et natives). Voir [/tools/slash-commands](/tools/slash-commands).

Points clés :

- `/status` pour un diagnostic rapide.
- `/config` pour les modifications de configuration persistées.
- `/debug` pour les surcharges de configuration en temps d'exécution uniquement (mémoire, pas disque ; nécessite `commands.debug: true`).

## Configuration + intégration

### `setup`

Initialisé la configuration et l'espace de travail.

Options :

- `--workspace <dir>` : chemin de l'espace de travail de l'agent (défaut `~/.openclaw/workspace`).
- `--wizard` : lancé l'assistant d'intégration.
- `--non-interactive` : exécute l'assistant sans invites.
- `--mode <local|remote>` : mode de l'assistant.
- `--remote-url <url>` : URL du Gateway distant.
- `--remote-token <token>` : jeton du Gateway distant.

L'assistant se lance automatiquement lorsqu'une option d'assistant est présente (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### `onboard`

Assistant interactif pour configurer le Gateway, l'espace de travail et les compétences.

Options :

- `--workspace <dir>`
- `--reset` (réinitialisé la configuration + identifiants + sessions avant l'assistant)
- `--reset-scope <config|config+creds+sessions|full>` (défaut `config+creds+sessions` ; utilisez `full` pour supprimer également l'espace de travail)
- `--non-interactive`
- `--mode <local|remote>`
- `--flow <quickstart|advanced|manual>` (manual est un alias de advanced)
- `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
- `--token-provider <id>` (non interactif ; utilisé avec `--auth-choice token`)
- `--token <token>` (non interactif ; utilisé avec `--auth-choice token`)
- `--token-profile-id <id>` (non interactif ; défaut : `<provider>:manual`)
- `--token-expires-in <duration>` (non interactif ; ex. `365d`, `12h`)
- `--secret-input-mode <plaintext|ref>` (défaut `plaintext` ; utilisez `ref` pour stocker des refs env par défaut du fournisseur au lieu de clés en clair)
- `--anthropic-api-key <key>`
- `--openai-api-key <key>`
- `--mistral-api-key <key>`
- `--openrouter-api-key <key>`
- `--ai-gateway-api-key <key>`
- `--moonshot-api-key <key>`
- `--kimi-code-api-key <key>`
- `--gemini-api-key <key>`
- `--zai-api-key <key>`
- `--minimax-api-key <key>`
- `--opencode-zen-api-key <key>`
- `--custom-base-url <url>` (non interactif ; utilisé avec `--auth-choice custom-api-key`)
- `--custom-model-id <id>` (non interactif ; utilisé avec `--auth-choice custom-api-key`)
- `--custom-api-key <key>` (non interactif ; optionnel ; utilisé avec `--auth-choice custom-api-key` ; se replie sur `CUSTOM_API_KEY` si omis)
- `--custom-provider-id <id>` (non interactif ; id de fournisseur personnalisé optionnel)
- `--custom-compatibility <openai|anthropic>` (non interactif ; optionnel ; défaut `openai`)
- `--gateway-port <port>`
- `--gateway-bind <loopback|lan|tailnet|auto|custom>`
- `--gateway-auth <token|password>`
- `--gateway-token <token>`
- `--gateway-password <password>`
- `--remote-url <url>`
- `--remote-token <token>`
- `--tailscale <off|serve|funnel>`
- `--tailscale-reset-on-exit`
- `--install-daemon`
- `--no-install-daemon` (alias : `--skip-daemon`)
- `--daemon-runtime <node|bun>`
- `--skip-channels`
- `--skip-skills`
- `--skip-health`
- `--skip-ui`
- `--node-manager <npm|pnpm|bun>` (pnpm recommandé ; bun non recommandé pour le runtime Gateway)
- `--json`

### `configuré`

Assistant de configuration interactif (modèles, canaux, compétences, Gateway).

### `config`

Utilitaires de configuration non interactifs (get/set/unset). Exécuter `openclaw config` sans sous-commande lancé l'assistant.

Sous-commandes :

- `config get <path>` : afficher une valeur de configuration (chemin point/crochet).
- `config set <path> <value>` : définir une valeur (JSON5 ou chaine brute).
- `config unset <path>` : supprimer une valeur.

### `doctor`

Vérifications de santé + corrections rapides (configuration + Gateway + services historiques).

Options :

- `--no-workspace-suggestions` : désactivé les suggestions de mémoire de l'espace de travail.
- `--yes` : accepte les valeurs par défaut sans invites (mode headless).
- `--non-interactive` : ignoré les invites ; applique uniquement les migrations sures.
- `--deep` : analysé les services système pour les installations de Gateway supplémentaires.

## Utilitaires de canaux

### `channels`

Gérez les comptes de canaux de chat (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams).

Sous-commandes :

- `channels list` : afficher les canaux configurés et les profils d'authentification.
- `channels status` : vérifier l'accessibilité du Gateway et la santé des canaux (`--probe` exécute des vérifications supplémentaires ; utilisez `openclaw health` ou `openclaw status --deep` pour les sondes de santé du Gateway).
- Astuce : `channels status` affiche des avertissements avec des corrections suggérées lorsqu'il peut détecter des erreurs de configuration courantes (puis vous dirige vers `openclaw doctor`).
- `channels logs` : afficher les logs récents des canaux depuis le fichier log du Gateway.
- `channels add` : configuration de type assistant lorsqu'aucune option n'est passée ; les options passent en mode non interactif.
- `channels remove` : désactivé par défaut ; passez `--delete` pour supprimer les entrées de configuration sans invites.
- `channels login` : connexion interactive au canal (WhatsApp Web uniquement).
- `channels logout` : déconnexion d'une session de canal (si supporté).

Options communes :

- `--channel <name>` : `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
- `--account <id>` : identifiant du compte de canal (défaut `default`)
- `--name <label>` : nom d'affichage pour le compte

Options de `channels login` :

- `--channel <channel>` (défaut `whatsapp` ; supporte `whatsapp`/`web`)
- `--account <id>`
- `--verbose`

Options de `channels logout` :

- `--channel <channel>` (défaut `whatsapp`)
- `--account <id>`

Options de `channels list` :

- `--no-usage` : ignoré les instantanés d'utilisation/quota du fournisseur de modèles (OAuth/API uniquement).
- `--json` : sortie JSON (inclut l'utilisation sauf si `--no-usage` est défini).

Options de `channels logs` :

- `--channel <name|all>` (défaut `all`)
- `--lines <n>` (défaut `200`)
- `--json`

Plus de détails : [/concepts/oauth](/concepts/oauth)

Exemples :

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### `skills`

Listez et inspectez les compétences disponibles ainsi que les informations de préparation.

Sous-commandes :

- `skills list` : lister les compétences (défaut lorsqu'aucune sous-commande).
- `skills info <name>` : afficher les détails d'une competence.
- `skills check` : résumé des compétences prêtes vs exigences manquantes.

Options :

- `--eligible` : afficher uniquement les compétences prêtes.
- `--json` : sortie JSON (sans style).
- `-v`, `--verbose` : inclure le detail des exigences manquantes.

Astuce : utilisez `npx clawhub` pour rechercher, installer et synchroniser les compétences.

### `pairing`

Approuver les demandes d'appairage DM entre les canaux.

Sous-commandes :

- `pairing list [channel] [--channel <channel>] [--account <id>] [--json]`
- `pairing approve <channel> <code> [--account <id>] [--notify]`
- `pairing approve --channel <channel> [--account <id>] <code> [--notify]`

### `devices`

Gérez les entrées d'appairage des appareils du Gateway et les jetons d'appareil par rôle.

Sous-commandes :

- `devices list [--json]`
- `devices approve [requestId] [--latest]`
- `devices reject <requestId>`
- `devices remove <deviceId>`
- `devices clear --yes [--pending]`
- `devices rotate --device <id> --rôle <rôle> [--scope <scope...>]`
- `devices revoke --device <id> --rôle <rôle>`

### `webhooks gmail`

Configuration et exécution du hook Gmail Pub/Sub. Voir [/automation/gmail-pubsub](/automation/gmail-pubsub).

Sous-commandes :

- `webhooks gmail setup` (nécessite `--account <email>` ; supporte `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
- `webhooks gmail run` (surcharges d'exécution pour les mêmes options)

### `dns setup`

Utilitaire DNS de découverte étendue (CoreDNS + Tailscale). Voir [/gateway/discovery](/gateway/discovery).

Options :

- `--apply` : installe/met à jour la configuration CoreDNS (nécessite sudo ; macOS uniquement).

## Messagerie + agent

### `message`

Messagerie sortante unifiée + actions de canal.

Voir : [/cli/message](/cli/message)

Sous-commandes :

- `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
- `message thread <create|list|reply>`
- `message emoji <list|upload>`
- `message sticker <send|upload>`
- `message rôle <info|add|remove>`
- `message channel <info|list>`
- `message member info`
- `message voice status`
- `message event <list|create>`

Exemples :

- `openclaw message send --target +15555550123 --message "Hi"`
- `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

### `agent`

Exécuter un tour d'agent via le Gateway (ou `--local` en mode intégré).

Requis :

- `--message <text>`

Options :

- `--to <dest>` (pour la clé de session et la livraison optionnelle)
- `--session-id <id>`
- `--thinking <off|minimal|low|medium|high|xhigh>` (modèles GPT-5.2 + Codex uniquement)
- `--verbose <on|full|off>`
- `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
- `--local`
- `--deliver`
- `--json`
- `--timeout <seconds>`

### `agents`

Gérez les agents isolés (espaces de travail + authentification + routage).

#### `agents list`

Lister les agents configurés.

Options :

- `--json`
- `--bindings`

#### `agents add [name]`

Ajouter un nouvel agent isolé. Lance l'assistant guidé sauf si des options (ou `--non-interactive`) sont passées ; `--workspace` est requis en mode non interactif.

Options :

- `--workspace <dir>`
- `--model <id>`
- `--agent-dir <dir>`
- `--bind <channel[:accountId]>` (repetable)
- `--non-interactive`
- `--json`

Les spécifications de liaison utilisent `channel[:accountId]`. Lorsque `accountId` est omis pour WhatsApp, l'identifiant de compte par défaut est utilisé.

#### `agents delete <id>`

Supprimer un agent et nettoyer son espace de travail + état.

Options :

- `--force`
- `--json`

### `acp`

Exécuter le pont ACP qui connecte les IDE au Gateway.

Voir [`acp`](/cli/acp) pour toutes les options et exemples.

### `status`

Afficher la santé de la session liée et les destinataires récents.

Options :

- `--json`
- `--all` (diagnostic complet ; lecture seule, copiable)
- `--deep` (sonder les canaux)
- `--usage` (afficher l'utilisation/quota du fournisseur de modèles)
- `--timeout <ms>`
- `--verbose`
- `--debug` (alias de `--verbose`)

Notes :

- L'aperçu inclut l'état du service Gateway + node host lorsque disponible.

### Suivi d'utilisation

OpenClaw peut afficher l'utilisation/quota du fournisseur lorsque des identifiants OAuth/API sont disponibles.

Surfaces :

- `/status` (ajouté une ligne courte d'utilisation du fournisseur lorsque disponible)
- `openclaw status --usage` (affiche la ventilation complète par fournisseur)
- Barre de menu macOS (section Utilisation sous Contexte)

Notes :

- Les données proviennent directement des endpoints d'utilisation du fournisseur (pas d'estimations).
- Fournisseurs : Anthropic, GitHub Copilot, OpenAI Codex OAuth, ainsi que Gemini CLI/Antigravity lorsque ces plugins fournisseurs sont activés.
- Si aucun identifiant correspondant n'existe, l'utilisation est masquée.
- Details : voir [Suivi d'utilisation](/concepts/usage-tracking).

### `health`

Récupérer la santé du Gateway en cours d'exécution.

Options :

- `--json`
- `--timeout <ms>`
- `--verbose`

### `sessions`

Lister les sessions de conversation stockées.

Options :

- `--json`
- `--verbose`
- `--store <path>`
- `--activated <minutes>`

## Réinitialisation / Desinstallation

### `reset`

Réinitialiser la configuration/l'état local (garde le CLI installé).

Options :

- `--scope <config|config+creds+sessions|full>`
- `--yes`
- `--non-interactive`
- `--dry-run`

Notes :

- `--non-interactive` nécessite `--scope` et `--yes`.

### `uninstall`

Désinstaller le service Gateway + les données locales (le CLI reste).

Options :

- `--service`
- `--state`
- `--workspace`
- `--app`
- `--all`
- `--yes`
- `--non-interactive`
- `--dry-run`

Notes :

- `--non-interactive` nécessite `--yes` et des portées explicites (ou `--all`).

## Gateway

### `gateway`

Exécuter le Gateway WebSocket.

Options :

- `--port <port>`
- `--bind <loopback|tailnet|lan|auto|custom>`
- `--token <token>`
- `--auth <token|password>`
- `--password <password>`
- `--tailscale <off|serve|funnel>`
- `--tailscale-reset-on-exit`
- `--allow-unconfigured`
- `--dev`
- `--reset` (réinitialiser la configuration dev + identifiants + sessions + espace de travail)
- `--force` (tuer l'écouteur existant sur le port)
- `--verbose`
- `--claude-cli-logs`
- `--ws-log <auto|full|compact>`
- `--compact` (alias de `--ws-log compact`)
- `--raw-stream`
- `--raw-stream-path <path>`

### `gateway service`

Gérer le service Gateway (launchd/systemd/schtasks).

Sous-commandes :

- `gateway status` (sonde le Gateway RPC par défaut)
- `gateway install` (installation du service)
- `gateway uninstall`
- `gateway start`
- `gateway stop`
- `gateway restart`

Notes :

- `gateway status` sonde le Gateway RPC par défaut en utilisant le port/la configuration résolus du service (surcharger avec `--url/--token/--password`).
- `gateway status` supporte `--no-probe`, `--deep` et `--json` pour le scripting.
- `gateway status` affiche également les services Gateway historiques ou supplémentaires lorsqu'il peut les détecter (`--deep` ajouté des analyses au niveau système). Les services OpenClaw nommés par profil sont traités comme de première classe et ne sont pas signalés comme « supplémentaires ».
- `gateway status` affiche quel chemin de configuration le CLI utilisé vs quel configuration le service utilise probablement (env du service), plus l'URL cible de la sonde résolue.
- `gateway install|uninstall|start|stop|restart` supportent `--json` pour le scripting (la sortie par défaut reste conviviale).
- `gateway install` utilisé le runtime Node par défaut ; bun n'est **pas recommandé** (bugs WhatsApp/Telegram).
- Options de `gateway install` : `--port`, `--runtime`, `--token`, `--force`, `--json`.

### `logs`

Suivre les logs du fichier Gateway via RPC.

Notes :

- Les sessions TTY affichent une vue colorisée et structurée ; les sessions non-TTY se replient sur du texte brut.
- `--json` émet du JSON delimitée par lignes (un événement de log par ligne).

Exemples :

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### `gateway <subcommand>`

Utilitaires CLI du Gateway (utilisez `--url`, `--token`, `--password`, `--timeout`, `--expect-final` pour les sous-commandes RPC).
Lorsque vous passez `--url`, le CLI n'applique pas automatiquement les identifiants de la configuration ou de l'environnement.
Incluez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

Sous-commandes :

- `gateway call <method> [--params <json>]`
- `gateway health`
- `gateway status`
- `gateway probe`
- `gateway discover`
- `gateway install|uninstall|start|stop|restart`
- `gateway run`

RPC courants :

- `config.apply` (valider + écrire la configuration + redémarrer + réveiller)
- `config.patch` (fusionner une mise à jour partielle + redémarrer + réveiller)
- `update.run` (exécuter la mise à jour + redémarrer + réveiller)

Astuce : lorsque vous appelez `config.set`/`config.apply`/`config.patch` directement, passez `baseHash` depuis `config.get` si une configuration existe déjà.

## Modèles

Voir [/concepts/models](/concepts/models) pour le comportement de repli et la stratégie d'analyse.

Authentification Anthropic préférée (setup-token) :

```bash
claude setup-token
openclaw models auth setup-token --provider anthropic
openclaw models status
```

### `models` (racine)

`openclaw models` est un alias de `models status`.

Options racine :

- `--status-json` (alias de `models status --json`)
- `--status-plain` (alias de `models status --plain`)

### `models list`

Options :

- `--all`
- `--local`
- `--provider <name>`
- `--json`
- `--plain`

### `models status`

Options :

- `--json`
- `--plain`
- `--check` (sortie 1=expire/manquant, 2=en cours d'expiration)
- `--probe` (sonde en direct des profils d'authentification configurés)
- `--probe-provider <name>`
- `--probe-profile <id>` (répéter ou séparer par virgules)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`

Inclut toujours l'aperçu d'authentification et l'état d'expiration OAuth pour les profils du magasin d'authentification.
`--probe` exécute des requêtes en direct (peut consommer des jetons et déclencher des limités de taux).

### `models set <model>`

Définir `agents.defaults.model.primary`.

### `models set-image <model>`

Définir `agents.defaults.imageModel.primary`.

### `models aliases list|add|remove`

Options :

- `list` : `--json`, `--plain`
- `add <alias> <model>`
- `remove <alias>`

### `models fallbacks list|add|remove|clear`

Options :

- `list` : `--json`, `--plain`
- `add <model>`
- `remove <model>`
- `clear`

### `models image-fallbacks list|add|remove|clear`

Options :

- `list` : `--json`, `--plain`
- `add <model>`
- `remove <model>`
- `clear`

### `models scan`

Options :

- `--min-params <b>`
- `--max-age-days <days>`
- `--provider <name>`
- `--max-candidates <n>`
- `--timeout <ms>`
- `--concurrency <n>`
- `--no-probe`
- `--yes`
- `--no-input`
- `--set-default`
- `--set-image`
- `--json`

### `models auth add|setup-token|paste-token`

Options :

- `add` : utilitaire d'authentification interactif
- `setup-token` : `--provider <name>` (défaut `anthropic`), `--yes`
- `paste-token` : `--provider <name>`, `--profile-id <id>`, `--expires-in <duration>`

### `models auth order get|set|clear`

Options :

- `get` : `--provider <name>`, `--agent <id>`, `--json`
- `set` : `--provider <name>`, `--agent <id>`, `<profileIds...>`
- `clear` : `--provider <name>`, `--agent <id>`

## Système

### `system event`

Mettre en file d'attente un événement système et optionnellement déclencher un heartbeat (Gateway RPC).

Requis :

- `--text <text>`

Options :

- `--mode <now|next-heartbeat>`
- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

### `system heartbeat last|enable|disable`

Contrôles du heartbeat (Gateway RPC).

Options :

- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

### `system presence`

Lister les entrées de presence système (Gateway RPC).

Options :

- `--json`
- `--url`, `--token`, `--timeout`, `--expect-final`

## Cron

Gérer les tâches planifiées (Gateway RPC). Voir [/automation/cron-jobs](/automation/cron-jobs).

Sous-commandes :

- `cron status [--json]`
- `cron list [--all] [--json]` (sortie en tableau par défaut ; utilisez `--json` pour le brut)
- `cron add` (alias : `create` ; nécessite `--name` et exactement un parmi `--at` | `--every` | `--cron`, et exactement une charge utile parmi `--system-event` | `--message`)
- `cron edit <id>` (modifier des champs)
- `cron rm <id>` (alias : `remove`, `delete`)
- `cron enable <id>`
- `cron disable <id>`
- `cron runs --id <id> [--limit <n>]`
- `cron run <id> [--force]`

Toutes les commandes `cron` acceptent `--url`, `--token`, `--timeout`, `--expect-final`.

## Node host

`node` exécute un **node host headless** ou le géré comme service en arrière-plan. Voir [`openclaw node`](/cli/node).

Sous-commandes :

- `node run --host <gateway-host> --port 18789`
- `node status`
- `node install [--host <gateway-host>] [--port <port>] [--tls] [--tls-fingerprint <sha256>] [--node-id <id>] [--display-name <name>] [--runtime <node|bun>] [--force]`
- `node uninstall`
- `node stop`
- `node restart`

## Nodes

`nodes` communique avec le Gateway et cible les nodes appairés. Voir [/nodes](/nodes).

Options communes :

- `--url`, `--token`, `--timeout`, `--json`

Sous-commandes :

- `nodes status [--connected] [--last-connected <duration>]`
- `nodes describe --node <id|name|ip>`
- `nodes list [--connected] [--last-connected <duration>]`
- `nodes pending`
- `nodes approve <requestId>`
- `nodes reject <requestId>`
- `nodes rename --node <id|name|ip> --name <displayName>`
- `nodes invoke --node <id|name|ip> --command <command> [--params <json>] [--invoke-timeout <ms>] [--idempotency-key <key>]`
- `nodes run --node <id|name|ip> [--cwd <path>] [--env KEY=VAL] [--command-timeout <ms>] [--needs-screen-recording] [--invoke-timeout <ms>] <command...>` (application Mac ou node host headless)
- `nodes notify --node <id|name|ip> [--title <text>] [--body <text>] [--sound <name>] [--priority <passive|activé|timeSensitive>] [--delivery <system|overlay|auto>] [--invoke-timeout <ms>]` (Mac uniquement)

Camera :

- `nodes camera list --node <id|name|ip>`
- `nodes camera snap --node <id|name|ip> [--facing front|back|both] [--device-id <id>] [--max-width <px>] [--quality <0-1>] [--delay-ms <ms>] [--invoke-timeout <ms>]`
- `nodes camera clip --node <id|name|ip> [--facing front|back] [--device-id <id>] [--duration <ms|10s|1m>] [--no-audio] [--invoke-timeout <ms>]`

Canvas + écran :

- `nodes canvas snapshot --node <id|name|ip> [--format png|jpg|jpeg] [--max-width <px>] [--quality <0-1>] [--invoke-timeout <ms>]`
- `nodes canvas présent --node <id|name|ip> [--target <urlOrPath>] [--x <px>] [--y <px>] [--width <px>] [--height <px>] [--invoke-timeout <ms>]`
- `nodes canvas hide --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes canvas navigate <url> --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes canvas eval [<js>] --node <id|name|ip> [--js <code>] [--invoke-timeout <ms>]`
- `nodes canvas a2ui push --node <id|name|ip> (--jsonl <path> | --text <text>) [--invoke-timeout <ms>]`
- `nodes canvas a2ui reset --node <id|name|ip> [--invoke-timeout <ms>]`
- `nodes screen record --node <id|name|ip> [--screen <index>] [--duration <ms|10s>] [--fps <n>] [--no-audio] [--out <path>] [--invoke-timeout <ms>]`

Localisation :

- `nodes location get --node <id|name|ip> [--max-age <ms>] [--accuracy <coarse|balanced|precise>] [--location-timeout <ms>] [--invoke-timeout <ms>]`

## Navigateur

CLI de contrôle du navigateur (Chrome/Brave/Edge/Chromium dédié). Voir [`openclaw browser`](/cli/browser) et l'[outil Navigateur](/tools/browser).

Options communes :

- `--url`, `--token`, `--timeout`, `--json`
- `--browser-profile <name>`

Gestion :

- `browser status`
- `browser start`
- `browser stop`
- `browser reset-profile`
- `browser tabs`
- `browser open <url>`
- `browser focus <targetId>`
- `browser close [targetId]`
- `browser profiles`
- `browser create-profile --name <name> [--color <hex>] [--cdp-url <url>]`
- `browser delete-profile --name <name>`

Inspection :

- `browser screenshot [targetId] [--full-page] [--ref <ref>] [--élément <selector>] [--type png|jpeg]`
- `browser snapshot [--format aria|ai] [--target-id <id>] [--limit <n>] [--interactive] [--compact] [--depth <n>] [--selector <sel>] [--out <path>]`

Actions :

- `browser navigate <url> [--target-id <id>]`
- `browser resize <width> <height> [--target-id <id>]`
- `browser click <ref> [--double] [--button <left|right|middle>] [--modifiers <csv>] [--target-id <id>]`
- `browser type <ref> <text> [--submit] [--slowly] [--target-id <id>]`
- `browser press <key> [--target-id <id>]`
- `browser hover <ref> [--target-id <id>]`
- `browser drag <startRef> <endRef> [--target-id <id>]`
- `browser select <ref> <values...> [--target-id <id>]`
- `browser upload <paths...> [--ref <ref>] [--input-ref <ref>] [--élément <selector>] [--target-id <id>] [--timeout-ms <ms>]`
- `browser fill [--fields <json>] [--fields-file <path>] [--target-id <id>]`
- `browser dialog --accept|--dismiss [--prompt <text>] [--target-id <id>] [--timeout-ms <ms>]`
- `browser wait [--time <ms>] [--text <value>] [--text-gone <value>] [--target-id <id>]`
- `browser evaluate --fn <code> [--ref <ref>] [--target-id <id>]`
- `browser console [--level <error|warn|info>] [--target-id <id>]`
- `browser pdf [--target-id <id>]`

## Recherche dans la documentation

### `docs [query...]`

Rechercher dans l'index de documentation en ligne.

## TUI

### `tui`

Ouvrir l'interface terminal connectée au Gateway.

Options :

- `--url <url>`
- `--token <token>`
- `--password <password>`
- `--session <key>`
- `--deliver`
- `--thinking <level>`
- `--message <text>`
- `--timeout-ms <ms>` (par défaut `agents.defaults.timeoutSeconds`)
- `--history-limit <n>`
