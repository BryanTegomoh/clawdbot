---
summary: "Guide opérationnel du service Gateway, cycle de vie et opérations"
read_when:
  - Exécution ou débogage du processus gateway
title: "Guide opérationnel Gateway"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/index.md
  workflow: manual
---

# Guide opérationnel Gateway

Utilisez cette page pour le démarrage initial et les opérations courantes du service Gateway.

<CardGroup cols={2}>
  <Card title="Dépannage approfondi" icon="siren" href="/gateway/troubleshooting">
    Diagnostics par symptôme avec séquences de commandes précises et signatures de logs.
  </Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Guide de configuration orienté tâches + référence complète de configuration.
  </Card>
  <Card title="Gestion des secrets" icon="key-round" href="/gateway/secrets">
    Contrat SecretRef, comportement de l'instantane d'exécution et opérations de migration/rechargement.
  </Card>
  <Card title="Contrat du plan de secrets" icon="shield-check" href="/gateway/secrets-plan-contract">
    Règles exactes de cible/chemin de `secrets apply` et comportement de profil d'authentification ref-only.
  </Card>
</CardGroup>

## Démarrage local en 5 minutes

<Steps>
  <Step title="Démarrer le Gateway">

```bash
openclaw gateway --port 18789
# debug/trace redirigé vers stdio
openclaw gateway --port 18789 --verbose
# forcer l'arrêt du listener sur le port sélectionné, puis démarrer
openclaw gateway --force
```

  </Step>

  <Step title="Vérifier l'état du service">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

État nominal : `Runtime: running` et `RPC probe: ok`.

  </Step>

  <Step title="Valider la disponibilité des canaux">

```bash
openclaw channels status --probe
```

  </Step>
</Steps>

<Note>
Le rechargement de la configuration du Gateway surveille le chemin du fichier de configuration actif (résolu à partir des valeurs par défaut du profil/état, ou `OPENCLAW_CONFIG_PATH` lorsqu'il est défini).
Le mode par défaut est `gateway.reload.mode="hybrid"`.
</Note>

## Modèle d'exécution

- Un processus permanent pour le routage, le plan de contrôle et les connexions de canaux.
- Un port unique multiplexé pour :
  - WebSocket contrôle/RPC
  - API HTTP (compatible OpenAI, Responses, invocation d'outils)
  - Interface de contrôle et hooks
- Mode de liaison par défaut : `loopback`.
- L'authentification est requise par défaut (`gateway.auth.token` / `gateway.auth.password`, ou `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Précédence du port et du mode de liaison

| Paramètre    | Ordre de résolution                                           |
| ------------ | ------------------------------------------------------------- |
| Port Gateway | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Mode de liaison | CLI/override → `gateway.bind` → `loopback`                 |

### Modes de rechargement à chaud

| `gateway.reload.mode`    | Comportement                                      |
| ------------------------ | ------------------------------------------------- |
| `off`                    | Pas de rechargement de configuration              |
| `hot`                    | Appliquer uniquement les modifications hot-safe   |
| `restart`                | Redémarrer lors de modifications nécessitant un redémarrage |
| `hybrid` (par défaut)    | Application à chaud si possible, redémarrage si nécessaire |

## Ensemble de commandes opérateur

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## Accès distant

Recommandé : Tailscale/VPN.
Alternative : tunnel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Ensuite, connectez les clients à `ws://127.0.0.1:18789` en local.

<Warning>
Si l'authentification du Gateway est configurée, les clients doivent toujours envoyer les identifiants (`token`/`password`) même via des tunnels SSH.
</Warning>

Voir : [Gateway distant](/gateway/remote), [Authentification](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Supervision et cycle de vie du service

Utilisez des exécutions supervisées pour une fiabilité de niveau production.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

Les labels LaunchAgent sont `ai.openclaw.gateway` (par défaut) ou `ai.openclaw.<profile>` (profil nommé). `openclaw doctor` audite et corrige les dérives de configuration du service.

  </Tab>

  <Tab title="Linux (systemd utilisateur)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Pour la persistance après déconnexion, activez le lingering :

```bash
sudo loginctl enable-linger <user>
```

  </Tab>

  <Tab title="Linux (service système)">

Utilisez une unité système pour les hôtes multi-utilisateurs/toujours actifs.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  </Tab>
</Tabs>

## Plusieurs gateways sur un même hôte

La plupart des configurations devraient exécuter **un seul** Gateway.
Utilisez-en plusieurs uniquement pour une isolation/redondance stricte (par exemple un profil de secours).

Liste de vérification par instance :

- `gateway.port` unique
- `OPENCLAW_CONFIG_PATH` unique
- `OPENCLAW_STATE_DIR` unique
- `agents.defaults.workspace` unique

Exemple :

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Voir : [Plusieurs gateways](/gateway/multiple-gateways).

### Chemin rapide pour profil de développement

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Les valeurs par défaut incluent un état/configuration isolé et le port Gateway de base `19001`.

## Référence rapide du protocole (vue opérateur)

- La première trame client doit être `connect`.
- Le Gateway renvoie un instantané `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, limités/politique).
- Requêtes : `req(method, params)` → `res(ok/payload|error)`.
- Événements courants : `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Les exécutions d'agent se font en deux étapes :

1. Accusé de réception immédiat (`status:"accepted"`)
2. Réponse de complétion finale (`status:"ok"|"error"`), avec des événements `agent` diffusés entre les deux.

Voir la documentation complète du protocole : [Protocole Gateway](/gateway/protocol).

## Vérifications opérationnelles

### Vivacité

- Ouvrir un WS et envoyer `connect`.
- Attendre une réponse `hello-ok` avec un instantané.

### Disponibilité

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Récupération de lacunes

Les événements ne sont pas rejoués. En cas de lacunes de séquence, rafraîchir l'état (`health`, `system-presence`) avant de continuer.

## Signatures de pannes courantes

| Signature                                                      | Problème probable                        |
| -------------------------------------------------------------- | ---------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Liaison non-loopback sans token/mot de passe |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflit de port                          |
| `Gateway start blocked: set gateway.mode=local`                | Configuration en mode distant            |
| `unauthorized` lors de la connexion                            | Incohérence d'authentification entre client et gateway |

Pour des procédures de diagnostic complètes, utilisez [Dépannage Gateway](/gateway/troubleshooting).

## Garanties de sécurité

- Les clients du protocole Gateway échouent rapidement lorsque le Gateway est indisponible (pas de repli implicite sur un canal direct).
- Les trames initiales invalides ou non-connect sont rejetées et la connexion est fermée.
- L'arrêt progressif émet un événement `shutdown` avant la fermeture du socket.

---

Liens connexes :

- [Dépannage](/gateway/troubleshooting)
- [Processus en arrière-plan](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Santé](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentification](/gateway/authentication)
