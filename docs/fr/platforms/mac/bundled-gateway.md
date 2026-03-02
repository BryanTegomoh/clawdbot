---
summary: "Runtime du Gateway sur macOS (service launchd externe)"
read_when:
  - Empaquetage d'OpenClaw.app
  - Debogage du service launchd du gateway macOS
  - Installation du CLI gateway pour macOS
title: "Gateway sur macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/bundled-gateway.md
  workflow: manual
---

# Gateway sur macOS (launchd externe)

OpenClaw.app n'intègre plus Node/Bun ni le runtime du Gateway. L'application macOS
attend une installation **externe** du CLI `openclaw`, ne lance pas le Gateway en tant que
processus enfant, et gère un service launchd par utilisateur pour maintenir le Gateway
en cours d'exécution (ou se connecte a un Gateway local existant s'il est déjà en cours d'exécution).

## Installer le CLI (requis pour le mode local)

Vous avez besoin de Node 22+ sur le Mac, puis installez `openclaw` globalement :

```bash
npm install -g openclaw@<version>
```

Le bouton **Install CLI** de l'application macOS exécute le même flux via npm/pnpm (bun non recommandé pour le runtime du Gateway).

## Launchd (Gateway en tant que LaunchAgent)

Label :

- `ai.openclaw.gateway` (ou `ai.openclaw.<profile>` ; l'ancien `com.openclaw.*` peut subsister)

Emplacement du plist (par utilisateur) :

- `~/Library/LaunchAgents/ai.openclaw.gateway.plist`
  (ou `~/Library/LaunchAgents/ai.openclaw.<profile>.plist`)

Gestionnaire :

- L'application macOS gère l'installation/mise a jour du LaunchAgent en mode Local.
- Le CLI peut également l'installer : `openclaw gateway install`.

Comportement :

- « OpenClaw Activé » activé/désactive le LaunchAgent.
- Quitter l'application **n'arrêté pas** le gateway (launchd le maintient actif).
- Si un Gateway est déjà en cours d'exécution sur le port configuré, l'application se connecte
  a celui-ci au lieu d'en démarrer un nouveau.

Logs :

- stdout/err de launchd : `/tmp/openclaw/openclaw-gateway.log`

## Compatibilité des versions

L'application macOS vérifie la version du gateway par rapport a sa propre version. Si elles sont
incompatibles, mettez a jour le CLI global pour correspondre a la version de l'application.

## Vérification rapide

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Puis :

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
