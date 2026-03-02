---
summary: "Skills : managed vs workspace, règles de gating et câblage config/env"
read_when:
  - Ajout ou modification de Skills
  - Modification des règles de gating ou de chargement des Skills
title: "Skills"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/skills.md
  workflow: manual
---

# Skills (OpenClaw)

OpenClaw utilise des dossiers de Skills compatibles **[AgentSkills](https://agentskills.io)** pour apprendre à l'agent comment utiliser les outils. Chaque Skill est un répertoire contenant un `SKILL.md` avec un frontmatter YAML et des instructions. OpenClaw charge les **Skills embarqués** plus des surcharges locales optionnelles, et les filtré au chargement en fonction de l'environnement, de la configuration et de la présence des binaires.

## Emplacements et préséance

Les Skills sont chargés depuis **trois** endroits :

1. **Skills embarqués** : livrés avec l'installation (paquet npm ou OpenClaw.app)
2. **Skills managed/locaux** : `~/.openclaw/skills`
3. **Skills de l'espace de travail** : `<workspace>/skills`

En cas de conflit de nom de Skill, la préséance est :

`<workspace>/skills` (la plus haute) → `~/.openclaw/skills` → Skills embarqués (la plus basse)

De plus, vous pouvez configurer des dossiers de Skills supplémentaires (préséance la plus basse) via `skills.load.extraDirs` dans `~/.openclaw/openclaw.json`.

## Skills par agent vs partagés

Dans les configurations **multi-agents**, chaque agent a son propre espace de travail. Cela signifie :

- Les **Skills par agent** vivent dans `<workspace>/skills` pour cet agent uniquement.
- Les **Skills partagés** vivent dans `~/.openclaw/skills` (managed/local) et sont visibles par **tous les agents** sur la même machine.
- Les **dossiers partagés** peuvent aussi être ajoutés via `skills.load.extraDirs` (préséance la plus basse) si vous voulez un pack de Skills commun utilisé par plusieurs agents.

Si le même nom de Skill existe à plusieurs endroits, la préséance habituelle s'applique : workspace l'emporte, puis managed/local, puis embarqué.

## Plugins + Skills

Les plugins peuvent embarquer leurs propres Skills en listant des répertoires `skills` dans `openclaw.plugin.json` (chemins relatifs à la racine du plugin). Les Skills de plugins se chargent quand le plugin est activé et participent aux règles normales de préséance des Skills. Vous pouvez les conditionner via `metadata.openclaw.requires.config` sur l'entrée de configuration du plugin. Voir [Plugins](/tools/plugin) pour la découverte/configuration et [Outils](/tools) pour la surface d'outils que ces Skills enseignent.

## ClawHub (installation + synchronisation)

ClawHub est le registre public de Skills pour OpenClaw. Parcourez sur [https://clawhub.com](https://clawhub.com). Utilisez-le pour découvrir, installer, mettre à jour et sauvegarder des Skills. Guide complet : [ClawHub](/tools/clawhub).

Flux courants :

- Installer un Skill dans votre espace de travail :
  - `clawhub install <skill-slug>`
- Mettre à jour tous les Skills installés :
  - `clawhub update --all`
- Synchroniser (scanner + publier les mises à jour) :
  - `clawhub sync --all`

Par défaut, `clawhub` installe dans `./skills` sous votre répertoire de travail actuel (ou se rabat sur l'espace de travail OpenClaw configure). OpenClaw le prend en compte comme `<workspace>/skills` à la prochaine session.

## Notes de sécurité

- Traitez les Skills tiers comme du **code non approuvé**. Lisez-les avant de les activer.
- Préférez les exécutions sandboxées pour les entrées non approuvées et les outils risqués. Voir [Sandboxing](/gateway/sandboxing).
- `skills.entries.*.env` et `skills.entries.*.apiKey` injectent des secrets dans le processus **hôte** pour ce tour d'agent (pas le bac à sable). Gardez les secrets hors des prompts et des logs.
- Pour un modèle de menaces plus large et des checklists, voir [Sécurité](/gateway/security).

## Format (compatible AgentSkills + Pi)

`SKILL.md` doit inclure au minimum :

```markdown
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

Notes :

- Nous suivons la spécification AgentSkills pour la disposition/l'intention.
- Le parseur utilisé par l'agent embarqué supporte uniquement les clés de frontmatter **sur une seule ligne**.
- `metadata` doit être un **objet JSON sur une seule ligne**.
- Utilisez `{baseDir}` dans les instructions pour référencer le chemin du dossier du Skill.
- Clés de frontmatter optionnelles :
  - `homepage` : URL affichée comme « Website » dans l'interface macOS des Skills (aussi supporté via `metadata.openclaw.homepage`).
  - `user-invocable` : `true|false` (défaut : `true`). Quand `true`, le Skill est exposé comme commande slash utilisateur.
  - `disable-model-invocation` : `true|false` (défaut : `false`). Quand `true`, le Skill est exclu du prompt du modèle (toujours disponible via invocation utilisateur).
  - `command-dispatch` : `tool` (optionnel). Quand défini à `tool`, la commande slash contourne le modèle et dispatche directement vers un outil.
  - `command-tool` : nom de l'outil à invoquer quand `command-dispatch: tool` est défini.
  - `command-arg-mode` : `raw` (défaut). Pour le dispatch d'outil, transmet la chaîne d'arguments brute à l'outil (pas d'analyse du noyau).

    L'outil est invoqué avec les paramètres :
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`.

## Gating (filtrés au chargement)

OpenClaw **filtré les Skills au chargement** en utilisant `metadata` (JSON sur une seule ligne) :

```markdown
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

Champs sous `metadata.openclaw` :

- `always: true` : toujours inclure le Skill (ignorer les autres portes).
- `emoji` : emoji optionnel utilisé par l'interface macOS des Skills.
- `homepage` : URL optionnelle affichée comme « Website » dans l'interface macOS des Skills.
- `os` : liste optionnelle de plateformes (`darwin`, `linux`, `win32`). Si défini, le Skill n'est éligible que sur ces OS.
- `requires.bins` : liste ; chacun doit exister dans `PATH`.
- `requires.anyBins` : liste ; au moins un doit exister dans `PATH`.
- `requires.env` : liste ; la variable d'environnement doit exister **ou** être fournie dans la configuration.
- `requires.config` : liste de chemins `openclaw.json` qui doivent être truthy.
- `primaryEnv` : nom de variable d'environnement associé à `skills.entries.<name>.apiKey`.
- `install` : tableau optionnel de spécifications d'installeur utilisé par l'interface macOS des Skills (brew/node/go/uv/download).

Note sur le sandboxing :

- `requires.bins` est vérifié sur l'**hôte** au chargement du Skill.
- Si un agent est sandboxé, le binaire doit aussi exister **à l'intérieur du conteneur**. Installez-le via `agents.defaults.sandbox.docker.setupCommand` (ou une image personnalisée). `setupCommand` s'exécute une fois après la création du conteneur. L'installation de paquets nécessite aussi l'accès réseau sortant, un système de fichiers racine en écriture, et un utilisateur root dans le bac à sable. Exemple : le Skill `summarize` (`skills/summarize/SKILL.md`) a besoin du CLI `summarize` dans le conteneur du bac à sable pour s'y exécuter.

Exemple d'installeur :

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

Notes :

- Si plusieurs installeurs sont listés, le gateway choisit une **seule** option préférée (brew quand disponible, sinon node).
- Si tous les installeurs sont `download`, OpenClaw liste chaque entrée pour que vous puissiez voir les artefacts disponibles.
- Les spécifications d'installeur peuvent inclure `os: ["darwin"|"linux"|"win32"]` pour filtrer les options par plateforme.
- Les installations node respectent `skills.install.nodeManager` dans `openclaw.json` (défaut : npm ; options : npm/pnpm/yarn/bun). Ceci n'affecte que les **installations de Skills** ; le runtime du Gateway devrait rester Node (Bun n'est pas recommandé pour WhatsApp/Telegram).
- Installations Go : si `go` est absent et que `brew` est disponible, le gateway installe Go via Homebrew d'abord et définit `GOBIN` sur le `bin` de Homebrew quand possible.
- Installations download : `url` (obligatoire), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (défaut : auto quand archive détectée), `stripComponents`, `targetDir` (défaut : `~/.openclaw/tools/<skillKey>`).

Si aucun `metadata.openclaw` n'est présent, le Skill est toujours éligible (sauf s'il est désactivé dans la configuration ou bloqué par `skills.allowBundled` pour les Skills embarqués).

## Substitutions de configuration (`~/.openclaw/openclaw.json`)

Les Skills embarqués/managed peuvent être basculés et alimentés avec des valeurs d'environnement :

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // ou chaîne en clair
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

Note : si le nom du Skill contient des tirets, mettez la clé entre guillemets (JSON5 autorisé les clés entre guillemets).

Les clés de configuration correspondent au **nom du Skill** par défaut. Si un Skill définit `metadata.openclaw.skillKey`, utilisez cette clé sous `skills.entries`.

Règles :

- `enabled: false` désactive le Skill même s'il est embarqué/installé.
- `env` : injecté **uniquement si** la variable n'est pas déjà définie dans le processus.
- `apiKey` : commodité pour les Skills qui déclarent `metadata.openclaw.primaryEnv`.
  Supporte une chaîne en clair ou un objet SecretRef (`{ source, provider, id }`).
- `config` : sac optionnel pour les champs personnalisés par Skill ; les clés personnalisées doivent y vivre.
- `allowBundled` : liste d'autorisation optionnelle pour les Skills **embarqués** uniquement. Si défini, seuls les Skills embarqués dans la liste sont éligibles (les Skills managed/workspace ne sont pas affectés).

## Injection d'environnement (par exécution d'agent)

Quand une exécution d'agent démarre, OpenClaw :

1. Lit les métadonnées des Skills.
2. Applique tout `skills.entries.<key>.env` ou `skills.entries.<key>.apiKey` à `process.env`.
3. Construit le prompt système avec les Skills **éligibles**.
4. Restaure l'environnement original après la fin de l'exécution.

Ceci est **limité à l'exécution de l'agent**, pas un environnement shell global.

## Instantané de session (performance)

OpenClaw prend un instantané des Skills éligibles **quand une session démarre** et réutilise cette liste pour les tours suivants dans la même session. Les changements de Skills ou de configuration prennent effet à la prochaine nouvelle session.

Les Skills peuvent aussi se rafraîchir en cours de session quand le watcher de Skills est activé ou quand un nouveau node distant éligible apparaît (voir ci-dessous). Considérez ceci comme un **rechargement à chaud** : la liste rafraîchie est prise en compte au prochain tour d'agent.

## Nodes macOS distants (gateway Linux)

Si le Gateway fonctionne sous Linux mais qu'un **node macOS** est connecté **avec `system.run` autorisé** (la sécurité des approbations exec n'est pas définie à `deny`), OpenClaw peut traiter les Skills réservés à macOS comme éligibles quand les binaires requis sont présents sur ce node. L'agent devrait exécuter ces Skills via l'outil `nodes` (typiquement `nodes.run`).

Ceci repose sur le node rapportant son support de commandes et sur un sondage de binaire via `system.run`. Si le node macOS se déconnecte plus tard, les Skills restent visibles ; les invocations peuvent échouer jusqu'à la reconnexion du node.

## Watcher de Skills (rafraîchissement automatique)

Par défaut, OpenClaw surveille les dossiers de Skills et met à jour l'instantané des Skills quand les fichiers `SKILL.md` changent. Configurez ceci sous `skills.load` :

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## Impact sur les tokens (liste des Skills)

Quand des Skills sont éligibles, OpenClaw injecte une liste XML compacte des Skills disponibles dans le prompt système (via `formatSkillsForPrompt` dans `pi-coding-agent`). Le coût est déterministe :

- **Surcharge de base (uniquement quand >= 1 Skill) :** 195 caractères.
- **Par Skill :** 97 caractères + la longueur des valeurs `<name>`, `<description>`, et `<location>` échappées en XML.

Formule (caractères) :

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

Notes :

- L'échappement XML étend `& < > " '` en entités (`&amp;`, `&lt;`, etc.), augmentant la longueur.
- Le nombre de tokens varie selon le tokenizer du modèle. Une estimation approximative de style OpenAI est ~4 caractères/token, donc **97 caractères ≈ 24 tokens** par Skill plus les longueurs réelles de vos champs.

## Cycle de vie des Skills managed

OpenClaw livre un ensemble de base de Skills comme **Skills embarqués** dans le cadre de l'installation (paquet npm ou OpenClaw.app). `~/.openclaw/skills` existe pour les surcharges locales (par exemple, épingler/patcher un Skill sans modifier la copie embarquée). Les Skills de l'espace de travail sont la propriété de l'utilisateur et substituent les deux en cas de conflits de noms.

## Référence de configuration

Voir [Configuration des Skills](/tools/skills-config) pour le schéma complet de configuration.

## Vous cherchez plus de Skills ?

Parcourez [https://clawhub.com](https://clawhub.com).

---
