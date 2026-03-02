---
title: "Bac à sable"
summary: "Fonctionnement du bac à sable OpenClaw : modes, scopes, accès à l'espace de travail et images"
read_when: "Vous voulez une explication dédiée du bac à sable ou devez ajuster agents.defaults.sandbox."
status: activé
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/sandboxing.md
  workflow: manual
---

# Bac à sable

OpenClaw peut exécuter les **outils dans des conteneurs Docker** pour réduire le rayon d'impact.
C'est **optionnel** et contrôle par la configuration (`agents.defaults.sandbox` ou
`agents.list[].sandbox`). Si le bac à sable est désactivé, les outils s'exécutent sur l'hôte.
Le Gateway reste sur l'hôte ; l'exécution des outils se fait dans un bac à sable isolé
lorsqu'il est activé.

Ce n'est pas une frontière de sécurité parfaite, mais cela limite matériellement l'accès
au système de fichiers et aux processus lorsque le modèle fait quelque chose d'inapproprié.

## Ce qui est mis en bac à sable

- L'exécution des outils (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, etc.).
- Le navigateur en bac à sable optionnel (`agents.defaults.sandbox.browser`).
  - Par défaut, le navigateur en bac à sable démarre automatiquement (assuré que le CDP est accessible) lorsque l'outil navigateur en a besoin.
    Configurable via `agents.defaults.sandbox.browser.autoStart` et `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
  - Par défaut, les conteneurs de navigateur en bac à sable utilisent un réseau Docker dédié (`openclaw-sandbox-browser`) au lieu du réseau `bridge` global.
    Configurable avec `agents.defaults.sandbox.browser.network`.
  - L'option `agents.defaults.sandbox.browser.cdpSourceRange` restreint l'ingress CDP au bord du conteneur avec une liste d'autorisation CIDR (par exemple `172.21.0.1/32`).
  - L'accès observateur noVNC est protégé par mot de passe par défaut ; OpenClaw émet une URL de jeton à durée limitée qui résout vers la session observateur.
  - `agents.defaults.sandbox.browser.allowHostControl` permet aux sessions en bac à sable de cibler explicitement le navigateur de l'hôte.
  - Des listes d'autorisation optionnelles contrôlent `target: "custom"` : `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Non mis en bac à sable :

- Le processus Gateway lui-même.
- Tout outil explicitement autorisé à s'exécuter sur l'hôte (ex. `tools.elevated`).
  - **L'exécution en mode élevé s'exécute sur l'hôte et contourne le bac à sable.**
  - Si le bac à sable est désactivé, `tools.elevated` ne change pas l'exécution (déjà sur l'hôte). Voir [Mode élevé](/tools/elevated).

## Modes

`agents.defaults.sandbox.mode` contrôle **quand** le bac à sable est utilisé :

- `"off"` : pas de bac à sable.
- `"non-main"` : bac à sable uniquement pour les sessions **non-principales** (par défaut si vous voulez des discussions normales sur l'hôte).
- `"all"` : chaque session s'exécute dans un bac à sable.
  Note : `"non-main"` est basé sur `session.mainKey` (par défaut `"main"`), pas sur l'identifiant de l'agent.
  Les sessions de groupe/canal utilisent leurs propres clés, elles sont donc considérées comme non-principales et seront mises en bac à sable.

## Scope

`agents.defaults.sandbox.scope` contrôle **combien de conteneurs** sont créés :

- `"session"` (par défaut) : un conteneur par session.
- `"agent"` : un conteneur par agent.
- `"shared"` : un conteneur partagé par toutes les sessions en bac à sable.

## Accès à l'espace de travail

`agents.defaults.sandbox.workspaceAccess` contrôle **ce que le bac à sable peut voir** :

- `"none"` (par défaut) : les outils voient un espace de travail bac à sable sous `~/.openclaw/sandboxes`.
- `"ro"` : monte l'espace de travail de l'agent en lecture seule à `/agent` (désactive `write`/`edit`/`apply_patch`).
- `"rw"` : monte l'espace de travail de l'agent en lecture/écriture à `/workspace`.

Les médias entrants sont copiés dans l'espace de travail actif du bac à sable (`média/inbound/*`).
Note sur les compétences : l'outil `read` est rooté au bac à sable. Avec `workspaceAccess: "none"`,
OpenClaw copie les compétences éligibles dans l'espace de travail du bac à sable (`.../skills`) pour
qu'elles puissent être lues. Avec `"rw"`, les compétences de l'espace de travail sont lisibles depuis
`/workspace/skills`.

## Montages de liaison personnalisés

`agents.defaults.sandbox.docker.binds` monte des répertoires hôte supplémentaires dans le conteneur.
Format : `hôte:conteneur:mode` (ex. `"/home/user/source:/source:rw"`).

Les montages globaux et par agent sont **fusionnés** (pas remplacés). Avec `scope: "shared"`, les montages par agent sont ignorés.

`agents.defaults.sandbox.browser.binds` monte des répertoires hôte supplémentaires dans le conteneur du **navigateur en bac à sable** uniquement.

- Lorsqu'il est défini (y compris `[]`), il remplace `agents.defaults.sandbox.docker.binds` pour le conteneur du navigateur.
- Lorsqu'il est omis, le conteneur du navigateur revient à `agents.defaults.sandbox.docker.binds` (rétro-compatible).

Exemple (source en lecture seule + un répertoire de données supplémentaire) :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

Notes de sécurité :

- Les montages contournent le système de fichiers du bac à sable : ils exposent les chemins de l'hôte avec le mode que vous définissez (`:ro` ou `:rw`).
- OpenClaw bloque les sources de montage dangereuses (par exemple : `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev` et les montages parents qui les exposeraient).
- Les montages sensibles (secrets, clés SSH, identifiants de service) devraient être `:ro` sauf si absolument nécessaire.
- Combinez avec `workspaceAccess: "ro"` si vous n'avez besoin que d'un accès en lecture à l'espace de travail ; les modes de montage restent indépendants.
- Voir [Bac à sable vs politique d'outils vs mode élevé](/gateway/sandbox-vs-tool-policy-vs-elevated) pour l'interaction entre les montages, la politique d'outils et l'exécution en mode élevé.

## Images et configuration

Image par défaut : `openclaw-sandbox:bookworm-slim`

Construisez-la une fois :

```bash
scripts/sandbox-setup.sh
```

Note : l'image par défaut n'inclut **pas** Node. Si une compétence nécessite Node (ou
d'autres environnements d'exécution), soit créez une image personnalisée soit installez via
`sandbox.docker.setupCommand` (nécessite un accès réseau sortant + root en écriture +
utilisateur root).

Image du navigateur en bac à sable :

```bash
scripts/sandbox-browser-setup.sh
```

Par défaut, les conteneurs en bac à sable fonctionnent **sans réseau**.
Remplacez avec `agents.defaults.sandbox.docker.network`.

Paramètres de sécurité par défaut :

- `network: "host"` est bloqué.
- `network: "container:<id>"` est bloqué par défaut (risque de contournement par jonction d'espace de noms).
- Override d'urgence : `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.

Les installations Docker et le gateway conteneurisé sont ici :
[Docker](/install/docker)

## setupCommand (configuration unique du conteneur)

`setupCommand` s'exécute **une seule fois** après la création du conteneur du bac à sable (pas à chaque exécution).
Il s'exécute à l'intérieur du conteneur via `sh -lc`.

Chemins :

- Global : `agents.defaults.sandbox.docker.setupCommand`
- Par agent : `agents.list[].sandbox.docker.setupCommand`

Pièges courants :

- Le `docker.network` par défaut est `"none"` (pas de sortie), donc les installations de paquets échoueront.
- `docker.network: "container:<id>"` nécessite `dangerouslyAllowContainerNamespaceJoin: true` et est réservé aux urgences.
- `readOnlyRoot: true` empêche les écritures ; définissez `readOnlyRoot: false` ou créez une image personnalisée.
- `user` doit être root pour les installations de paquets (omettez `user` ou définissez `user: "0:0"`).
- L'exécution dans le bac à sable n'hérite **pas** du `process.env` de l'hôte. Utilisez
  `agents.defaults.sandbox.docker.env` (ou une image personnalisée) pour les clés API de compétences.

## Politique d'outils et voies de sortie

Les politiques d'autorisation/refus des outils s'appliquent toujours avant les règles du bac à sable. Si un outil est refusé
globalement ou par agent, le bac à sable ne le réactive pas.

`tools.elevated` est une voie de sortie explicite qui exécute `exec` sur l'hôte.
Les directives `/exec` ne s'appliquent que pour les expéditeurs autorisés et persistent par session ; pour désactiver définitivement
`exec`, utilisez la politique de refus d'outils (voir [Bac à sable vs politique d'outils vs mode élevé](/gateway/sandbox-vs-tool-policy-vs-elevated)).

Débogage :

- Utilisez `openclaw sandbox explain` pour inspecter le mode effectif du bac à sable, la politique d'outils et les clés de configuration de correction.
- Voir [Bac à sable vs politique d'outils vs mode élevé](/gateway/sandbox-vs-tool-policy-vs-elevated) pour le modèle mental « pourquoi est-ce bloque ? ».
  Gardez-le verrouillé.

## Surcharges multi-agent

Chaque agent peut surcharger le bac à sable et les outils :
`agents.list[].sandbox` et `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools` pour la politique d'outils du bac à sable).
Voir [Bac à sable et outils multi-agent](/tools/multi-agent-sandbox-tools) pour la priorité.

## Exemple minimal d'activation

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## Documentation connexe

- [Configuration du bac à sable](/gateway/configuration#agentsdefaults-sandbox)
- [Bac à sable et outils multi-agent](/tools/multi-agent-sandbox-tools)
- [Sécurité](/gateway/security)
