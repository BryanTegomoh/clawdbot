---
summary: "Espace de travail de l'agent : emplacement, disposition et stratégie de sauvegarde"
read_when:
  - Vous devez expliquer l'espace de travail de l'agent ou sa disposition de fichiers
  - Vous souhaitez sauvegarder ou migrer un espace de travail d'agent
title: "Espace de travail de l'agent"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/agent-workspace.md
  workflow: manual
---

# Espace de travail de l'agent

L'espace de travail est le foyer de l'agent. C'est le seul répertoire de travail utilise pour les outils de fichiers et pour le contexte de l'espace de travail. Gardez-le privé et traitez-le comme de la mémoire.

Ceci est séparé de `~/.openclaw/`, qui stocke la configuration, les identifiants et les sessions.

**Important :** l'espace de travail est le **cwd par défaut**, pas un bac à sable strict. Les outils résolvent les chemins relatifs par rapport à l'espace de travail, mais les chemins absolus peuvent toujours atteindre d'autres emplacements sur l'hôte sauf si l'isolation en bac à sable est activée. Si vous avez besoin d'isolation, utilisez [`agents.defaults.sandbox`](/gateway/sandboxing) (et/ou la configuration sandbox par agent).
Lorsque l'isolation en bac à sable est activée et que `workspaceAccess` n'est pas `"rw"`, les outils opèrent dans un espace de travail sandbox sous `~/.openclaw/sandboxes`, pas votre espace de travail hôte.

## Emplacement par défaut

- Par défaut : `~/.openclaw/workspace`
- Si `OPENCLAW_PROFILE` est défini et différent de `"default"`, la valeur par défaut devient `~/.openclaw/workspace-<profile>`.
- Surcharge dans `~/.openclaw/openclaw.json` :

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

`openclaw onboard`, `openclaw configuré` ou `openclaw setup` créeront l'espace de travail et initialiseront les fichiers bootstrap s'ils sont manquants.

Si vous gérez déjà les fichiers de l'espace de travail vous-même, vous pouvez désactiver la création des fichiers bootstrap :

```json5
{ agent: { skipBootstrap: true } }
```

## Dossiers d'espace de travail supplémentaires

Les installations anciennes peuvent avoir créé `~/openclaw`. Conserver plusieurs répertoires d'espace de travail peut causer des problèmes d'authentification ou de dérive d'état, car un seul espace de travail est actif à la fois.

**Recommandation :** conservez un seul espace de travail actif. Si vous n'utilisez plus les dossiers supplémentaires, archivez-les ou déplacez-les à la corbeille (par exemple `trash ~/openclaw`). Si vous conservez intentionnellement plusieurs espaces de travail, assurez-vous que `agents.defaults.workspace` pointe vers celui qui est actif.

`openclaw doctor` avertit quand il détecte des répertoires d'espace de travail supplémentaires.

## Carte des fichiers de l'espace de travail (signification de chaque fichier)

Voici les fichiers standards qu'OpenClaw attend dans l'espace de travail :

- `AGENTS.md`
  - Instructions de fonctionnement pour l'agent et comment il doit utiliser la mémoire.
  - Chargé au début de chaque session.
  - Bon endroit pour les règles, priorités et détails de « comment se comporter ».

- `SOUL.md`
  - Personnalité, ton et limités.
  - Chargé à chaque session.

- `USER.md`
  - Qui est l'utilisateur et comment s'adresser à lui.
  - Chargé à chaque session.

- `IDENTITY.md`
  - Le nom, le style et l'emoji de l'agent.
  - Créé/mis à jour pendant le rituel bootstrap.

- `TOOLS.md`
  - Notes sur vos outils locaux et conventions.
  - Ne contrôle pas la disponibilité des outils ; c'est uniquement un guide.

- `HEARTBEAT.md`
  - Petite checklist optionnelle pour les exécutions heartbeat.
  - Gardez-la courte pour éviter la consommation de tokens.

- `BOOT.md`
  - Checklist de démarrage optionnelle exécutée au redémarrage du Gateway quand les hooks internes sont activés.
  - Gardez-la courte ; utilisez l'outil message pour les envois sortants.

- `BOOTSTRAP.md`
  - Rituel de premier lancement unique.
  - Créé uniquement pour un tout nouvel espace de travail.
  - Supprimez-le après la complétion du rituel.

- `memory/YYYY-MM-DD.md`
  - Journal de mémoire quotidien (un fichier par jour).
  - Recommandé de lire aujourd'hui + hier au début de la session.

- `MEMORY.md` (optionnel)
  - Mémoire à long terme curatée.
  - Chargez uniquement dans la session principale privée (pas dans les contextes partagés/de groupe).

Voir [Mémoire](/concepts/memory) pour le flux de travail et la purge automatique de mémoire.

- `skills/` (optionnel)
  - Skills spécifiques à l'espace de travail.
  - Remplacent les skills gérés/intégrés en cas de collision de noms.

- `canvas/` (optionnel)
  - Fichiers d'interface Canvas pour les affichages de nœuds (par exemple `canvas/index.html`).

Si un fichier bootstrap est manquant, OpenClaw injecte un marqueur « fichier manquant » dans la session et continue. Les fichiers bootstrap volumineux sont tronqués lors de l'injection ; ajustez les limités avec `agents.defaults.bootstrapMaxChars` (défaut : 20000) et `agents.defaults.bootstrapTotalMaxChars` (défaut : 150000).
`openclaw setup` peut recréer les valeurs par défaut manquantes sans écraser les fichiers existants.

## Ce qui n'est PAS dans l'espace de travail

Ces éléments se trouvent sous `~/.openclaw/` et ne doivent PAS être commitées dans le dépôt de l'espace de travail :

- `~/.openclaw/openclaw.json` (configuration)
- `~/.openclaw/credentials/` (tokens OAuth, clés API)
- `~/.openclaw/agents/<agentId>/sessions/` (transcriptions de session + métadonnées)
- `~/.openclaw/skills/` (skills gérés)

Si vous devez migrer des sessions ou la configuration, copiez-les séparément et gardez-les hors du contrôle de version.

## Sauvegarde Git (recommandée, privée)

Traitez l'espace de travail comme de la mémoire privée. Placez-le dans un dépôt git **privé** pour qu'il soit sauvegardé et récupérable.

Exécutez ces étapes sur la machine où le Gateway fonctionne (c'est là que l'espace de travail réside).

### 1) Initialiser le dépôt

Si git est installé, les tout nouveaux espaces de travail sont initialisés automatiquement. Si cet espace de travail n'est pas encore un dépôt, exécutez :

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Add agent workspace"
```

### 2) Ajouter un remote privé (options pour débutants)

Option A : Interface web GitHub

1. Créez un nouveau dépôt **privé** sur GitHub.
2. N'initialisez pas avec un README (évite les conflits de fusion).
3. Copiez l'URL remote HTTPS.
4. Ajoutez le remote et poussez :

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Option B : GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Option C : Interface web GitLab

1. Créez un nouveau dépôt **privé** sur GitLab.
2. N'initialisez pas avec un README (évite les conflits de fusion).
3. Copiez l'URL remote HTTPS.
4. Ajoutez le remote et poussez :

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

### 3) Mises à jour continues

```bash
git status
git add .
git commit -m "Update memory"
git push
```

## Ne commitez pas de secrets

Même dans un dépôt privé, évitez de stocker des secrets dans l'espace de travail :

- Clés API, tokens OAuth, mots de passe ou identifiants privés.
- Tout ce qui se trouve sous `~/.openclaw/`.
- Dumps bruts de conversations ou pièces jointes sensibles.

Si vous devez stocker des références sensibles, utilisez des espaces réservés et conservez le vrai secret ailleurs (gestionnaire de mots de passe, variables d'environnement ou `~/.openclaw/`).

Exemple de `.gitignore` de départ :

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

## Déplacer l'espace de travail vers une nouvelle machine

1. Clonez le dépôt vers le chemin souhaité (par défaut `~/.openclaw/workspace`).
2. Définissez `agents.defaults.workspace` à ce chemin dans `~/.openclaw/openclaw.json`.
3. Exécutez `openclaw setup --workspace <chemin>` pour initialiser les fichiers manquants.
4. Si vous avez besoin des sessions, copiez `~/.openclaw/agents/<agentId>/sessions/` depuis l'ancienne machine séparément.

## Notes avancées

- Le routage multi-agent peut utiliser différents espaces de travail par agent. Voir [Routage de canaux](/channels/channel-routing) pour la configuration du routage.
- Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent utiliser des espaces de travail sandbox par session sous `agents.defaults.sandbox.workspaceRoot`.
