---
summary: "Configuration Docker optionnelle et configuration initiale pour OpenClaw"
read_when:
  - Vous souhaitez un gateway conteneurisé au lieu d'installations locales
  - Vous validez le flux Docker
title: "Docker"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/docker.md
  workflow: manual
---

# Docker (optionnel)

Docker est **optionnel**. Utilisez-le uniquement si vous souhaitez un gateway conteneurisé ou pour valider le flux Docker.

## Docker est-il fait pour moi ?

- **Oui** : vous voulez un environnement gateway isolé et jetable, ou exécuter OpenClaw sur un hôte sans installations locales.
- **Non** : vous êtes sur votre propre machine et voulez la boucle de développement la plus rapide. Utilisez plutôt le flux d'installation normal.
- **Note sur le bac à sable** : le bac à sable des agents utilisé aussi Docker, mais il ne nécessite **pas** que le gateway complet s'exécute dans Docker. Voir [Bac à sable](/gateway/sandboxing).

Ce guide couvre :

- Gateway conteneurisé (OpenClaw complet dans Docker)
- Bac à sable d'agent par session (gateway hôte + outils d'agent isolés dans Docker)

Détails du bac à sable : [Bac à sable](/gateway/sandboxing)

## Prérequis

- Docker Desktop (ou Docker Engine) + Docker Compose v2
- Au moins 2 Go de RAM pour la construction de l'image (`pnpm install` peut être tué par OOM sur les hôtes avec 1 Go, code de sortie 137)
- Suffisamment d'espace disque pour les images + journaux

## Gateway conteneurisé (Docker Compose)

### Démarrage rapide (recommandé)

Depuis la racine du dépôt :

```bash
./docker-setup.sh
```

Ce script :

- compile l'image du gateway
- exécute l'assistant de configuration initiale
- affiche des conseils optionnels de configuration des fournisseurs
- démarre le gateway via Docker Compose
- génère un jeton de gateway et l'écrit dans `.env`

Variables d'environnement optionnelles :

- `OPENCLAW_DOCKER_APT_PACKAGES` — installer des paquets apt supplémentaires lors de la compilation
- `OPENCLAW_EXTRA_MOUNTS` — ajouter des montages bind hôte supplémentaires
- `OPENCLAW_HOME_VOLUME` — persister `/home/node` dans un volume nommé

Après la fin du script :

- Ouvrez `http://127.0.0.1:18789/` dans votre navigateur.
- Collez le jeton dans l'interface de contrôle (Paramètres puis jeton).
- Besoin de l'URL à nouveau ? Exécutez `docker compose run --rm openclaw-cli dashboard --no-open`.

Il écrit la configuration/espace de travail sur l'hôte :

- `~/.openclaw/`
- `~/.openclaw/workspace`

Vous utilisez un VPS ? Voir [Hetzner (Docker VPS)](/install/hetzner).

### Assistants Shell (optionnel)

Pour une gestion quotidienne plus facile de Docker, installez `ClawDock` :

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**Ajoutez à votre configuration shell (zsh) :**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

Utilisez ensuite `clawdock-start`, `clawdock-stop`, `clawdock-dashboard`, etc. Exécutez `clawdock-help` pour toutes les commandes.

Voir le [README de l'assistant `ClawDock`](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) pour plus de détails.

### Flux manuel (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

Note : exécutez `docker compose ...` depuis la racine du dépôt. Si vous avez activé
`OPENCLAW_EXTRA_MOUNTS` ou `OPENCLAW_HOME_VOLUME`, le script de configuration écrit
`docker-compose.extra.yml` ; incluez-le lors de l'exécution de Compose ailleurs :

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Jeton de l'interface de contrôle + appairage (Docker)

Si vous voyez « unauthorized » ou « disconnected (1008): pairing required », récupérez un
lien dashboard récent et approuvez le périphérique navigateur :

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

Plus de détails : [Dashboard](/web/dashboard), [Périphériques](/cli/devices).

### Montages supplémentaires (optionnel)

Si vous souhaitez monter des répertoires hôte supplémentaires dans les conteneurs, définissez
`OPENCLAW_EXTRA_MOUNTS` avant d'exécuter `docker-setup.sh`. Cela accepte une
liste séparée par des virgules de montages bind Docker et les applique aux deux services
`openclaw-gateway` et `openclaw-cli` en générant `docker-compose.extra.yml`.

Exemple :

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes :

- Les chemins doivent être partagés avec Docker Desktop sur macOS/Windows.
- Chaque entrée doit être `source:cible[:options]` sans espaces, tabulations ou retours à la ligne.
- Si vous modifiez `OPENCLAW_EXTRA_MOUNTS`, relancez `docker-setup.sh` pour regénérer le
  fichier compose supplémentaire.
- `docker-compose.extra.yml` est généré. Ne le modifiez pas manuellement.

### Persister le répertoire home complet du conteneur (optionnel)

Si vous souhaitez que `/home/node` persiste entre les recréations de conteneurs, définissez un volume
nommé via `OPENCLAW_HOME_VOLUME`. Cela crée un volume Docker et le monte à
`/home/node`, tout en conservant les montages bind standard de configuration/espace de travail. Utilisez un
volume nommé ici (pas un chemin bind) ; pour les montages bind, utilisez
`OPENCLAW_EXTRA_MOUNTS`.

Exemple :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Vous pouvez combiner cela avec des montages supplémentaires :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes :

- Les volumes nommés doivent correspondre à `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
- Si vous changez `OPENCLAW_HOME_VOLUME`, relancez `docker-setup.sh` pour regénérer le
  fichier compose supplémentaire.
- Le volume nommé persiste jusqu'à sa suppression avec `docker volume rm <nom>`.

### Installer des paquets apt supplémentaires (optionnel)

Si vous avez besoin de paquets système dans l'image (par exemple, outils de compilation ou
bibliothèques média), définissez `OPENCLAW_DOCKER_APT_PACKAGES` avant d'exécuter `docker-setup.sh`.
Cela installe les paquets pendant la compilation de l'image, de sorte qu'ils persistent même si le
conteneur est supprimé.

Exemple :

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notes :

- Cela accepte une liste de noms de paquets apt séparés par des espaces.
- Si vous changez `OPENCLAW_DOCKER_APT_PACKAGES`, relancez `docker-setup.sh` pour recompiler
  l'image.

### Conteneur complet pour utilisateurs avancés (opt-in)

L'image Docker par défaut est orientée **sécurité** et s'exécute en tant qu'utilisateur non-root `node`.
Cela réduit la surface d'attaque, mais cela signifie :

- pas d'installation de paquets système au runtime
- pas de Homebrew par défaut
- pas de navigateurs Chromium/Playwright intégrés

Si vous voulez un conteneur plus complet, utilisez ces options opt-in :

1. **Persister `/home/node`** pour que les téléchargements de navigateurs et les caches d'outils survivent :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2. **Intégrer les dépendances système dans l'image** (reproductible + persistant) :

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3. **Installer les navigateurs Playwright sans `npx`** (évite les conflits de remplacement npm) :

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Si vous avez besoin que Playwright installe les dépendances système, recompilez l'image avec
`OPENCLAW_DOCKER_APT_PACKAGES` au lieu d'utiliser `--with-deps` au runtime.

4. **Persister les téléchargements de navigateurs Playwright** :

- Définissez `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` dans
  `docker-compose.yml`.
- Assurez-vous que `/home/node` persiste via `OPENCLAW_HOME_VOLUME`, ou montez
  `/home/node/.cache/ms-playwright` via `OPENCLAW_EXTRA_MOUNTS`.

### Permissions + EACCES

L'image s'exécute en tant que `node` (uid 1000). Si vous voyez des erreurs de permission sur
`/home/node/.openclaw`, assurez-vous que vos montages bind hôte appartiennent à l'uid 1000.

Exemple (hôte Linux) :

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

Si vous choisissez de lancer en tant que root par commodité, vous acceptez le compromis de sécurité.

### Recompilations plus rapides (recommandé)

Pour accélérer les recompilations, ordonnez votre Dockerfile de sorte que les couches de dépendances soient mises en cache.
Cela évite de relancer `pnpm install` sauf si les fichiers lock changent :

```dockerfile
FROM node:22-bookworm

# Installer Bun (nécessaire pour les scripts de compilation)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Mettre en cache les dépendances sauf si les métadonnées de paquet changent
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### Configuration des canaux (optionnel)

Utilisez le conteneur CLI pour configurer les canaux, puis redémarrez le gateway si nécessaire.

WhatsApp (QR) :

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (jeton bot) :

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (jeton bot) :

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Documentation : [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord)

### OpenAI Codex OAuth (Docker headless)

Si vous choisissez OpenAI Codex OAuth dans l'assistant, il ouvre une URL de navigateur et tente
de capturer un callback sur `http://127.0.0.1:1455/auth/callback`. Dans Docker ou
les configurations headless, ce callback peut afficher une erreur de navigateur. Copiez l'URL de redirection
complète sur laquelle vous arrivez et collez-la dans l'assistant pour terminer l'authentification.

### Vérification de santé

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### Test de bout en bout (Docker)

```bash
scripts/e2e/onboard-docker.sh
```

### Test d'import QR (Docker)

```bash
pnpm test:docker:qr
```

### Notes

- Le bind du Gateway est par défaut `lan` pour l'utilisation en conteneur.
- Le CMD du Dockerfile utilisé `--allow-unconfigured` ; une configuration montée avec `gateway.mode` différent de `local` démarrera quand même. Remplacez le CMD pour appliquer la restriction.
- Le conteneur gateway est la source de vérité pour les sessions (`~/.openclaw/agents/<agentId>/sessions/`).

## Bac à sable d'agent (gateway hôte + outils Docker)

Guide approfondi : [Bac à sable](/gateway/sandboxing)

### Fonctionnement

Lorsque `agents.defaults.sandbox` est activé, les **sessions non principales** exécutent les outils dans un conteneur
Docker. Le gateway reste sur votre hôte, mais l'exécution des outils est isolée :

- scope : `"agent"` par défaut (un conteneur + espace de travail par agent)
- scope : `"session"` pour une isolation par session
- dossier d'espace de travail par scope monté à `/workspace`
- accès optionnel à l'espace de travail de l'agent (`agents.defaults.sandbox.workspaceAccess`)
- politique d'outils autoriser/refuser (refuser l'emporte)
- les médias entrants sont copiés dans l'espace de travail du bac à sable actif (`média/inbound/*`) pour que les outils puissent les lire (avec `workspaceAccess: "rw"`, cela atterrit dans l'espace de travail de l'agent)

Avertissement : `scope: "shared"` désactive l'isolation inter-sessions. Toutes les sessions partagent
un conteneur et un espace de travail.

### Profils de bac à sable par agent (multi-agent)

Si vous utilisez le routage multi-agent, chaque agent peut remplacer les paramètres de bac à sable + outils :
`agents.list[].sandbox` et `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools`). Cela vous permet d'exécuter
des niveaux d'accès mixtes dans un seul gateway :

- Accès complet (agent personnel)
- Outils en lecture seule + espace de travail en lecture seule (agent familial/professionnel)
- Pas d'outils système de fichiers/shell (agent public)

Voir [Bac à sable et outils multi-agents](/tools/multi-agent-sandbox-tools) pour des exemples,
la priorité et le dépannage.

### Comportement par défaut

- Image : `openclaw-sandbox:bookworm-slim`
- Un conteneur par agent
- Accès à l'espace de travail de l'agent : `workspaceAccess: "none"` (par défaut) utilisé `~/.openclaw/sandboxes`
  - `"ro"` garde l'espace de travail du bac à sable à `/workspace` et monte l'espace de travail de l'agent en lecture seule à `/agent` (désactive `write`/`edit`/`apply_patch`)
  - `"rw"` monte l'espace de travail de l'agent en lecture/écriture à `/workspace`
- Nettoyage automatique : inactif > 24h OU âge > 7j
- Réseau : `none` par défaut (activez explicitement si vous avez besoin de sortie réseau)
  - `host` est bloqué.
  - `container:<id>` est bloqué par défaut (risque de jonction d'espace de noms).
- Autoriser par défaut : `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- Refuser par défaut : `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### Activer le bac à sable

Si vous prévoyez d'installer des paquets dans `setupCommand`, notez :

- Le `docker.network` par défaut est `"none"` (pas de sortie réseau).
- `docker.network: "host"` est bloqué.
- `docker.network: "container:<id>"` est bloqué par défaut.
- Dérogation d'urgence : `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.
- `readOnlyRoot: true` bloque les installations de paquets.
- `user` doit être root pour `apt-get` (omettez `user` ou définissez `user: "0:0"`).
  OpenClaw recrée automatiquement les conteneurs lorsque `setupCommand` (ou la configuration docker) change
  sauf si le conteneur a été **récemment utilisé** (dans les ~5 dernières minutes). Les conteneurs actifs
  journalisent un avertissement avec la commande exacte `openclaw sandbox recreate ...`.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent est la valeur par défaut)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 désactive le nettoyage par inactivité
          maxAgeDays: 7, // 0 désactive le nettoyage par âge maximum
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

Les paramètres de durcissement se trouvent sous `agents.defaults.sandbox.docker` :
`network`, `user`, `pidsLimit`, `memory`, `memorySwap`, `cpus`, `ulimits`,
`seccompProfile`, `apparmorProfile`, `dns`, `extraHosts`,
`dangerouslyAllowContainerNamespaceJoin` (urgence uniquement).

Multi-agent : remplacez `agents.defaults.sandbox.{docker,browser,prune}.*` par agent via `agents.list[].sandbox.{docker,browser,prune}.*`
(ignoré quand `agents.defaults.sandbox.scope` / `agents.list[].sandbox.scope` est `"shared"`).

### Compiler l'image de bac à sable par défaut

```bash
scripts/sandbox-setup.sh
```

Cela compile `openclaw-sandbox:bookworm-slim` en utilisant `Dockerfile.sandbox`.

### Image commune de bac à sable (optionnel)

Si vous voulez une image de bac à sable avec des outils de compilation courants (Node, Go, Rust, etc.), compilez l'image commune :

```bash
scripts/sandbox-common-setup.sh
```

Cela compile `openclaw-sandbox-common:bookworm-slim`. Pour l'utiliser :

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "openclaw-sandbox-common:bookworm-slim" } },
    },
  },
}
```

### Image de bac à sable avec navigateur

Pour exécuter l'outil navigateur dans le bac à sable, compilez l'image navigateur :

```bash
scripts/sandbox-browser-setup.sh
```

Cela compile `openclaw-sandbox-browser:bookworm-slim` en utilisant
`Dockerfile.sandbox-browser`. Le conteneur exécute Chromium avec CDP activé et
un observateur noVNC optionnel (mode graphique via Xvfb).

Notes :

- Le mode graphique (Xvfb) réduit le blocage des bots par rapport au mode headless.
- Le mode headless peut toujours être utilisé en définissant `agents.defaults.sandbox.browser.headless=true`.
- Aucun environnement de bureau complet (GNOME) n'est nécessaire ; Xvfb fournit l'affichage.
- Les conteneurs navigateur utilisent par défaut un réseau Docker dédié (`openclaw-sandbox-browser`) au lieu du `bridge` global.
- L'option `agents.defaults.sandbox.browser.cdpSourceRange` restreint l'ingress CDP en bordure de conteneur par CIDR (par exemple `172.21.0.1/32`).
- L'accès à l'observateur noVNC est protégé par mot de passe par défaut ; OpenClaw fournit une URL de jeton d'observateur à durée limitée au lieu de partager le mot de passe brut dans l'URL.

Configuration d'utilisation :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: { enabled: true },
      },
    },
  },
}
```

Image navigateur personnalisée :

```json5
{
  agents: {
    defaults: {
      sandbox: { browser: { image: "my-openclaw-browser" } },
    },
  },
}
```

Lorsqu'il est activé, l'agent reçoit :

- une URL de contrôle du navigateur du bac à sable (pour l'outil `browser`)
- une URL noVNC (si activé et headless=false)

N'oubliez pas : si vous utilisez une liste d'autorisation pour les outils, ajoutez `browser` (et retirez-le de
deny) sinon l'outil reste bloqué.
Les règles de nettoyage (`agents.defaults.sandbox.prune`) s'appliquent aussi aux conteneurs navigateur.

### Image de bac à sable personnalisée

Compilez votre propre image et pointez la configuration vers elle :

```bash
docker build -t my-openclaw-sbx -f Dockerfile.sandbox .
```

```json5
{
  agents: {
    defaults: {
      sandbox: { docker: { image: "my-openclaw-sbx" } },
    },
  },
}
```

### Politique d'outils (autoriser/refuser)

- `deny` l'emporte sur `allow`.
- Si `allow` est vide : tous les outils (sauf deny) sont disponibles.
- Si `allow` est non vide : seuls les outils dans `allow` sont disponibles (moins deny).

### Stratégie de nettoyage

Deux paramètres :

- `prune.idleHours` : supprimer les conteneurs non utilisés depuis X heures (0 = désactiver)
- `prune.maxAgeDays` : supprimer les conteneurs plus anciens que X jours (0 = désactiver)

Exemple :

- Garder les sessions activés mais limiter la durée de vie :
  `idleHours: 24`, `maxAgeDays: 7`
- Ne jamais nettoyer :
  `idleHours: 0`, `maxAgeDays: 0`

### Notes de sécurité

- La barrière stricte ne s'applique qu'aux **outils** (exec/read/write/edit/apply_patch).
- Les outils réservés à l'hôte comme browser/camera/canvas sont bloqués par défaut.
- Autoriser `browser` dans le bac à sable **casse l'isolation** (le navigateur s'exécute sur l'hôte).

## Dépannage

- Image manquante : compilez avec [`scripts/sandbox-setup.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/sandbox-setup.sh) ou définissez `agents.defaults.sandbox.docker.image`.
- Conteneur non démarré : il sera créé automatiquement par session à la demande.
- Erreurs de permission dans le bac à sable : définissez `docker.user` à un UID:GID correspondant à
  la propriété de votre espace de travail monté (ou faites un chown du dossier d'espace de travail).
- Outils personnalisés introuvables : OpenClaw exécute les commandes avec `sh -lc` (shell de connexion), qui
  source `/etc/profile` et peut réinitialiser PATH. Définissez `docker.env.PATH` pour préfixer vos
  chemins d'outils personnalisés (par ex., `/custom/bin:/usr/local/share/npm-global/bin`), ou ajoutez
  un script sous `/etc/profile.d/` dans votre Dockerfile.
