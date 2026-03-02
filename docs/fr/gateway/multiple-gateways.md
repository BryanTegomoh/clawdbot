---
summary: "Exécuter plusieurs Gateways OpenClaw sur un même hôte (isolation, ports et profils)"
read_when:
  - Exécution de plus d'un Gateway sur la même machine
  - Vous avez besoin d'une configuration/état/ports isolés par Gateway
title: "Gateways multiples"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/multiple-gateways.md
  workflow: manual
---

# Gateways multiples (même hôte)

La plupart des installations devraient utiliser un seul Gateway car un Gateway unique peut gérer plusieurs connexions de messagerie et agents. Si vous avez besoin d'une isolation ou redondance plus forte (par ex. un bot de secours), exécutez des Gateways séparés avec des profils/ports isolés.

## Checklist d'isolation (requise)

- `OPENCLAW_CONFIG_PATH` : fichier de configuration par instance
- `OPENCLAW_STATE_DIR` : sessions, identifiants, caches par instance
- `agents.defaults.workspace` : racine workspace par instance
- `gateway.port` (ou `--port`) : unique par instance
- Les ports dérivés (browser/canvas) ne doivent pas se chevaucher

Si ceux-ci sont partagés, vous rencontrerez des courses de configuration et des conflits de ports.

## Recommandé : profils (`--profile`)

Les profils définissent automatiquement `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` et suffixent les noms de service.

```bash
# principal
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# secours
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Services par profil :

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## Guide du bot de secours

Exécutez un second Gateway sur le même hôte avec ses propres :

- profil/configuration
- répertoire d'état
- workspace
- port de base (plus les ports dérivés)

Cela garde le bot de secours isolé du bot principal afin qu'il puisse déboguer ou appliquer des changements de configuration si le bot principal est hors service.

Espacement des ports : laissez au moins 20 ports entre les ports de base pour que les ports dérivés browser/canvas/CDP ne se chevauchent jamais.

### Comment installer (bot de secours)

```bash
# Bot principal (existant ou nouveau, sans paramètre --profile)
# S'exécute sur le port 18789 + ports Chrome CDC/Canvas/...
openclaw onboard
openclaw gateway install

# Bot de secours (profil isolé + ports)
openclaw --profile rescue onboard
# Notes :
# - le nom du workspace sera postfixé avec -rescue par défaut
# - Le port doit être au moins 18789 + 20 ports,
#   mieux vaut choisir un port de base complètement différent, comme 19789,
# - le reste de l'onboarding est identique au normal

# Pour installer le service (si pas fait automatiquement pendant l'onboarding)
openclaw --profile rescue gateway install
```

## Mappage des ports (dérivés)

Port de base = `gateway.port` (ou `OPENCLAW_GATEWAY_PORT` / `--port`).

- port du service de contrôle browser = base + 2 (loopback uniquement)
- l'hôte canvas est servi sur le serveur HTTP du Gateway (même port que `gateway.port`)
- les ports CDP des profils browser s'auto-allouent depuis `browser.controlPort + 9 .. + 108`

Si vous remplacez l'un de ceux-ci dans la configuration ou l'environnement, vous devez les garder uniques par instance.

## Notes browser/CDP (piège courant)

- Ne fixez **pas** `browser.cdpUrl` aux mêmes valeurs sur plusieurs instances.
- Chaque instance a besoin de son propre port de contrôle browser et de sa plage CDP (dérivés de son port gateway).
- Si vous avez besoin de ports CDP explicites, définissez `browser.profiles.<name>.cdpPort` par instance.
- Chrome distant : utilisez `browser.profiles.<name>.cdpUrl` (par profil, par instance).

## Exemple env manuel

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## Vérifications rapides

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
