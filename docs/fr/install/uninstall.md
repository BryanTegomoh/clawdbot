---
summary: "Désinstaller OpenClaw complètement (CLI, service, état, espace de travail)"
read_when:
  - Vous souhaitez supprimer OpenClaw d'une machine
  - Le service Gateway continue de tourner après la désinstallation
title: "Désinstallation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/install/uninstall.md
  workflow: manual
---

# Désinstallation

Deux méthodes :

- **Méthode simple** si `openclaw` est encore installé.
- **Suppression manuelle du service** si le CLI a disparu mais que le service tourne toujours.

## Méthode simple (CLI encore installé)

Recommandé : utilisez le désinstallateur intégré :

```bash
openclaw uninstall
```

Non interactif (automatisation / npx) :

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Étapes manuelles (même résultat) :

1. Arrêtez le service Gateway :

```bash
openclaw gateway stop
```

2. Désinstallez le service Gateway (launchd/systemd/schtasks) :

```bash
openclaw gateway uninstall
```

3. Supprimez l'état + la configuration :

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Si vous avez défini `OPENCLAW_CONFIG_PATH` vers un emplacement personnalisé en dehors du répertoire d'état, supprimez ce fichier également.

4. Supprimez votre espace de travail (optionnel, supprimé les fichiers d'agent) :

```bash
rm -rf ~/.openclaw/workspace
```

5. Supprimez l'installation CLI (choisissez celle que vous avez utilisée) :

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. Si vous avez installé l'application macOS :

```bash
rm -rf /Applications/OpenClaw.app
```

Notes :

- Si vous avez utilisé des profils (`--profile` / `OPENCLAW_PROFILE`), répétez l'étape 3 pour chaque répertoire d'état (les chemins par défaut sont `~/.openclaw-<profile>`).
- En mode distant, le répertoire d'état se trouve sur l'**hôte du Gateway**, exécutez donc les étapes 1-4 là aussi.

## Suppression manuelle du service (CLI non installé)

Utilisez cette méthode si le service Gateway continue de tourner mais que `openclaw` est absent.

### macOS (launchd)

Le libellé par défaut est `ai.openclaw.gateway` (ou `ai.openclaw.<profile>` ; l'ancien `com.openclaw.*` peut encore exister) :

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

Si vous avez utilisé un profil, remplacez le libellé et le nom du plist par `ai.openclaw.<profile>`. Supprimez tout ancien plist `com.openclaw.*` si présent.

### Linux (unité utilisateur systemd)

Le nom d'unité par défaut est `openclaw-gateway.service` (ou `openclaw-gateway-<profile>.service`) :

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (tâche planifiée)

Le nom de tâche par défaut est `OpenClaw Gateway` (ou `OpenClaw Gateway (<profile>)`).
Le script de la tâche se trouve sous votre répertoire d'état.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Si vous avez utilisé un profil, supprimez le nom de tâche correspondant et `~\.openclaw-<profile>\gateway.cmd`.

## Installation normale vs checkout des sources

### Installation normale (install.sh / npm / pnpm / bun)

Si vous avez utilisé `https://openclaw.ai/install.sh` ou `install.ps1`, le CLI a été installé avec `npm install -g openclaw@latest`.
Supprimez-le avec `npm rm -g openclaw` (ou `pnpm remove -g` / `bun remove -g` si vous avez installé de cette façon).

### Checkout des sources (git clone)

Si vous exécutez depuis un checkout de dépôt (`git clone` + `openclaw ...` / `bun run openclaw ...`) :

1. Désinstallez le service Gateway **avant** de supprimer le dépôt (utilisez la méthode simple ci-dessus ou la suppression manuelle du service).
2. Supprimez le répertoire du dépôt.
3. Supprimez l'état + l'espace de travail comme indiqué ci-dessus.
