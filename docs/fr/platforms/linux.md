---
summary: "Support Linux + statut de l'application compagnon"
read_when:
  - Recherche du statut de l'application compagnon Linux
  - Planification de la couverture des plateformes ou des contributions
title: "Application Linux"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/linux.md
  workflow: manual
---

# Application Linux

Le Gateway est pleinement supporte sur Linux. **Node est le runtime recommandé**.
Bun n'est pas recommandé pour le Gateway (bugs WhatsApp/Telegram).

Des applications compagnon Linux natives sont prevues. Les contributions sont les bienvenues si vous souhaitez aider a en construire une.

## Chemin rapide pour debutants (VPS)

1. Installez Node 22+
2. `npm i -g openclaw@latest`
3. `openclaw onboard --install-daemon`
4. Depuis votre ordinateur portable : `ssh -N -L 18789:127.0.0.1:18789 <user>@<host>`
5. Ouvrez `http://127.0.0.1:18789/` et collez votre jeton

Guide VPS étape par étape : [exe.dev](/install/exe-dev)

## Installation

- [Premiers pas](/start/getting-started)
- [Installation et mises a jour](/install/updating)
- Flux optionnels : [Bun (experimental)](/install/bun), [Nix](/install/nix), [Docker](/install/docker)

## Gateway

- [Runbook Gateway](/gateway)
- [Configuration](/gateway/configuration)

## Installation du service Gateway (CLI)

Utilisez l'une de ces méthodes :

```
openclaw onboard --install-daemon
```

Ou :

```
openclaw gateway install
```

Ou :

```
openclaw configure
```

Selectionnez **Gateway service** lorsque vous y etes invite.

Réparation/migration :

```
openclaw doctor
```

## Contrôle système (unite utilisateur systemd)

OpenClaw installe un service **utilisateur** systemd par défaut. Utilisez un service
**système** pour les serveurs partages ou toujours allumes. L'exemple complet de l'unite et les conseils
se trouvent dans le [Runbook Gateway](/gateway).

Configuration minimale :

Creez `~/.config/systemd/user/openclaw-gateway[-<profile>].service` :

```
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Activez-le :

```
systemctl --user enable --now openclaw-gateway[-<profile>].service
```
