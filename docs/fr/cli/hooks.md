---
summary: "Référence CLI pour `openclaw hooks` (hooks d'agent)"
read_when:
  - Vous souhaitez gérer les hooks d'agent
  - Vous souhaitez installer ou mettre à jour des hooks
title: "hooks"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/hooks.md
  workflow: manual
---

# `openclaw hooks`

Gérer les hooks d'agent (automatisations déclenchées par des événements pour les commandes comme `/new`, `/reset` et le démarrage du Gateway).

Liens :

- Hooks : [Hooks](/automation/hooks)
- Hooks de plugins : [Plugins](/tools/plugin#plugin-hooks)

## Lister tous les hooks

```bash
openclaw hooks list
```

Lister tous les hooks découverts depuis les répertoires de l'espace de travail, gérés et intégrés.

**Options :**

- `--eligible` : afficher uniquement les hooks éligibles (exigences satisfaites)
- `--json` : sortie en JSON
- `-v, --verbose` : afficher les informations détaillées incluant les exigences manquantes

**Exemple de sortie :**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📎 bootstrap-extra-files ✓ - Inject extra workspace bootstrap files during agent bootstrap
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
```

**Exemple (verbose) :**

```bash
openclaw hooks list --verbose
```

Affiche les exigences manquantes pour les hooks inéligibles.

**Exemple (JSON) :**

```bash
openclaw hooks list --json
```

Renvoie du JSON structure pour une utilisation programmatique.

## Obtenir les informations d'un hook

```bash
openclaw hooks info <name>
```

Afficher les informations détaillées d'un hook spécifique.

**Arguments :**

- `<name>` : nom du hook (par ex., `session-memory`)

**Options :**

- `--json` : sortie en JSON

**Exemple :**

```bash
openclaw hooks info session-memory
```

**Sortie :**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/automation/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## Vérifier l'eligibilite des hooks

```bash
openclaw hooks check
```

Afficher un résumé de l'état d'eligibilite des hooks (combien sont prêts vs non prêts).

**Options :**

- `--json` : sortie en JSON

**Exemple de sortie :**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Activer un hook

```bash
openclaw hooks enable <name>
```

Activer un hook spécifique en l'ajoutant à votre configuration (`~/.openclaw/config.json`).

**Note :** Les hooks gérés par des plugins affichent `plugin:<id>` dans `openclaw hooks list` et ne peuvent pas être activés/désactivés ici. Activez/désactivez le plugin à la place.

**Arguments :**

- `<name>` : nom du hook (par ex., `session-memory`)

**Exemple :**

```bash
openclaw hooks enable session-memory
```

**Sortie :**

```
✓ Enabled hook: 💾 session-memory
```

**Ce que cela fait :**

- Vérifié que le hook existe et est éligible
- Met à jour `hooks.internal.entries.<name>.enabled = true` dans votre configuration
- Sauvegarde la configuration sur le disque

**Après l'activation :**

- Redémarrez le Gateway pour que les hooks soient rechargés (redémarrage de l'application barre de menu sur macOS, ou redémarrage de votre processus Gateway en dev).

## Désactiver un hook

```bash
openclaw hooks disable <name>
```

Désactiver un hook spécifique en mettant à jour votre configuration.

**Arguments :**

- `<name>` : nom du hook (par ex., `command-logger`)

**Exemple :**

```bash
openclaw hooks disable command-logger
```

**Sortie :**

```
⏸ Disabled hook: 📝 command-logger
```

**Après la désactivation :**

- Redémarrez le Gateway pour que les hooks soient recharges

## Installer des hooks

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

Installer un pack de hooks depuis un dossier/archive local ou npm.

Les specs npm sont **registre uniquement** (nom de paquet + version/tag optionnel). Les specs Git/URL/fichier sont rejetées. Les installations de dependances s'exécutent avec `--ignore-scripts` pour la sécurité.

**Ce que cela fait :**

- Copie le pack de hooks dans `~/.openclaw/hooks/<id>`
- Activé les hooks installés dans `hooks.internal.entries.*`
- Enregistre l'installation sous `hooks.internal.installs`

**Options :**

- `-l, --link` : lier un répertoire local au lieu de copier (l'ajouté à `hooks.internal.load.extraDirs`)
- `--pin` : enregistrer les installations npm comme `name@version` exacte résolue dans `hooks.internal.installs`

**Archives supportées :** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Exemples :**

```bash
# Répertoire local
openclaw hooks install ./my-hook-pack

# Archive locale
openclaw hooks install ./my-hook-pack.zip

# Paquet NPM
openclaw hooks install @openclaw/my-hook-pack

# Lier un répertoire local sans copier
openclaw hooks install -l ./my-hook-pack
```

## Mettre à jour des hooks

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Mettre à jour les packs de hooks installés (installations npm uniquement).

**Options :**

- `--all` : mettre à jour tous les packs de hooks suivis
- `--dry-run` : montrer ce qui changerait sans écrire

Lorsqu'un hash d'intégrité stocké existe et que le hash de l'artefact récupéré change, OpenClaw affiche un avertissement et demande confirmation avant de continuer. Utilisez l'option globale `--yes` pour ignorer les invites en CI/exécutions non interactives.

## Hooks intégrés

### session-memory

Sauvegarde le contexte de session en mémoire lorsque vous exécutez `/new`.

**Activer :**

```bash
openclaw hooks enable session-memory
```

**Sortie :** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Voir :** [documentation session-memory](/automation/hooks#session-memory)

### bootstrap-extra-files

Injecte des fichiers bootstrap supplémentaires (par exemple `AGENTS.md` / `TOOLS.md` locaux au monorepo) pendant `agent:bootstrap`.

**Activer :**

```bash
openclaw hooks enable bootstrap-extra-files
```

**Voir :** [documentation bootstrap-extra-files](/automation/hooks#bootstrap-extra-files)

### command-logger

Enregistre tous les événements de commande dans un fichier d'audit centralise.

**Activer :**

```bash
openclaw hooks enable command-logger
```

**Sortie :** `~/.openclaw/logs/commands.log`

**Consulter les logs :**

```bash
# Commandes récentes
tail -n 20 ~/.openclaw/logs/commands.log

# Affichage formate
cat ~/.openclaw/logs/commands.log | jq .

# Filtrer par action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Voir :** [documentation command-logger](/automation/hooks#command-logger)

### boot-md

Exécute `BOOT.md` au démarrage du Gateway (après le démarrage des canaux).

**Événements** : `gateway:startup`

**Activer** :

```bash
openclaw hooks enable boot-md
```

**Voir :** [documentation boot-md](/automation/hooks#boot-md)
