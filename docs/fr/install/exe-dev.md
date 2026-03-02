---
summary: "Exécuter OpenClaw Gateway sur exe.dev (VM + proxy HTTPS) pour l'accès distant"
read_when:
  - Vous voulez un hôte Linux bon marché toujours allumé pour le Gateway
  - Vous voulez un accès distant à l'interface de contrôle sans gérer votre propre VPS
title: "exe.dev"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/exe-dev.md
  workflow: manual
---

# exe.dev

Objectif : OpenClaw Gateway fonctionnant sur une VM exe.dev, accessible depuis votre laptop via : `https://<vm-name>.exe.xyz`

Cette page suppose l'image par défaut **exeuntu** de exe.dev. Si vous avez choisi une distribution différente, adaptez les paquets en conséquence.

## Chemin rapide pour débutants

1. [https://exe.new/openclaw](https://exe.new/openclaw)
2. Renseignez votre clé d'authentification/jeton selon les besoins
3. Cliquez sur « Agent » à côté de votre VM, et patientez...
4. ???
5. Profit

## Ce dont vous avez besoin

- Compte exe.dev
- Accès `ssh exe.dev` aux machines virtuelles [exe.dev](https://exe.dev) (optionnel)

## Installation automatisée avec Shelley

Shelley, l'agent de [exe.dev](https://exe.dev), peut installer OpenClaw instantanément avec notre
prompt. Le prompt utilisé est le suivant :

```
Set up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by "openclaw devices list" and "openclaw devices approve <request id>". Make sure the dashboard shows that OpenClaw's health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final "reachable" should be <vm-name>.exe.xyz, without port specification.
```

## Installation manuelle

## 1) Créer la VM

Depuis votre appareil :

```bash
ssh exe.dev new
```

Puis connectez-vous :

```bash
ssh <vm-name>.exe.xyz
```

Astuce : gardez cette VM **avec état**. OpenClaw stocke l'état sous `~/.openclaw/` et `~/.openclaw/workspace/`.

## 2) Installer les prérequis (sur la VM)

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

## 3) Installer OpenClaw

Exécutez le script d'installation OpenClaw :

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 4) Configurer nginx pour rediriger OpenClaw via un proxy vers le port 8000

Éditez `/etc/nginx/sites-enabled/default` avec

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # Support WebSocket
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # En-têtes proxy standard
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Paramètres de timeout pour les connexions longue durée
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

## 5) Accéder à OpenClaw et accorder les privilèges

Accédez à `https://<vm-name>.exe.xyz/` (voir la sortie de l'interface de contrôle lors de la configuration initiale). Si une authentification est demandée, collez le
jeton de `gateway.auth.token` sur la VM (récupérez avec `openclaw config get gateway.auth.token`, ou générez-en un
avec `openclaw doctor --generate-gateway-token`). Approuvez les appareils avec `openclaw devices list` et
`openclaw devices approve <requestId>`. En cas de doute, utilisez Shelley depuis votre navigateur !

## Accès distant

L'accès distant est géré par l'authentification de [exe.dev](https://exe.dev). Par
défaut, le trafic HTTP depuis le port 8000 est redirigé vers `https://<vm-name>.exe.xyz`
avec authentification par email.

## Mise à jour

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Guide : [Mise à jour](/install/updating)
