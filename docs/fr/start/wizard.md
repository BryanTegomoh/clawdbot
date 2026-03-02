---
read_when:
  - Exécution ou configuration de l'assistant de configuration initiale
  - Installation sur une nouvelle machine
summary: "Assistant de configuration initiale CLI : configuration guidée du gateway, de l'espace de travail, des canaux et des skills"
title: "Assistant de configuration initiale (CLI)"
sidebarTitle: "Configuration initiale : CLI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/wizard.md
  workflow: manual
---

# Assistant de configuration initiale (CLI)

L'assistant de configuration initiale est la méthode **recommandée** pour installer OpenClaw sur macOS,
Linux ou Windows (via WSL2 ; fortement recommandé).
Il configure un Gateway local ou une connexion à un Gateway distant, ainsi que les canaux, les skills
et les paramètres par défaut de l'espace de travail en un seul flux guidé.

```bash
openclaw onboard
```

<Info>
Chat le plus rapide : ouvrez l'interface de contrôle (aucune configuration de canal nécessaire). Exécutez
`openclaw dashboard` et discutez dans le navigateur. Documentation : [Tableau de bord](/web/dashboard).
</Info>

Pour reconfigurer ultérieurement :

```bash
openclaw configure
openclaw agents add <nom>
```

<Note>
`--json` n'implique pas le mode non-interactif. Pour les scripts, utilisez `--non-interactive`.
</Note>

<Tip>
Recommandé : configurez une clé API Brave Search pour que l'agent puisse utiliser `web_search`
(`web_fetch` fonctionne sans clé). Chemin le plus simple : `openclaw configuré --section web`
qui enregistre `tools.web.search.apiKey`. Documentation : [Outils Web](/tools/web).
</Tip>

## Démarrage rapide vs Avancé

L'assistant commence par **Démarrage rapide** (paramètres par défaut) vs **Avancé** (contrôle total).

<Tabs>
  <Tab title="Démarrage rapide (par défaut)">
    - Gateway local (local loopback)
    - Espace de travail par défaut (ou espace de travail existant)
    - Port du Gateway **18789**
    - Authentification du Gateway par **jeton** (généré automatiquement, même en local loopback)
    - Isolation DM par défaut : la configuration initiale locale écrit `session.dmScope: "per-channel-peer"` lorsque non défini. Détails : [Référence de la configuration initiale CLI](/start/wizard-cli-reference#outputs-and-internals)
    - Exposition Tailscale **désactivée**
    - Les messages privés Telegram + WhatsApp utilisent par défaut la **liste autorisée** (vous serez invité à fournir votre numéro de téléphone)
  </Tab>
  <Tab title="Avancé (contrôle total)">
    - Expose chaque étape (mode, espace de travail, gateway, canaux, démon, skills).
  </Tab>
</Tabs>

## Ce que l'assistant configure

**Mode local (par défaut)** vous guide à travers ces étapes :

1. **Modèle/Authentification** — Clé API Anthropic (recommandée), OpenAI ou fournisseur personnalisé
   (compatible OpenAI, compatible Anthropic ou détection automatique inconnue). Choisissez un modèle par défaut.
   Pour les exécutions non interactives, `--secret-input-mode ref` stocke des références basées sur l'environnement dans les profils d'authentification au lieu de valeurs de clé API en clair.
   En mode `ref` non interactif, la variable d'environnement du fournisseur doit être définie ; passer des drapeaux de clé en ligne sans cette variable d'environnement échoue immédiatement.
   En mode interactif, le choix du mode référence de secret permet de pointer vers une variable d'environnement ou une référence de fournisseur configurée (`file` ou `exec`), avec une validation préliminaire rapide avant la sauvegarde.
2. **Espace de travail** — Emplacement des fichiers de l'agent (par défaut `~/.openclaw/workspace`). Initialisé les fichiers de démarrage.
3. **Gateway** — Port, adresse de liaison, mode d'authentification, exposition Tailscale.
4. **Canaux** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles ou iMessage.
5. **Démon** — Installe un LaunchAgent (macOS) ou une unité utilisateur systemd (Linux/WSL2).
6. **Vérification de santé** — Démarre le Gateway et vérifie qu'il fonctionne.
7. **Skills** — Installe les skills recommandés et les dépendances optionnelles.

<Note>
Relancer l'assistant **ne supprime rien** sauf si vous choisissez explicitement **Réinitialiser** (ou passez `--reset`).
CLI `--reset` par défaut réinitialise la configuration, les identifiants et les sessions ; utilisez `--reset-scope full` pour inclure l'espace de travail.
Si la configuration est invalide ou contient des clés obsolètes, l'assistant vous demande d'exécuter `openclaw doctor` d'abord.
</Note>

**Mode distant** configuré uniquement le client local pour se connecter à un Gateway ailleurs.
Il **n'installe ni ne modifie rien** sur l'hôte distant.

## Ajouter un autre agent

Utilisez `openclaw agents add <nom>` pour créer un agent séparé avec son propre espace de travail,
ses sessions et ses profils d'authentification. L'exécution sans `--workspace` lance l'assistant.

Ce qui est défini :

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

Notes :

- Les espaces de travail par défaut suivent `~/.openclaw/workspace-<agentId>`.
- Ajoutez des `bindings` pour router les messages entrants (l'assistant peut le faire).
- Options non-interactives : `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Référence complète

Pour des détails étape par étape, le scripting non-interactif, la configuration de Signal,
l'API RPC et la liste complète des champs de configuration écrits par l'assistant, consultez la
[Référence de l'assistant](/reference/wizard).

## Documentation associée

- Référence de commande CLI : [`openclaw onboard`](/cli/onboard)
- Vue d'ensemble de la configuration initiale : [Vue d'ensemble de la configuration initiale](/start/onboarding-overview)
- Configuration initiale de l'application macOS : [Configuration initiale](/start/onboarding)
- Rituel de premier démarrage de l'agent : [Initialisation de l'agent](/start/bootstrapping)
