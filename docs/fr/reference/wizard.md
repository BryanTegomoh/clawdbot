---
summary: "Référence complète de l'assistant d'onboarding CLI : chaque étape, drapeau et champ de configuration"
read_when:
  - Recherche d'une étape ou d'un drapeau spécifique de l'assistant
  - Automatisation de l'onboarding en mode non interactif
  - Débogage du comportement de l'assistant
title: "Référence de l'assistant d'onboarding"
sidebarTitle: "Référence de l'assistant"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/wizard.md
  workflow: manual
---

# Référence de l'assistant d'onboarding

Ceci est la référence complète de l'assistant CLI `openclaw onboard`.
Pour un aperçu de haut niveau, voir [Onboarding Wizard](/start/wizard).

## Détails du flux (mode local)

<Steps>
  <Step title="Détection de configuration existante">
    - Si `~/.openclaw/openclaw.json` existe, choisir **Conserver / Modifier / Réinitialiser**.
    - Relancer l'assistant ne **supprimé rien** sauf si vous choisissez explicitement **Réinitialiser**
      (ou passez `--reset`).
    - CLI `--reset` par défaut réinitialise `config+creds+sessions` ; utilisez `--reset-scope full`
      pour supprimer aussi l'espace de travail.
    - Si la configuration est invalide ou contient des clés obsolètes, l'assistant s'arrête et vous demande
      d'exécuter `openclaw doctor` avant de continuer.
    - La réinitialisation utilise `trash` (jamais `rm`) et propose des portées :
      - Configuration uniquement
      - Configuration + identifiants + sessions
      - Réinitialisation complète (supprimé également l'espace de travail)
  </Step>
  <Step title="Modèle/Authentification">
    - **Clé API Anthropic (recommandé)** : utilisé `ANTHROPIC_API_KEY` si présente ou demande une clé, puis la sauvegarde pour l'utilisation du démon.
    - **OAuth Anthropic (Claude Code CLI)** : sur macOS, l'assistant vérifie l'élément Trousseau "Claude Code-credentials" (choisissez "Toujours autoriser" pour que les démarrages launchd ne bloquent pas) ; sur Linux/Windows, il réutilise `~/.claude/.credentials.json` si présent.
    - **Jeton Anthropic (coller le setup-token)** : exécutez `claude setup-token` sur n'importe quelle machine, puis collez le jeton (vous pouvez le nommer ; vide = défaut).
    - **Abonnement OpenAI Code (Codex) (Codex CLI)** : si `~/.codex/auth.json` existe, l'assistant peut le réutiliser.
    - **Abonnement OpenAI Code (Codex) (OAuth)** : flux navigateur ; collez le `code#state`.
      - Définit `agents.defaults.model` à `openai-codex/gpt-5.2` quand le modèle n'est pas défini ou est `openai/*`.
    - **Clé API OpenAI** : utilisé `OPENAI_API_KEY` si présente ou demande une clé, puis la stocke dans les profils d'authentification.
    - **Clé API xAI (Grok)** : demande `XAI_API_KEY` et configure xAI comme fournisseur de modèle.
    - **OpenCode Zen (proxy multi-modèles)** : demande `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`, obtenez-la sur https://opencode.ai/auth).
    - **Clé API** : stocke la clé pour vous.
    - **Vercel AI Gateway (proxy multi-modèles)** : demande `AI_GATEWAY_API_KEY`.
    - Plus de détails : [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway** : demande l'ID de compte, l'ID de passerelle et `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    - Plus de détails : [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1** : la configuration est écrite automatiquement.
    - Plus de détails : [MiniMax](/providers/minimax)
    - **Synthetic (compatible Anthropic)** : demande `SYNTHETIC_API_KEY`.
    - Plus de détails : [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)** : la configuration est écrite automatiquement.
    - **Kimi Coding** : la configuration est écrite automatiquement.
    - Plus de détails : [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Ignorer** : pas d'authentification configurée pour l'instant.
    - Choisir un modèle par défaut parmi les options détectées (ou saisir fournisseur/modèle manuellement).
    - L'assistant exécute une vérification du modèle et avertit si le modèle configuré est inconnu ou manque d'authentification.
    - Le mode de stockage des clés API est par défaut en clair dans les profils d'authentification. Utilisez `--secret-input-mode ref` pour stocker des références basées sur l'environnement (par exemple `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
    - Les identifiants OAuth se trouvent dans `~/.openclaw/credentials/oauth.json` ; les profils d'authentification dans `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (clés API + OAuth).
    - Plus de détails : [/concepts/oauth](/concepts/oauth)
    <Note>
    Astuce headless/serveur : complétez OAuth sur une machine avec un navigateur, puis copiez
    `~/.openclaw/credentials/oauth.json` (ou `$OPENCLAW_STATE_DIR/credentials/oauth.json`) vers
    l'hôte Gateway.
    </Note>
  </Step>
  <Step title="Espace de travail">
    - Par défaut `~/.openclaw/workspace` (configurable).
    - Initialisé les fichiers d'espace de travail nécessaires au rituel de démarrage de l'agent.
    - Disposition complète de l'espace de travail + guide de sauvegarde : [Agent workspace](/concepts/agent-workspace)
  </Step>
  <Step title="Gateway">
    - Port, bind, mode d'authentification, exposition tailscale.
    - Recommandation d'authentification : gardez **Token** même pour le loopback afin que les clients WS locaux doivent s'authentifier.
    - Désactivez l'authentification uniquement si vous faites entièrement confiance à tous les processus locaux.
    - Les binds non loopback requièrent toujours l'authentification.
  </Step>
  <Step title="Canaux">
    - [WhatsApp](/channels/whatsapp) : connexion QR optionnelle.
    - [Telegram](/channels/telegram) : jeton de bot.
    - [Discord](/channels/discord) : jeton de bot.
    - [Google Chat](/channels/googlechat) : JSON de compte de service + audience webhook.
    - [Mattermost](/channels/mattermost) (plugin) : jeton de bot + URL de base.
    - [Signal](/channels/signal) : installation optionnelle de `signal-cli` + configuration du compte.
    - [BlueBubbles](/channels/bluebubbles) : **recommandé pour iMessage** ; URL du serveur + mot de passe + webhook.
    - [iMessage](/channels/imessage) : chemin CLI `imsg` legacy + accès base de données.
    - Sécurité DM : par défaut, appariement. Le premier DM envoie un code ; approuvez via `openclaw pairing approve <channel> <code>` ou utilisez des listes d'autorisation.
  </Step>
  <Step title="Installation du démon">
    - macOS : LaunchAgent
      - Nécessite une session utilisateur connectée ; pour le headless, utilisez un LaunchDaemon personnalisé (non fournit).
    - Linux (et Windows via WSL2) : unité systemd utilisateur
      - L'assistant tente d'activer le lingering via `loginctl enable-linger <user>` pour que le Gateway reste actif après la déconnexion.
      - Peut demander sudo (écriture dans `/var/lib/systemd/linger`) ; il essaie d'abord sans sudo.
    - **Sélection du runtime :** Node (recommandé ; requis pour WhatsApp/Telegram). Bun n'est **pas recommandé**.
  </Step>
  <Step title="Vérification de santé">
    - Démarre le Gateway (si nécessaire) et exécute `openclaw health`.
    - Astuce : `openclaw status --deep` ajouté des sondes de santé du Gateway à la sortie du statut (nécessite un Gateway joignable).
  </Step>
  <Step title="Skills (recommandé)">
    - Lit les skills disponibles et vérifie les prérequis.
    - Vous permet de choisir un gestionnaire de paquets : **npm / pnpm** (bun non recommandé).
    - Installe les dépendances optionnelles (certaines utilisent Homebrew sur macOS).
  </Step>
  <Step title="Fin">
    - Résumé + prochaines étapes, incluant les applications iOS/Android/macOS pour des fonctionnalités supplémentaires.
  </Step>
</Steps>

<Note>
Si aucune interface graphique n'est détectée, l'assistant affiche les instructions de redirection de port SSH pour le Control UI au lieu d'ouvrir un navigateur.
Si les ressources du Control UI sont manquantes, l'assistant tente de les compiler ; le repli est `pnpm ui:build` (installe automatiquement les dépendances UI).
</Note>

## Mode non interactif

Utilisez `--non-interactive` pour automatiser ou scripter l'onboarding :

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Ajoutez `--json` pour un résumé lisible par machine.

<Note>
`--json` n'implique **pas** le mode non interactif. Utilisez `--non-interactive` (et `--workspace`) pour les scripts.
</Note>

<AccordionGroup>
  <Accordion title="Exemple Gemini">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Exemple Z.AI">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Exemple Vercel AI Gateway">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Exemple Cloudflare AI Gateway">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Exemple Moonshot">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Exemple Synthetic">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Exemple OpenCode Zen">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### Ajouter un agent (non interactif)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## RPC de l'assistant Gateway

Le Gateway expose le flux de l'assistant via RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Les clients (application macOS, Control UI) peuvent rendre les étapes sans ré-implémenter la logique d'onboarding.

## Configuration Signal (signal-cli)

L'assistant peut installer `signal-cli` depuis les releases GitHub :

- Télécharge l'asset de release approprié.
- Le stocke sous `~/.openclaw/tools/signal-cli/<version>/`.
- Écrit `channels.signal.cliPath` dans votre configuration.

Notes :

- Les builds JVM nécessitent **Java 21**.
- Les builds natifs sont utilisés quand disponibles.
- Windows utilise WSL2 ; l'installation de signal-cli suit le flux Linux à l'intérieur de WSL.

## Ce que l'assistant écrit

Champs typiques dans `~/.openclaw/openclaw.json` :

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (si Minimax choisi)
- `gateway.*` (mode, bind, auth, tailscale)
- `session.dmScope` (détails de comportement : [CLI Onboarding Référence](/start/wizard-cli-reference#outputs-and-internals))
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Listes d'autorisation de canaux (Slack/Discord/Matrix/Microsoft Teams) quand vous optez pendant les invites (les noms sont résolus en identifiants quand possible).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` écrit `agents.list[]` et optionnellement `bindings`.

Les identifiants WhatsApp vont sous `~/.openclaw/credentials/whatsapp/<accountId>/`.
Les sessions sont stockées sous `~/.openclaw/agents/<agentId>/sessions/`.

Certains canaux sont fournis sous forme de plugins. Quand vous en choisissez un pendant l'onboarding, l'assistant
vous demandera de l'installer (npm ou un chemin local) avant de pouvoir le configurer.

## Documentation connexe

- Aperçu de l'assistant : [Onboarding Wizard](/start/wizard)
- Onboarding application macOS : [Onboarding](/start/onboarding)
- Référence de configuration : [Gateway configuration](/gateway/configuration)
- Fournisseurs : [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (legacy)
- Skills : [Skills](/tools/skills), [Skills config](/tools/skills-config)
