---
summary: "Guide ClawHub : registre public de Skills + workflows CLI"
read_when:
  - Introduction de ClawHub aux nouveaux utilisateurs
  - Installation, recherche ou publication de Skills
  - Explication des options CLI ClawHub et du comportement de synchronisation
title: "ClawHub"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/clawhub.md
  workflow: manual
---

# ClawHub

ClawHub est le **registre public de Skills pour OpenClaw**. C'est un service gratuit : tous les Skills sont publics, ouverts et visibles par tous pour le partage et la réutilisation. Un Skill est simplement un dossier avec un fichier `SKILL.md` (plus des fichiers texte de support). Vous pouvez parcourir les Skills dans l'application web ou utiliser le CLI pour rechercher, installer, mettre à jour et publier des Skills.

Site : [clawhub.ai](https://clawhub.ai)

## Qu'est-ce que ClawHub

- Un registre public pour les Skills OpenClaw.
- Un magasin versionné de bundles de Skills et de métadonnées.
- Une surface de découverte pour la recherche, les tags et les signaux d'utilisation.

## Comment ça fonctionne

1. Un utilisateur publie un bundle de Skill (fichiers + métadonnées).
2. ClawHub stocke le bundle, analyse les métadonnées et attribue une version.
3. Le registre indexe le Skill pour la recherche et la découverte.
4. Les utilisateurs parcourent, téléchargent et installent les Skills dans OpenClaw.

## Ce que vous pouvez faire

- Publier de nouveaux Skills et de nouvelles versions de Skills existants.
- Découvrir des Skills par nom, tags ou recherche.
- Télécharger des bundles de Skills et inspecter leurs fichiers.
- Signaler des Skills abusifs ou dangereux.
- Si vous êtes modérateur, masquer, démasquer, supprimer ou bannir.

## Pour qui (accessible aux débutants)

Si vous souhaitez ajouter de nouvelles capacités à votre agent OpenClaw, ClawHub est le moyen le plus simple de trouver et installer des Skills. Vous n'avez pas besoin de savoir comment le backend fonctionne. Vous pouvez :

- Rechercher des Skills en langage naturel.
- Installer un Skill dans votre espace de travail.
- Mettre à jour les Skills plus tard avec une seule commande.
- Sauvegarder vos propres Skills en les publiant.

## Démarrage rapide (non technique)

1. Installez le CLI (voir la section suivante).
2. Recherchez quelque chose dont vous avez besoin :
   - `clawhub search "calendar"`
3. Installez un Skill :
   - `clawhub install <skill-slug>`
4. Démarrez une nouvelle session OpenClaw pour qu'elle prenne en compte le nouveau Skill.

## Installer le CLI

Choisissez l'une des options :

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

## Intégration avec OpenClaw

Par défaut, le CLI installe les Skills dans `./skills` sous votre répertoire de travail actuel. Si un espace de travail OpenClaw est configuré, `clawhub` se rabat sur cet espace de travail sauf si vous substituez `--workdir` (ou `CLAWHUB_WORKDIR`). OpenClaw charge les Skills de l'espace de travail depuis `<workspace>/skills` et les prendra en compte dans la **prochaine** session. Si vous utilisez déjà `~/.openclaw/skills` ou des Skills intégrés, les Skills de l'espace de travail ont la priorité.

Pour plus de détails sur le chargement, le partage et le contrôle d'accès des Skills, voir
[Skills](/tools/skills).

## Vue d'ensemble du système de Skills

Un Skill est un bundle versionné de fichiers qui apprend à OpenClaw comment effectuer une
tâche spécifique. Chaque publication crée une nouvelle version, et le registre conserve un
historique des versions pour que les utilisateurs puissent auditer les changements.

Un Skill typique inclut :

- Un fichier `SKILL.md` avec la description principale et l'utilisation.
- Des configurations, scripts ou fichiers de support optionnels utilisés par le Skill.
- Des métadonnées telles que les tags, le résumé et les exigences d'installation.

ClawHub utilisé les métadonnées pour alimenter la découverte et exposer en toute sécurité les capacités des Skills.
Le registre suit également les signaux d'utilisation (comme les étoiles et les téléchargements) pour améliorer
le classement et la visibilité.

## Fonctionnalités du service

- **Navigation publique** des Skills et de leur contenu `SKILL.md`.
- **Recherche** alimentée par des embeddings (recherche vectorielle), pas seulement des mots-clés.
- **Versionnage** avec semver, changelogs et tags (y compris `latest`).
- **Téléchargements** sous forme de zip par version.
- **Étoiles et commentaires** pour les retours de la communauté.
- **Modération** avec des hooks pour les approbations et les audits.
- **API compatible CLI** pour l'automatisation et le scripting.

## Sécurité et modération

ClawHub est ouvert par défaut. N'importe qui peut télécharger des Skills, mais un compte GitHub doit
avoir au moins une semaine pour publier. Cela aide à ralentir les abus sans bloquer
les contributeurs légitimes.

Signalement et modération :

- Tout utilisateur connecté peut signaler un Skill.
- Les raisons du signalement sont obligatoires et enregistrées.
- Chaque utilisateur peut avoir jusqu'à 20 signalements actifs à la fois.
- Les Skills avec plus de 3 signalements uniques sont auto-masqués par défaut.
- Les modérateurs peuvent voir les Skills masqués, les démasquer, les supprimer ou bannir des utilisateurs.
- L'abus de la fonctionnalité de signalement peut entraîner des bannissements de compte.

Intéressé pour devenir modérateur ? Demandez sur le Discord OpenClaw et contactez un
modérateur ou mainteneur.

## Commandes CLI et paramètres

Options globales (s'appliquent à toutes les commandes) :

- `--workdir <dir>` : Répertoire de travail (défaut : répertoire courant ; se rabat sur l'espace de travail OpenClaw).
- `--dir <dir>` : Répertoire des Skills, relatif au workdir (défaut : `skills`).
- `--site <url>` : URL de base du site (connexion navigateur).
- `--registry <url>` : URL de base de l'API du registre.
- `--no-input` : Désactiver les invites (non interactif).
- `-V, --cli-version` : Afficher la version du CLI.

Authentification :

- `clawhub login` (flux navigateur) ou `clawhub login --token <token>`
- `clawhub logout`
- `clawhub whoami`

Options :

- `--token <token>` : Coller un token API.
- `--label <label>` : Label stocké pour les tokens de connexion navigateur (défaut : `CLI token`).
- `--no-browser` : Ne pas ouvrir de navigateur (nécessite `--token`).

Recherche :

- `clawhub search "query"`
- `--limit <n>` : Résultats maximum.

Installation :

- `clawhub install <slug>`
- `--version <version>` : Installer une version spécifique.
- `--force` : Écraser si le dossier existe déjà.

Mise à jour :

- `clawhub update <slug>`
- `clawhub update --all`
- `--version <version>` : Mettre à jour vers une version spécifique (slug unique seulement).
- `--force` : Écraser quand les fichiers locaux ne correspondent à aucune version publiée.

Liste :

- `clawhub list` (lit `.clawhub/lock.json`)

Publication :

- `clawhub publish <path>`
- `--slug <slug>` : Slug du Skill.
- `--name <name>` : Nom d'affichage.
- `--version <version>` : Version semver.
- `--changelog <text>` : Texte du changelog (peut être vide).
- `--tags <tags>` : Tags séparés par des virgules (défaut : `latest`).

Suppression/restauration (propriétaire/admin uniquement) :

- `clawhub delete <slug> --yes`
- `clawhub undelete <slug> --yes`

Synchronisation (scanner les Skills locaux + publier les nouveaux/mis à jour) :

- `clawhub sync`
- `--root <dir...>` : Racines de scan supplémentaires.
- `--all` : Tout télécharger sans invites.
- `--dry-run` : Afficher ce qui serait téléchargé.
- `--bump <type>` : `patch|minor|major` pour les mises à jour (défaut : `patch`).
- `--changelog <text>` : Changelog pour les mises à jour non interactives.
- `--tags <tags>` : Tags séparés par des virgules (défaut : `latest`).
- `--concurrency <n>` : Vérifications du registre (défaut : 4).

## Workflows courants pour les agents

### Rechercher des Skills

```bash
clawhub search "postgres backups"
```

### Télécharger de nouveaux Skills

```bash
clawhub install my-skill-pack
```

### Mettre à jour les Skills installés

```bash
clawhub update --all
```

### Sauvegarder vos Skills (publier ou synchroniser)

Pour un seul dossier de Skill :

```bash
clawhub publish ./my-skill --slug my-skill --name "My Skill" --version 1.0.0 --tags latest
```

Pour scanner et sauvegarder plusieurs Skills à la fois :

```bash
clawhub sync --all
```

## Détails avancés (technique)

### Versionnage et tags

- Chaque publication crée une nouvelle version **semver** `SkillVersion`.
- Les tags (comme `latest`) pointent vers une version ; déplacer les tags vous permet de revenir en arrière.
- Les changelogs sont attachés par version et peuvent être vides lors de la synchronisation ou de la publication de mises à jour.

### Changements locaux vs versions du registre

Les mises à jour comparent le contenu local du Skill aux versions du registre en utilisant un hash de contenu. Si les fichiers locaux ne correspondent à aucune version publiée, le CLI demande avant d'écraser (ou nécessite `--force` en exécutions non interactives).

### Scan de synchronisation et racines de repli

`clawhub sync` scanne d'abord votre workdir actuel. Si aucun Skill n'est trouvé, il se rabat sur les emplacements hérités connus (par exemple `~/openclaw/skills` et `~/.openclaw/skills`). Ceci est conçu pour trouver les anciennes installations de Skills sans options supplémentaires.

### Stockage et fichier de verrouillage

- Les Skills installés sont enregistrés dans `.clawhub/lock.json` sous votre workdir.
- Les tokens d'authentification sont stockés dans le fichier de configuration du CLI ClawHub (substituable via `CLAWHUB_CONFIG_PATH`).

### Télémétrie (compteurs d'installation)

Quand vous exécutez `clawhub sync` en étant connecté, le CLI envoie un instantané minimal pour calculer les compteurs d'installation. Vous pouvez désactiver ceci entièrement :

```bash
export CLAWHUB_DISABLE_TELEMETRY=1
```

## Variables d'environnement

- `CLAWHUB_SITE` : Substituer l'URL du site.
- `CLAWHUB_REGISTRY` : Substituer l'URL de l'API du registre.
- `CLAWHUB_CONFIG_PATH` : Substituer l'emplacement de stockage du token/configuration du CLI.
- `CLAWHUB_WORKDIR` : Substituer le workdir par défaut.
- `CLAWHUB_DISABLE_TELEMETRY=1` : Désactiver la télémétrie lors du `sync`.
