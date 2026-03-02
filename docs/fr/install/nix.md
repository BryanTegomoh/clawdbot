---
summary: "Installer OpenClaw de manière déclarative avec Nix"
read_when:
  - Vous souhaitez des installations reproductibles avec possibilité de rollback
  - Vous utilisez déjà Nix/NixOS/Home Manager
  - Vous voulez tout épingler et gérer de manière déclarative
title: "Nix"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/install/nix.md
  workflow: manual
---

# Installation Nix

La méthode recommandée pour exécuter OpenClaw avec Nix est via **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** : un module Home Manager complet et prêt à l'emploi.

## Démarrage rapide

Collez ceci à votre agent IA (Claude, Cursor, etc.) :

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **Guide complet : [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Le dépôt nix-openclaw est la source de référence pour l'installation Nix. Cette page n'est qu'un aperçu rapide.

## Ce que vous obtenez

- Gateway + application macOS + outils (whisper, spotify, cameras) : tout épinglé
- Service launchd qui survit aux redémarrages
- Système de plugins avec configuration déclarative
- Rollback instantané : `home-manager switch --rollback`

---

## Comportement à l'exécution en mode Nix

Quand `OPENCLAW_NIX_MODE=1` est défini (automatique avec nix-openclaw) :

OpenClaw prend en charge un **mode Nix** qui rend la configuration déterministe et désactive les flux d'installation automatique.
Activez-le en exportant :

```bash
OPENCLAW_NIX_MODE=1
```

Sur macOS, l'application GUI n'hérite pas automatiquement des variables d'environnement du shell. Vous pouvez
également activer le mode Nix via defaults :

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### Chemins de configuration et d'état

OpenClaw lit la configuration JSON5 depuis `OPENCLAW_CONFIG_PATH` et stocke les données modifiables dans `OPENCLAW_STATE_DIR`.
Si nécessaire, vous pouvez également définir `OPENCLAW_HOME` pour contrôler le répertoire personnel de base utilise pour la résolution interne des chemins.

- `OPENCLAW_HOME` (priorité par défaut : `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (par défaut : `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (par défaut : `$OPENCLAW_STATE_DIR/openclaw.json`)

Sous Nix, définissez-les explicitement vers des emplacements gérés par Nix afin que l'état d'exécution et la configuration
restent en dehors du store immuable.

### Comportement à l'exécution en mode Nix

- Les flux d'installation automatique et d'auto-mutation sont désactivés
- Les dépendances manquantes affichent des messages de remédiation spécifiques à Nix
- L'interface affiche une bannière mode Nix en lecture seule quand elle est présente

## Note sur le packaging (macOS)

Le flux de packaging macOS s'attend à un modèle Info.plist stable dans :

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copie ce modèle dans le bundle de l'application et applique les champs dynamiques
(bundle ID, version/build, Git SHA, clés Sparkle). Cela maintient le plist déterministe pour le packaging SwiftPM
et les builds Nix (qui ne reposent pas sur une toolchain Xcode complète).

## Liens connexes

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) : guide de configuration complet
- [Assistant](/start/wizard) : configuration CLI sans Nix
- [Docker](/install/docker) : configuration conteneurisée
