---
summary: "Référence complète pour le flux d'intégration CLI, la configuration auth/modèle, les sorties et les détails internes"
read_when:
  - Vous avez besoin du comportement détaillé de openclaw onboard
  - Vous déboguez les résultats d'intégration ou intégrez des clients d'intégration
title: "Référence d'intégration CLI"
sidebarTitle: "Référence CLI"
---

# Référence d'intégration CLI

Cette page est la référence complète pour `openclaw onboard`.
Pour le guide court, voir [Assistant d'intégration (CLI)](/start/wizard).

## Ce que fait l'assistant

Le mode local (par défaut) vous guide à travers :

- Configuration du modèle et de l'authentification (abonnement OpenAI Code OAuth, clé API Anthropic ou token de configuration, plus les options MiniMax, GLM, Moonshot et AI Gateway)
- Emplacement de l'espace de travail et fichiers d'amorçage
- Paramètres de la passerelle (port, bind, auth, tailscale)
- Canaux et fournisseurs (Telegram, WhatsApp, Discord, Google Chat, plugin Mattermost, Signal)
- Installation du daemon (LaunchAgent ou unité utilisateur systemd)
- Vérification de santé
- Configuration des compétences

Le mode distant configure cette machine pour se connecter à une passerelle située ailleurs.
Il n'installe ni ne modifie rien sur l'hôte distant.

## Détails du flux local

<Steps>
  <Step title="Détection de la configuration existante">
    - Si `~/.openclaw/openclaw.json` existe, choisissez Conserver, Modifier ou Réinitialiser.
    - Réexécuter l'assistant ne supprime rien sauf si vous choisissez explicitement Réinitialiser (ou passez `--reset`).
    - Le CLI `--reset` utilise par défaut `config+creds+sessions` ; utilisez `--reset-scope full` pour supprimer également l'espace de travail.
    - Si la configuration est invalide ou contient des clés héritées, l'assistant s'arrête et vous demande d'exécuter `openclaw doctor` avant de continuer.
    - La réinitialisation utilise `trash` et propose des périmètres :
      - Configuration uniquement
      - Configuration + identifiants + sessions
      - Réinitialisation complète (supprime également l'espace de travail)
  </Step>
  <Step title="Modèle et authentification">
    - La matrice complète des options est dans [Options d'authentification et de modèle](#options-dauthentification-et-de-modèle).
  </Step>
  <Step title="Espace de travail">
    - Par défaut `~/.openclaw/workspace` (configurable).
    - Alimente les fichiers d'espace de travail nécessaires au rituel d'amorçage initial.
    - Disposition de l'espace de travail : [Espace de travail de l'agent](/concepts/agent-workspace).
  </Step>
  <Step title="Passerelle">
    - Demande le port, le bind, le mode d'authentification et l'exposition tailscale.
    - Recommandé : gardez l'authentification par token activée même pour loopback afin que les clients WS locaux doivent s'authentifier.
    - Désactivez l'authentification uniquement si vous faites entièrement confiance à chaque processus local.
    - Les binds non-loopback nécessitent toujours l'authentification.
  </Step>
  <Step title="Canaux">
    - [WhatsApp](/channels/whatsapp) : connexion QR optionnelle
    - [Telegram](/channels/telegram) : token du bot
    - [Discord](/channels/discord) : token du bot
    - [Google Chat](/channels/googlechat) : JSON du compte de service + audience webhook
    - [Mattermost](/channels/mattermost) plugin : token du bot + URL de base
    - [Signal](/channels/signal) : installation optionnelle de `signal-cli` + configuration du compte
    - [BlueBubbles](/channels/bluebubbles) : recommandé pour iMessage ; URL du serveur + mot de passe + webhook
    - [iMessage](/channels/imessage) : chemin CLI `imsg` hérité + accès DB
    - Sécurité des DM : le défaut est l'appairage. Le premier DM envoie un code ; approuvez via
      `openclaw pairing approve <channel> <code>` ou utilisez des listes d'autorisation.
  </Step>
  <Step title="Installation du daemon">
    - macOS : LaunchAgent
      - Nécessite une session utilisateur connectée ; pour le mode headless, utilisez un LaunchDaemon personnalisé (non fournit).
    - Linux et Windows via WSL2 : unité utilisateur systemd
      - L'assistant tente `loginctl enable-linger <user>` pour que la passerelle reste active après la déconnexion.
      - Peut demander sudo (écrit dans `/var/lib/systemd/linger`) ; il essaie d'abord sans sudo.
    - Sélection du runtime : Node (recommandé ; requis pour WhatsApp et Telegram). Bun n'est pas recommandé.
  </Step>
  <Step title="Vérification de santé">
    - Démarre la passerelle (si nécessaire) et exécute `openclaw health`.
    - `openclaw status --deep` ajoute les sondes de santé de la passerelle à la sortie de statut.
  </Step>
  <Step title="Compétences">
    - Lit les compétences disponibles et vérifie les prérequis.
    - Vous permet de choisir le gestionnaire de nœuds : npm ou pnpm (bun non recommandé).
    - Installe les dépendances optionnelles (certaines utilisent Homebrew sur macOS).
  </Step>
  <Step title="Terminer">
    - Résumé et prochaines étapes, y compris les options d'application iOS, Android et macOS.
  </Step>
</Steps>

<Note>
Si aucune interface graphique n'est détectée, l'assistant affiche les instructions de redirection de port SSH pour l'UI de contrôle au lieu d'ouvrir un navigateur.
Si les ressources de l'UI de contrôle sont manquantes, l'assistant tente de les construire ; le repli est `pnpm ui:build` (installe automatiquement les dépendances UI).
</Note>

## Détails du mode distant

Le mode distant configure cette machine pour se connecter à une passerelle située ailleurs.

<Info>
Le mode distant n'installe ni ne modifie rien sur l'hôte distant.
</Info>

Ce que vous définissez :

- URL de la passerelle distante (`ws://...`)
- Token si l'authentification de la passerelle distante est requise (recommandé)

<Note>
- Si la passerelle est en loopback uniquement, utilisez le tunneling SSH ou un tailnet.
- Indications de découverte :
  - macOS : Bonjour (`dns-sd`)
  - Linux : Avahi (`avahi-browse`)
</Note>

## Options d'authentification et de modèle

<AccordionGroup>
  <Accordion title="Clé API Anthropic (recommandé)">
    Utilise `ANTHROPIC_API_KEY` si présente ou demande une clé, puis la sauvegarde pour l'utilisation du daemon.
  </Accordion>
  <Accordion title="Anthropic OAuth (CLI Claude Code)">
    - macOS : vérifie l'élément Keychain "Claude Code-credentials"
    - Linux et Windows : réutilise `~/.claude/.credentials.json` si présent

    Sur macOS, choisissez "Toujours autoriser" pour que les démarrages launchd ne bloquent pas.

  </Accordion>
  <Accordion title="Token Anthropic (collage de setup-token)">
    Exécutez `claude setup-token` sur n'importe quelle machine, puis collez le token.
    Vous pouvez le nommer ; laissez vide pour le défaut.
  </Accordion>
  <Accordion title="Abonnement OpenAI Code (réutilisation du CLI Codex)">
    Si `~/.codex/auth.json` existe, l'assistant peut le réutiliser.
  </Accordion>
  <Accordion title="Abonnement OpenAI Code (OAuth)">
    Flux navigateur ; collez `code#state`.

    Définit `agents.defaults.model` à `openai-codex/gpt-5.3-codex` lorsque le modèle n'est pas défini ou est `openai/*`.

  </Accordion>
  <Accordion title="Clé API OpenAI">
    Utilise `OPENAI_API_KEY` si présente ou demande une clé, puis stocke l'identifiant dans les profils d'authentification.

    Définit `agents.defaults.model` à `openai/gpt-5.1-codex` lorsque le modèle n'est pas défini, est `openai/*` ou `openai-codex/*`.

  </Accordion>
  <Accordion title="Clé API xAI (Grok)">
    Demande `XAI_API_KEY` et configure xAI comme fournisseur de modèle.
  </Accordion>
  <Accordion title="OpenCode Zen">
    Demande `OPENCODE_API_KEY` (ou `OPENCODE_ZEN_API_KEY`).
    URL de configuration : [opencode.ai/auth](https://opencode.ai/auth).
  </Accordion>
  <Accordion title="Clé API (générique)">
    Stocke la clé pour vous.
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    Demande `AI_GATEWAY_API_KEY`.
    Plus de détails : [Vercel AI Gateway](/providers/vercel-ai-gateway).
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    Demande l'ID de compte, l'ID de passerelle et `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    Plus de détails : [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  </Accordion>
  <Accordion title="MiniMax M2.1">
    La configuration est auto-écrite.
    Plus de détails : [MiniMax](/providers/minimax).
  </Accordion>
  <Accordion title="Synthetic (compatible Anthropic)">
    Demande `SYNTHETIC_API_KEY`.
    Plus de détails : [Synthetic](/providers/synthetic).
  </Accordion>
  <Accordion title="Moonshot et Kimi Coding">
    Les configurations Moonshot (Kimi K2) et Kimi Coding sont auto-écrites.
    Plus de détails : [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  </Accordion>
  <Accordion title="Fournisseur personnalisé">
    Fonctionne avec les endpoints compatibles OpenAI et Anthropic.

    L'intégration interactive prend en charge les mêmes choix de stockage de clé API que les autres flux de clé API de fournisseur :
    - **Coller la clé API maintenant** (texte clair)
    - **Utiliser une référence de secret** (référence env ou référence de fournisseur configurée, avec validation préalable)

    Drapeaux non interactifs :
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (optionnel ; se rabat sur `CUSTOM_API_KEY`)
    - `--custom-provider-id` (optionnel)
    - `--custom-compatibility <openai|anthropic>` (optionnel ; défaut `openai`)

  </Accordion>
  <Accordion title="Ignorer">
    Laisse l'authentification non configurée.
  </Accordion>
</AccordionGroup>

Comportement du modèle :

- Choisissez le modèle par défaut parmi les options détectées, ou entrez manuellement le fournisseur et le modèle.
- L'assistant exécute une vérification du modèle et avertit si le modèle configuré est inconnu ou sans authentification.

Chemins des identifiants et profils :

- Identifiants OAuth : `~/.openclaw/credentials/oauth.json`
- Profils d'authentification (clés API + OAuth) : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

Mode de stockage de clé API :

- Le comportement par défaut de l'intégration persiste les clés API comme valeurs en texte clair dans les profils d'authentification.
- `--secret-input-mode ref` active le mode référence au lieu du stockage de clé en texte clair.
  En intégration interactive, vous pouvez choisir :
  - référence de variable d'environnement (par exemple `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`)
  - référence de fournisseur configurée (`file` ou `exec`) avec alias de fournisseur + id
- Le mode référence interactif exécute une validation préalable rapide avant d'enregistrer.
  - Références env : valide le nom de la variable + la valeur non vide dans l'environnement d'intégration actuel.
  - Références de fournisseur : valide la configuration du fournisseur et résout l'id demandé.
  - Si la validation préalable échoue, l'intégration affiche l'erreur et vous permet de réessayer.
- En mode non interactif, `--secret-input-mode ref` est basé sur env uniquement.
  - Définissez la variable env du fournisseur dans l'environnement du processus d'intégration.
  - Les drapeaux de clé en ligne (par exemple `--openai-api-key`) nécessitent que cette variable env soit définie ; sinon l'intégration échoue rapidement.
  - Pour les fournisseurs personnalisés, le mode `ref` non interactif stocke `models.providers.<id>.apiKey` comme `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.
  - Dans ce cas de fournisseur personnalisé, `--custom-api-key` nécessite que `CUSTOM_API_KEY` soit définie ; sinon l'intégration échoue rapidement.
- Les configurations existantes en texte clair continuent de fonctionner sans changement.

<Note>
Astuce pour serveur headless : complétez OAuth sur une machine avec un navigateur, puis copiez
`~/.openclaw/credentials/oauth.json` (ou `$OPENCLAW_STATE_DIR/credentials/oauth.json`)
vers l'hôte de la passerelle.
</Note>

## Sorties et détails internes

Champs typiques dans `~/.openclaw/openclaw.json` :

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (si Minimax choisi)
- `gateway.*` (mode, bind, auth, tailscale)
- `session.dmScope` (l'intégration locale définit par défaut à `per-channel-peer` si non défini ; les valeurs explicites existantes sont préservées)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Listes d'autorisation des canaux (Slack, Discord, Matrix, Microsoft Teams) lorsque vous optez pendant les invites (les noms sont résolus en IDs quand possible)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` écrit `agents.list[]` et les `bindings` optionnels.

Les identifiants WhatsApp vont sous `~/.openclaw/credentials/whatsapp/<accountId>/`.
Les sessions sont stockées sous `~/.openclaw/agents/<agentId>/sessions/`.

<Note>
Certains canaux sont livrés sous forme de plugins. Lorsqu'ils sont sélectionnés pendant l'intégration, l'assistant
propose d'installer le plugin (npm ou chemin local) avant la configuration du canal.
</Note>

RPC de l'assistant de la passerelle :

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

Les clients (application macOS et UI de contrôle) peuvent afficher les étapes sans réimplémenter la logique d'intégration.

Comportement de la configuration Signal :

- Télécharge la ressource de publication appropriée
- La stocke sous `~/.openclaw/tools/signal-cli/<version>/`
- Écrit `channels.signal.cliPath` dans la configuration
- Les builds JVM nécessitent Java 21
- Les builds natifs sont utilisés lorsqu'ils sont disponibles
- Windows utilise WSL2 et suit le flux signal-cli Linux à l'intérieur de WSL

## Documents connexes

- Hub d'intégration : [Assistant d'intégration (CLI)](/start/wizard)
- Automation et scripts : [Automation CLI](/start/wizard-cli-automation)
- Référence de commande : [`openclaw onboard`](/cli/onboard)
