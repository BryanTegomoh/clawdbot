---
summary: "Référence CLI pour `openclaw browser` (profils, onglets, actions, relais d'extension)"
read_when:
  - Vous utilisez `openclaw browser` et voulez des exemples pour les tâches courantes
  - Vous souhaitez contrôler un navigateur s'exécutant sur une autre machine via un node host
  - Vous souhaitez utiliser le relais d'extension Chrome (attacher/detacher via le bouton de la barre d'outils)
title: "browser"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/browser.md
  workflow: manual
---

# `openclaw browser`

Gérer le serveur de contrôle du navigateur d'OpenClaw et exécuter des actions de navigateur (onglets, instantanés, captures d'écran, navigation, clics, saisie).

Liens :

- Outil navigateur + API : [Browser tool](/tools/browser)
- Relais d'extension Chrome : [Chrome extension](/tools/chrome-extension)

## Options communes

- `--url <gatewayWsUrl>` : URL WebSocket du Gateway (par défaut depuis la configuration).
- `--token <token>` : jeton du Gateway (si requis).
- `--timeout <ms>` : délai d'expiration de la requête (ms).
- `--browser-profile <name>` : choisir un profil de navigateur (défaut depuis la configuration).
- `--json` : sortie lisible par machine (lorsque supporté).

## Démarrage rapide (local)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## Profils

Les profils sont des configurations de routage de navigateur nommées. En pratique :

- `openclaw` : lance/s'attache à une instance Chrome dédiée gérée par OpenClaw (répertoire de données utilisateur isole).
- `chrome` : contrôle vos onglets Chrome existants via le relais d'extension Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Utiliser un profil spécifique :

```bash
openclaw browser --browser-profile work tabs
```

## Onglets

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## Instantane / capture d'écran / actions

Instantane :

```bash
openclaw browser snapshot
```

Capture d'écran :

```bash
openclaw browser screenshot
```

Naviguer/cliquer/saisir (automatisation UI basée sur les références) :

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Relais d'extension Chrome (attacher via le bouton de la barre d'outils)

Ce mode permet à l'agent de contrôler un onglet Chrome existant que vous attachez manuellement (il ne s'attache pas automatiquement).

Installer l'extension non empaquetée vers un chemin stable :

```bash
openclaw browser extension install
openclaw browser extension path
```

Puis Chrome -> `chrome://extensions` -> activer "Mode développéur" -> "Charger l'extension non empaquetée" -> sélectionnez le dossier affiche.

Guide complet : [Chrome extension](/tools/chrome-extension)

## Contrôle de navigateur à distance (proxy node host)

Si le Gateway s'exécute sur une machine différente de celle du navigateur, exécutez un **node host** sur la machine qui dispose de Chrome/Brave/Edge/Chromium. Le Gateway transmettra les actions de navigateur à ce node (aucun serveur de contrôle de navigateur séparé requis).

Utilisez `gateway.nodes.browser.mode` pour contrôler le routage automatique et `gateway.nodes.browser.node` pour fixer un node spécifique si plusieurs sont connectés.

Sécurité + configuration distante : [Browser tool](/tools/browser), [Remote access](/gateway/remote), [Tailscale](/gateway/tailscale), [Security](/gateway/security)
