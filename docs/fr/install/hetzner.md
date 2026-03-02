---
summary: "Exécuter OpenClaw Gateway 24/7 sur un VPS Hetzner bon marché (Docker) avec état durable et binaires intégrés"
read_when:
  - Vous voulez OpenClaw fonctionnant 24/7 sur un VPS cloud (pas votre laptop)
  - Vous voulez un Gateway de production, toujours actif, sur votre propre VPS
  - Vous voulez un contrôle total sur la persistance, les binaires et le comportement de redémarrage
  - Vous exécutez OpenClaw dans Docker sur Hetzner ou un fournisseur similaire
title: "Hetzner"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/hetzner.md
  workflow: manual
---

# OpenClaw sur Hetzner (Docker, guide VPS production)

## Objectif

Exécuter un Gateway OpenClaw persistant sur un VPS Hetzner avec Docker, un état durable, des binaires intégrés et un comportement de redémarrage sûr.

Si vous voulez « OpenClaw 24/7 pour ~5 $ », c'est la configuration fiable la plus simple.
Les tarifs Hetzner changent ; choisissez le plus petit VPS Debian/Ubuntu et augmentez si vous rencontrez des OOM.

Rappel sur le modèle de sécurité :

- Les agents partagés en entreprise sont acceptables quand tout le monde est dans la même frontière de confiance et que le runtime est professionnel uniquement.
- Maintenez une séparation stricte : VPS/runtime dédié + comptes dédiés ; pas de profils personnels Apple/Google/navigateur/gestionnaire de mots de passe sur cet hôte.
- Si les utilisateurs sont adversaires entre eux, séparez par gateway/hôte/utilisateur OS.

Voir [Sécurité](/gateway/security) et [Hébergement VPS](/vps).

## En termes simples, que faisons-nous ?

- Louer un petit serveur Linux (VPS Hetzner)
- Installer Docker (runtime applicatif isolé)
- Démarrer le Gateway OpenClaw dans Docker
- Persister `~/.openclaw` + `~/.openclaw/workspace` sur l'hôte (survit aux redémarrages/recompilations)
- Accéder à l'interface de contrôle depuis votre laptop via un tunnel SSH

Le Gateway est accessible via :

- Redirection de port SSH depuis votre laptop
- Exposition directe du port si vous gérez le pare-feu et les jetons vous-même

Ce guide suppose Ubuntu ou Debian sur Hetzner.
Si vous êtes sur un autre VPS Linux, adaptez les paquets en conséquence.
Pour le flux Docker générique, voir [Docker](/install/docker).

---

## Chemin rapide (opérateurs expérimentés)

1. Provisionner un VPS Hetzner
2. Installer Docker
3. Cloner le dépôt OpenClaw
4. Créer les répertoires persistants sur l'hôte
5. Configurer `.env` et `docker-compose.yml`
6. Intégrer les binaires requis dans l'image
7. `docker compose up -d`
8. Vérifier la persistance et l'accès au Gateway

---

## Ce dont vous avez besoin

- VPS Hetzner avec accès root
- Accès SSH depuis votre laptop
- Aisance de base avec SSH + copier/coller
- ~20 minutes
- Docker et Docker Compose
- Identifiants d'authentification modèle
- Identifiants de fournisseur optionnels
  - WhatsApp QR
  - Jeton bot Telegram
  - Gmail OAuth

---

## 1) Provisionner le VPS

Créez un VPS Ubuntu ou Debian chez Hetzner.

Connectez-vous en tant que root :

```bash
ssh root@YOUR_VPS_IP
```

Ce guide suppose que le VPS est avec état.
Ne le traitez pas comme une infrastructure jetable.

---

## 2) Installer Docker (sur le VPS)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Vérifiez :

```bash
docker --version
docker compose version
```

---

## 3) Cloner le dépôt OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Ce guide suppose que vous allez compiler une image personnalisée pour garantir la persistance des binaires.

---

## 4) Créer les répertoires persistants sur l'hôte

Les conteneurs Docker sont éphémères.
Tout état à longue durée de vie doit résider sur l'hôte.

```bash
mkdir -p /root/.openclaw/workspace

# Définir la propriété pour l'utilisateur du conteneur (uid 1000) :
chown -R 1000:1000 /root/.openclaw
```

---

## 5) Configurer les variables d'environnement

Créez `.env` à la racine du dépôt.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Générez des secrets forts :

```bash
openssl rand -hex 32
```

**Ne committez pas ce fichier.**

---

## 6) Configuration Docker Compose

Créez ou mettez à jour `docker-compose.yml`.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommandé : garder le Gateway en loopback uniquement sur le VPS ; accéder via tunnel SSH.
      # Pour l'exposer publiquement, supprimez le préfixe `127.0.0.1:` et configurez le pare-feu en conséquence.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured` est uniquement pour la commodité du bootstrap, ce n'est pas un remplacement pour une configuration gateway appropriée. Définissez toujours l'authentification (`gateway.auth.token` ou mot de passe) et utilisez des paramètres de bind sûrs pour votre déploiement.

---

## 7) Intégrer les binaires requis dans l'image (critique)

Installer des binaires dans un conteneur en cours d'exécution est un piège.
Tout ce qui est installé au runtime sera perdu au redémarrage.

Tous les binaires externes requis par les skills doivent être installés lors de la compilation de l'image.

Les exemples ci-dessous montrent trois binaires courants uniquement :

- `gog` pour l'accès Gmail
- `goplaces` pour Google Places
- `wacli` pour WhatsApp

Ce sont des exemples, pas une liste complète.
Vous pouvez installer autant de binaires que nécessaire en suivant le même modèle.

Si vous ajoutez de nouveaux skills plus tard qui dépendent de binaires supplémentaires, vous devez :

1. Mettre à jour le Dockerfile
2. Recompiler l'image
3. Redémarrer les conteneurs

**Exemple de Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Exemple de binaire 1 : CLI Gmail
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Exemple de binaire 2 : CLI Google Places
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Exemple de binaire 3 : CLI WhatsApp
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Ajoutez d'autres binaires ci-dessous en suivant le même modèle

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

---

## 8) Compiler et lancer

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Vérifier les binaires :

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Sortie attendue :

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

---

## 9) Vérifier le Gateway

```bash
docker compose logs -f openclaw-gateway
```

Succès :

```
[gateway] listening on ws://0.0.0.0:18789
```

Depuis votre laptop :

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Ouvrez :

`http://127.0.0.1:18789/`

Collez votre jeton gateway.

---

## Ce qui persiste où (source de vérité)

OpenClaw s'exécute dans Docker, mais Docker n'est pas la source de vérité.
Tout état à longue durée de vie doit survivre aux redémarrages, recompilations et reboots.

| Composant           | Emplacement                       | Mécanisme de persistance | Notes                            |
| ------------------- | --------------------------------- | ------------------------ | -------------------------------- |
| Config gateway      | `/home/node/.openclaw/`           | Montage volume hôte      | Inclut `openclaw.json`, jetons   |
| Profils auth modèle | `/home/node/.openclaw/`           | Montage volume hôte      | Jetons OAuth, clés API           |
| Config skills       | `/home/node/.openclaw/skills/`    | Montage volume hôte      | État au niveau skill             |
| Espace de travail   | `/home/node/.openclaw/workspace/` | Montage volume hôte      | Code et artefacts agent          |
| Session WhatsApp    | `/home/node/.openclaw/`           | Montage volume hôte      | Préserve la connexion QR         |
| Trousseau Gmail     | `/home/node/.openclaw/`           | Volume hôte + mot de passe | Nécessite `GOG_KEYRING_PASSWORD` |
| Binaires externes   | `/usr/local/bin/`                 | Image Docker             | Doivent être intégrés à la compilation |
| Runtime Node        | Système de fichiers conteneur     | Image Docker             | Recompilé à chaque build d'image |
| Paquets OS          | Système de fichiers conteneur     | Image Docker             | Ne pas installer au runtime      |
| Conteneur Docker    | Éphémère                          | Redémarrable             | Peut être détruit en toute sécurité |

---

## Infrastructure as Code (Terraform)

Pour les équipes préférant des workflows infrastructure-as-code, une configuration Terraform maintenue par la communauté fournit :

- Configuration Terraform modulaire avec gestion d'état distant
- Provisionnement automatisé via cloud-init
- Scripts de déploiement (bootstrap, déploiement, sauvegarde/restauration)
- Durcissement de sécurité (pare-feu, UFW, accès SSH uniquement)
- Configuration de tunnel SSH pour l'accès au gateway

**Dépôts :**

- Infrastructure : [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
- Config Docker : [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

Cette approche complète la configuration Docker ci-dessus avec des déploiements reproductibles, une infrastructure versionnée et une reprise après sinistre automatisée.

> **Note :** Maintenu par la communauté. Pour les problèmes ou contributions, voir les liens des dépôts ci-dessus.
