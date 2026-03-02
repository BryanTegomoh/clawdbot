---
summary: "Outils de débogage : mode watch, flux bruts des modèles et traçage des fuites de raisonnement"
read_when:
  - Vous devez inspecter la sortie brute du modèle pour détecter des fuites de raisonnement
  - Vous voulez exécuter le Gateway en mode watch pendant vos itérations
  - Vous avez besoin d'un flux de travail de débogage reproductible
title: "Débogage"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: help/debugging.md
  workflow: manual
---

# Débogage

Cette page couvre les utilitaires de débogage pour la sortie en streaming, en particulier lorsqu'un fournisseur mélange du raisonnement dans le texte normal.

## Remplacements de débogage à l'exécution

Utilisez `/debug` dans le chat pour définir des remplacements de configuration **uniquement en mémoire** (mémoire, pas sur disque).
`/debug` est désactivé par défaut ; activez-le avec `commands.debug: true`.
C'est pratique lorsque vous devez basculer des paramètres obscurs sans modifier `openclaw.json`.

Exemples :

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` efface tous les remplacements et revient à la configuration sur disque.

## Mode watch du Gateway

Pour une itération rapide, exécutez le Gateway sous le surveillant de fichiers :

```bash
pnpm gateway:watch
```

Cela correspond à :

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

Ajoutez n'importe quel flag CLI du Gateway après `gateway:watch` et ils seront transmis à chaque redémarrage.

## Profil dev + Gateway dev (--dev)

Utilisez le profil dev pour isoler l'état et créer une configuration jetable et sûre pour le débogage. Il y a **deux** flags `--dev` :

- **`--dev` global (profil) :** isole l'état sous `~/.openclaw-dev` et met par défaut le port du Gateway à `19001` (les ports dérivés se décalent en conséquence).
- **`gateway --dev` : indique au Gateway de créer automatiquement une configuration par défaut + un espace de travail** lorsqu'ils sont manquants (et ignore BOOTSTRAP.md).

Flux recommandé (profil dev + bootstrap dev) :

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Si vous n'avez pas encore d'installation globale, exécutez le CLI via `pnpm openclaw ...`.

Ce que cela fait :

1. **Isolation du profil** (`--dev` global)
   - `OPENCLAW_PROFILE=dev`
   - `OPENCLAW_STATE_DIR=~/.openclaw-dev`
   - `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
   - `OPENCLAW_GATEWAY_PORT=19001` (le navigateur/canvas se décale en conséquence)

2. **Bootstrap dev** (`gateway --dev`)
   - Écrit une configuration minimale si manquante (`gateway.mode=local`, liaison loopback).
   - Définit `agent.workspace` sur l'espace de travail dev.
   - Définit `agent.skipBootstrap=true` (pas de BOOTSTRAP.md).
   - Initialisé les fichiers de l'espace de travail s'ils sont manquants :
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   - Identité par défaut : **C3-PO** (droïde de protocole).
   - Ignoré les fournisseurs de canaux en mode dev (`OPENCLAW_SKIP_CHANNELS=1`).

Flux de réinitialisation (redémarrage propre) :

```bash
pnpm gateway:dev:reset
```

Note : `--dev` est un flag de profil **global** et est absorbé par certains exécuteurs.
Si vous devez l'expliciter, utilisez la forme variable d'environnement :

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` supprimé la configuration, les identifiants, les sessions et l'espace de travail dev (en utilisant `trash`, pas `rm`), puis recrée la configuration dev par défaut.

Astuce : si un Gateway non-dev est déjà en cours d'exécution (launchd/systemd), arrêtez-le d'abord :

```bash
openclaw gateway stop
```

## Journalisation du flux brut (OpenClaw)

OpenClaw peut journaliser le **flux brut de l'assistant** avant tout filtrage/formatage.
C'est la meilleure façon de voir si le raisonnement arrive sous forme de deltas de texte brut (ou comme des blocs de réflexion séparés).

Activez-le via le CLI :

```bash
pnpm gateway:watch --raw-stream
```

Remplacement optionnel du chemin :

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Variables d'environnement équivalentes :

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Fichier par défaut :

`~/.openclaw/logs/raw-stream.jsonl`

## Journalisation des chunks bruts (pi-mono)

Pour capturer les **chunks bruts compatibles OpenAI** avant qu'ils soient analysés en blocs,
pi-mono expose un journaliseur séparé :

```bash
PI_RAW_STREAM=1
```

Chemin optionnel :

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Fichier par défaut :

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> Note : ceci est émis uniquement par les processus utilisant le fournisseur `openai-completions` de pi-mono.

## Notes de sécurité

- Les journaux de flux bruts peuvent inclure des prompts complets, des sorties d'outils et des données utilisateur.
- Gardez les journaux en local et supprimez-les après le débogage.
- Si vous partagez des journaux, nettoyez d'abord les secrets et les données personnelles.
