---
summary: "Déplacer (migrer) une installation OpenClaw d'une machine à une autre"
read_when:
  - Vous déplacez OpenClaw vers un nouveau laptop/serveur
  - Vous souhaitez préserver les sessions, l'authentification et les connexions aux canaux (WhatsApp, etc.)
title: "Guide de migration"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/install/migrating.md
  workflow: manual
---

# Migrer OpenClaw vers une nouvelle machine

Ce guide migre un Gateway OpenClaw d'une machine à une autre **sans refaire l'intégration initiale**.

La migration est conceptuellement simple :

- Copiez le **répertoire d'état** (`$OPENCLAW_STATE_DIR`, par défaut : `~/.openclaw/`) : il contient la configuration, l'authentification, les sessions et l'état des canaux.
- Copiez votre **espace de travail** (`~/.openclaw/workspace/` par défaut) : il contient vos fichiers d'agent (mémoire, prompts, etc.).

Mais il y a des pièges courants concernant les **profils**, les **permissions** et les **copies partielles**.

## Avant de commencer (ce que vous migrez)

### 1) Identifiez votre répertoire d'état

La plupart des installations utilisent le chemin par défaut :

- **Répertoire d'état :** `~/.openclaw/`

Mais il peut être différent si vous utilisez :

- `--profile <name>` (devient souvent `~/.openclaw-<profile>/`)
- `OPENCLAW_STATE_DIR=/some/path`

Si vous n'êtes pas sûr, exécutez sur l'**ancienne** machine :

```bash
openclaw status
```

Cherchez les mentions de `OPENCLAW_STATE_DIR` / profil dans la sortie. Si vous exécutez plusieurs Gateways, répétez pour chaque profil.

### 2) Identifiez votre espace de travail

Chemins par défaut courants :

- `~/.openclaw/workspace/` (espace de travail recommandé)
- un dossier personnalisé que vous avez créé

Votre espace de travail est l'endroit où se trouvent les fichiers comme `MEMORY.md`, `USER.md` et `memory/*.md`.

### 3) Comprenez ce que vous allez préserver

Si vous copiez **les deux** (répertoire d'état et espace de travail), vous conservez :

- La configuration du Gateway (`openclaw.json`)
- Les profils d'authentification / clés API / jetons OAuth
- L'historique des sessions + état de l'agent
- L'état des canaux (par ex. connexion/session WhatsApp)
- Vos fichiers d'espace de travail (mémoire, notes Skills, etc.)

Si vous copiez **uniquement** l'espace de travail (par ex., via Git), vous ne préservez **pas** :

- les sessions
- les identifiants
- les connexions aux canaux

Ceux-ci se trouvent sous `$OPENCLAW_STATE_DIR`.

## Étapes de migration (recommandées)

### Étape 0 : Faites une sauvegarde (ancienne machine)

Sur l'**ancienne** machine, arrêtez d'abord le Gateway pour que les fichiers ne changent pas pendant la copie :

```bash
openclaw gateway stop
```

(Optionnel mais recommandé) archivez le répertoire d'état et l'espace de travail :

```bash
# Ajustez les chemins si vous utilisez un profil ou des emplacements personnalisés
cd ~
tar -czf openclaw-state.tgz .openclaw

tar -czf openclaw-workspace.tgz .openclaw/workspace
```

Si vous avez plusieurs profils/répertoires d'état (par ex. `~/.openclaw-main`, `~/.openclaw-work`), archivez chacun.

### Étape 1 : Installez OpenClaw sur la nouvelle machine

Sur la **nouvelle** machine, installez le CLI (et Node si nécessaire) :

- Voir : [Installer](/install)

À cette étape, c'est normal si l'intégration crée un nouveau `~/.openclaw/` : vous l'écraserez à l'étape suivante.

### Étape 2 : Copiez le répertoire d'état + espace de travail sur la nouvelle machine

Copiez **les deux** :

- `$OPENCLAW_STATE_DIR` (par défaut `~/.openclaw/`)
- votre espace de travail (par défaut `~/.openclaw/workspace/`)

Méthodes courantes :

- `scp` des archives puis extraction
- `rsync -a` via SSH
- disque externe

Après la copie, vérifiez :

- Les répertoires cachés ont été inclus (par ex. `.openclaw/`)
- La propriété des fichiers est correcte pour l'utilisateur exécutant le Gateway

### Étape 3 : Exécutez Doctor (migrations + réparation du service)

Sur la **nouvelle** machine :

```bash
openclaw doctor
```

Doctor est la commande « sûre et fiable ». Elle répare les services, applique les migrations de configuration et signale les incohérences.

Puis :

```bash
openclaw gateway restart
openclaw status
```

## Pièges courants (et comment les éviter)

### Piège : décalage profil / répertoire d'état

Si vous exécutiez l'ancien Gateway avec un profil (ou `OPENCLAW_STATE_DIR`), et que le nouveau Gateway en utilisé un différent, vous verrez des symptômes comme :

- les modifications de configuration ne prennent pas effet
- les canaux manquants / déconnectés
- l'historique des sessions vide

Solution : exécutez le Gateway/service avec le **même** profil/répertoire d'état que celui que vous avez migré, puis relancez :

```bash
openclaw doctor
```

### Piège : copier uniquement `openclaw.json`

`openclaw.json` ne suffit pas. De nombreux fournisseurs stockent leur état sous :

- `$OPENCLAW_STATE_DIR/credentials/`
- `$OPENCLAW_STATE_DIR/agents/<agentId>/...`

Migrez toujours le dossier `$OPENCLAW_STATE_DIR` en entier.

### Piège : permissions / propriété

Si vous avez copié en tant que root ou changé d'utilisateur, le Gateway peut échouer à lire les identifiants/sessions.

Solution : assurez-vous que le répertoire d'état + l'espace de travail appartiennent à l'utilisateur exécutant le Gateway.

### Piège : migration entre les modes distant/local

- Si votre interface (WebUI/TUI) pointe vers un Gateway **distant**, c'est l'hôte distant qui possède le stockage des sessions + l'espace de travail.
- Migrer votre laptop ne déplacera pas l'état du Gateway distant.

Si vous êtes en mode distant, migrez l'**hôte du Gateway**.

### Piège : secrets dans les sauvegardes

`$OPENCLAW_STATE_DIR` contient des secrets (clés API, jetons OAuth, identifiants WhatsApp). Traitez les sauvegardes comme des secrets de production :

- stockez-les chiffrés
- évitez de les partager sur des canaux non sécurisés
- renouvelez les clés si vous suspectez une exposition

## Liste de vérification

Sur la nouvelle machine, confirmez :

- `openclaw status` montre le Gateway en cours d'exécution
- Vos canaux sont toujours connectés (par ex. WhatsApp ne demande pas de réappairage)
- Le tableau de bord s'ouvre et affiche les sessions existantes
- Vos fichiers d'espace de travail (mémoire, configurations) sont présents

## Liens connexes

- [Doctor](/gateway/doctor)
- [Dépannage du Gateway](/gateway/troubleshooting)
- [Où OpenClaw stocke-t-il ses données ?](/help/faq#where-does-openclaw-store-its-data)
