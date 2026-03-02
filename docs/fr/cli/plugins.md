---
summary: "Référence CLI pour `openclaw plugins` (lister, installer, désinstaller, activer/désactiver, diagnostiquer)"
read_when:
  - Vous souhaitez installer ou gérer des plugins Gateway en processus
  - Vous souhaitez déboguer les échecs de chargement de plugins
title: "plugins"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/plugins.md
  workflow: manual
---

# `openclaw plugins`

Gérer les plugins/extensions du Gateway (charges en processus).

Liens :

- Système de plugins : [Plugins](/tools/plugin)
- Manifeste et schema de plugin : [Plugin manifest](/plugins/manifest)
- Renforcement de la sécurité : [Security](/gateway/security)

## Commandes

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Les plugins intégrés sont livres avec OpenClaw mais demarrent désactivés. Utilisez `plugins enable` pour les activer.

Tous les plugins doivent fournir un fichier `openclaw.plugin.json` avec un JSON Schema en ligne (`configSchema`, même si vide). Les manifestes ou schemas manquants/invalides empechent le plugin de se charger et échouent à la validation de la configuration.

### Installer

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

Note de sécurité : traitez les installations de plugins comme l'exécution de code. Préférez les versions fixées.

Les specs npm sont **registre uniquement** (nom de paquet + version/tag optionnel). Les specs Git/URL/fichier sont rejetées. Les installations de dependances s'exécutent avec `--ignore-scripts` pour la sécurité.

Archives supportées : `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Utilisez `--link` pour éviter de copier un répertoire local (l'ajouté à `plugins.load.paths`) :

```bash
openclaw plugins install -l ./my-plugin
```

Utilisez `--pin` sur les installations npm pour sauvegarder la spec exacte résolue (`name@version`) dans `plugins.installs` tout en gardant le comportement par défaut non fixe.

### Désinstaller

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` supprimé les enregistrements de plugin de `plugins.entries`, `plugins.installs`, la liste d'autorisation de plugins et les entrées liées `plugins.load.paths` le cas échéant. Pour les plugins mémoire actifs, le slot mémoire est réinitialisé à `memory-core`.

Par défaut, la désinstallation supprime également le répertoire d'installation du plugin sous la racine des extensions du répertoire d'état actif (`$OPENCLAW_STATE_DIR/extensions/<id>`). Utilisez `--keep-files` pour conserver les fichiers sur le disque.

`--keep-config` est supporte comme alias obsolète de `--keep-files`.

### Mettre à jour

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Les mises à jour ne s'appliquent qu'aux plugins installés depuis npm (suivis dans `plugins.installs`).

Lorsqu'un hash d'intégrité stocké existe et que le hash de l'artefact récupéré change, OpenClaw affiche un avertissement et demande confirmation avant de continuer. Utilisez l'option globale `--yes` pour ignorer les invites en CI/exécutions non interactives.
