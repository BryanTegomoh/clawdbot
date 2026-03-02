---
title: "Mémoire"
summary: "Comment fonctionne la mémoire OpenClaw (fichiers de l'espace de travail + purge automatique de mémoire)"
read_when:
  - Vous voulez la disposition des fichiers mémoire et le workflow
  - Vous voulez ajuster la purge automatique de mémoire avant compaction
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/memory.md
  workflow: manual
---

# Mémoire

La mémoire OpenClaw est du **Markdown brut dans l'espace de travail de l'agent**. Les fichiers sont la source de vérité ; le modèle ne « se souvient » que de ce qui est écrit sur le disque.

Les outils de recherche mémoire sont fournis par le plugin de mémoire actif (par défaut : `memory-core`). Désactivez les plugins de mémoire avec `plugins.slots.memory = "none"`.

## Fichiers mémoire (Markdown)

La disposition par défaut de l'espace de travail utilise deux couches de mémoire :

- `memory/YYYY-MM-DD.md`
  - Journal quotidien (ajout uniquement).
  - Lire aujourd'hui + hier au début de la session.
- `MEMORY.md` (optionnel)
  - Mémoire à long terme curatée.
  - **Charger uniquement dans la session principale privée** (jamais dans les contextes de groupe).

Ces fichiers résident sous l'espace de travail (`agents.defaults.workspace`, par défaut `~/.openclaw/workspace`). Voir [Espace de travail de l'agent](/concepts/agent-workspace) pour la disposition complète.

## Outils mémoire

OpenClaw expose deux outils côté agent pour ces fichiers Markdown :

- `memory_search` : rappel sémantique sur les extraits indexés.
- `memory_get` : lecture ciblée d'un fichier/plage de lignes Markdown spécifique.

`memory_get` **dégrade gracieusement quand un fichier n'existe pas** (par exemple, le journal quotidien d'aujourd'hui avant la première écriture). Le gestionnaire intégré et le backend QMD retournent `{ text: "", path }` au lieu de lancer `ENOENT`, pour que les agents puissent gérer « rien n'a été enregistré encore » et continuer leur workflow sans encapsuler l'appel d'outil dans une logique try/catch.

## Quand écrire en mémoire

- Les décisions, préférences et faits durables vont dans `MEMORY.md`.
- Les notes du jour et le contexte courant vont dans `memory/YYYY-MM-DD.md`.
- Si quelqu'un dit « souviens-toi de ça », écrivez-le (ne le gardez pas en RAM).
- Ce domaine est encore en évolution. Il est utile de rappeler au modèle de stocker des souvenirs ; il saura quoi faire.
- Si vous voulez que quelque chose persiste, **demandez au bot de l'écrire** en mémoire.

## Purge automatique de mémoire (ping pré-compaction)

Quand une session est **proche de l'auto-compaction**, OpenClaw déclenche un **tour agentique silencieux** qui rappelle au modèle d'écrire la mémoire durable **avant** que le contexte ne soit compacté. Les prompts par défaut disent explicitement que le modèle _peut répondre_, mais généralement `NO_REPLY` est la réponse correcte pour que l'utilisateur ne voie jamais ce tour.

Ceci est contrôlé par `agents.defaults.compaction.memoryFlush` :

```json5
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Détails :

- **Seuil souple** : la purge se déclenche quand l'estimation de tokens de session dépasse `contextWindow - reserveTokensFloor - softThresholdTokens`.
- **Silencieuse** par défaut : les prompts incluent `NO_REPLY` pour que rien ne soit livré.
- **Deux prompts** : un prompt utilisateur plus un ajout de prompt système pour le rappel.
- **Une purge par cycle de compaction** (suivi dans `sessions.json`).
- **L'espace de travail doit être inscriptible** : si la session fonctionne en bac à sable avec `workspaceAccess: "ro"` ou `"none"`, la purge est ignorée.

Pour le cycle de vie complet de la compaction, voir [Gestion des sessions + compaction](/reference/session-management-compaction).

## Recherche mémoire vectorielle

OpenClaw peut construire un petit index vectoriel sur `MEMORY.md` et `memory/*.md` afin que les requêtes sémantiques puissent trouver des notes liées même quand la formulation diffère.

Valeurs par défaut :

- Activée par défaut.
- Surveille les fichiers mémoire pour les changements (avec anti-rebond).
- Configurer la recherche mémoire sous `agents.defaults.memorySearch` (pas au niveau supérieur `memorySearch`).
- Utilisé les embeddings distants par défaut. Si `memorySearch.provider` n'est pas défini, OpenClaw sélectionne automatiquement :
  1. `local` si un `memorySearch.local.modelPath` est configuré et que le fichier existe.
  2. `openai` si une clé OpenAI peut être résolue.
  3. `gemini` si une clé Gemini peut être résolue.
  4. `voyage` si une clé Voyage peut être résolue.
  5. `mistral` si une clé Mistral peut être résolue.
  6. Sinon la recherche mémoire reste désactivée jusqu'à configuration.
- Le mode local utilisé node-llama-cpp et peut nécessiter `pnpm approve-builds`.
- Utilisé sqlite-vec (quand disponible) pour accélérer la recherche vectorielle dans SQLite.

Les embeddings distants **nécessitent** une clé API pour le fournisseur d'embeddings. OpenClaw résout les clés depuis les profils d'authentification, `models.providers.*.apiKey` ou les variables d'environnement. Codex OAuth ne couvre que chat/completions et ne satisfait **pas** les embeddings pour la recherche mémoire. Pour Gemini, utilisez `GEMINI_API_KEY` ou `models.providers.google.apiKey`. Pour Voyage, utilisez `VOYAGE_API_KEY` ou `models.providers.voyage.apiKey`. Pour Mistral, utilisez `MISTRAL_API_KEY` ou `models.providers.mistral.apiKey`.
Quand vous utilisez un point d'accès personnalisé compatible OpenAI, définissez `memorySearch.remote.apiKey` (et optionnellement `memorySearch.remote.headers`).

### Backend QMD (expérimental)

Définissez `memory.backend = "qmd"` pour remplacer l'indexeur SQLite intégré par [QMD](https://github.com/tobi/qmd) : un sidecar de recherche local-first qui combine BM25 + vecteurs + reclassement. Le Markdown reste la source de vérité ; OpenClaw appelle QMD via shell pour la récupération. Points clés :

**Prérequis**

- Désactivé par défaut. Activer par configuration (`memory.backend = "qmd"`).
- Installer la CLI QMD séparément (`bun install -g https://github.com/tobi/qmd` ou télécharger une release) et s'assurer que le binaire `qmd` est dans le `PATH` du Gateway.
- QMD nécessite un build SQLite qui autorise les extensions (`brew install sqlite` sur macOS).
- QMD fonctionne entièrement localement via Bun + `node-llama-cpp` et télécharge automatiquement les modèles GGUF depuis HuggingFace à la première utilisation (pas de démon Ollama séparé requis).
- Le Gateway exécute QMD dans un home XDG autonome sous `~/.openclaw/agents/<agentId>/qmd/` en définissant `XDG_CONFIG_HOME` et `XDG_CACHE_HOME`.
- Support OS : macOS et Linux fonctionnent directement une fois Bun + SQLite installés. Windows est mieux supporté via WSL2.

**Comment le sidecar fonctionne**

- Le Gateway écrit un home QMD autonome sous `~/.openclaw/agents/<agentId>/qmd/` (config + cache + DB SQLite).
- Les collections sont créées via `qmd collection add` depuis `memory.qmd.paths` (plus les fichiers mémoire par défaut de l'espace de travail), puis `qmd update` + `qmd embed` s'exécutent au démarrage et à un intervalle configurable (`memory.qmd.update.interval`, défaut 5 min).
- Le Gateway initialisé maintenant le gestionnaire QMD au démarrage, donc les minuteurs de mise à jour périodique sont armés avant même le premier appel `memory_search`.
- Le rafraîchissement au démarrage s'exécute en arrière-plan par défaut pour ne pas bloquer le démarrage du chat ; définir `memory.qmd.update.waitForBootSync = true` pour garder le comportement bloquant précédent.
- Les recherches s'exécutent via `memory.qmd.searchMode` (défaut `qmd search --json` ; supporte aussi `vsearch` et `query`). Si le mode sélectionné rejette des options sur votre build QMD, OpenClaw réessaie avec `qmd query`. Si QMD échoue ou que le binaire est manquant, OpenClaw replie automatiquement vers le gestionnaire SQLite intégré pour que les outils mémoire continuent de fonctionner.
- OpenClaw n'expose pas aujourd'hui le réglage de taille de lot d'embedding QMD ; le comportement de lot est contrôlé par QMD lui-même.
- **La première recherche peut être lente** : QMD peut télécharger des modèles GGUF locaux (reclasseur/expansion de requête) lors de la première exécution de `qmd query`.
  - OpenClaw définit `XDG_CONFIG_HOME`/`XDG_CACHE_HOME` automatiquement quand il exécute QMD.
  - Si vous voulez pré-télécharger les modèles manuellement (et préchauffer le même index qu'OpenClaw utilise), exécutez une requête ponctuelle avec les répertoires XDG de l'agent.

    L'état QMD d'OpenClaw réside sous votre **répertoire d'état** (par défaut `~/.openclaw`).
    Vous pouvez pointer `qmd` vers exactement le même index en exportant les mêmes variables XDG qu'OpenClaw utilise :

    ```bash
    # Choisir le même répertoire d'état qu'OpenClaw utilise
    STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"

    export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
    export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"

    # (Optionnel) forcer un rafraîchissement d'index + embeddings
    qmd update
    qmd embed

    # Préchauffer / déclencher les téléchargements de modèles de première utilisation
    qmd query "test" -c memory-root --json >/dev/null 2>&1
    ```

**Surface de configuration (`memory.qmd.*`)**

- `command` (défaut `qmd`) : surcharger le chemin de l'exécutable.
- `searchMode` (défaut `search`) : choisir quelle commande QMD alimente `memory_search` (`search`, `vsearch`, `query`).
- `includeDefaultMemory` (défaut `true`) : indexer automatiquement `MEMORY.md` + `memory/**/*.md`.
- `paths[]` : ajouter des répertoires/fichiers supplémentaires (`path`, optionnel `pattern`, optionnel `name` stable).
- `sessions` : activer l'indexation JSONL des sessions (`enabled`, `retentionDays`, `exportDir`).
- `update` : contrôle la cadence de rafraîchissement et l'exécution de maintenance : (`interval`, `debounceMs`, `onBoot`, `waitForBootSync`, `embedInterval`, `commandTimeoutMs`, `updateTimeoutMs`, `embedTimeoutMs`).
- `limits` : limiter la charge utile de rappel (`maxResults`, `maxSnippetChars`, `maxInjectedChars`, `timeoutMs`).
- `scope` : même schéma que [`session.sendPolicy`](/gateway/configuration#session). Le défaut est DM uniquement (`deny` tout, `allow` conversations directes) ; relâcher pour afficher les résultats QMD dans les groupes/canaux.
  - `match.keyPrefix` correspond à la clé de session **normalisée** (minuscule, avec tout préfixe `agent:<id>:` retiré). Exemple : `discord:channel:`.
  - `match.rawKeyPrefix` correspond à la clé de session **brute** (minuscule), incluant `agent:<id>:`. Exemple : `agent:main:discord:`.
  - Hérité : `match.keyPrefix: "agent:..."` est encore traité comme préfixe de clé brute, mais préférez `rawKeyPrefix` pour la clarté.
- Quand `scope` refuse une recherche, OpenClaw journalise un avertissement avec le `channel`/`chatType` dérivé pour faciliter le débogage des résultats vides.
- Les extraits provenant de l'extérieur de l'espace de travail apparaissent comme `qmd/<collection>/<relative-path>` dans les résultats `memory_search` ; `memory_get` comprend ce préfixe et lit depuis la racine de collection QMD configurée.
- Quand `memory.qmd.sessions.enabled = true`, OpenClaw exporte les transcriptions de session nettoyées (tours Utilisateur/Assistant) dans une collection QMD dédiée sous `~/.openclaw/agents/<id>/qmd/sessions/`, pour que `memory_search` puisse rappeler les conversations récentes sans toucher l'index SQLite intégré.
- Les extraits `memory_search` incluent maintenant un pied de page `Source: <path#line>` quand `memory.citations` est `auto`/`on` ; définir `memory.citations = "off"` pour garder les métadonnées de chemin internes (l'agent reçoit toujours le chemin pour `memory_get`, mais le texte de l'extrait omet le pied de page et le prompt système avertit l'agent de ne pas le citer).

**Exemple**

```json5
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // Préfixe de clé de session normalisé (retire `agent:<id>:`).
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // Préfixe de clé de session brut (inclut `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**Citations et repli**

- `memory.citations` s'applique quel que soit le backend (`auto`/`on`/`off`).
- Quand `qmd` fonctionne, nous marquons `status().backend = "qmd"` pour que les diagnostics montrent quel moteur a servi les résultats. Si le sous-processus QMD se termine ou que la sortie JSON ne peut pas être analysée, le gestionnaire de recherche journalise un avertissement et retourne le fournisseur intégré (embeddings Markdown existants) jusqu'à ce que QMD récupère.

### Chemins mémoire supplémentaires

Si vous voulez indexer des fichiers Markdown en dehors de la disposition par défaut de l'espace de travail, ajoutez des chemins explicites :

```json5
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Notes :

- Les chemins peuvent être absolus ou relatifs à l'espace de travail.
- Les répertoires sont scannés récursivement pour les fichiers `.md`.
- Seuls les fichiers Markdown sont indexés.
- Les liens symboliques sont ignorés (fichiers ou répertoires).

### Embeddings Gemini (natifs)

Définir le fournisseur sur `gemini` pour utiliser l'API d'embeddings Gemini directement :

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Notes :

- `remote.baseUrl` est optionnel (par défaut l'URL de base de l'API Gemini).
- `remote.headers` permet d'ajouter des en-têtes supplémentaires si nécessaire.
- Modèle par défaut : `gemini-embedding-001`.

Si vous voulez utiliser un **point d'accès personnalisé compatible OpenAI** (OpenRouter, vLLM ou un proxy), vous pouvez utiliser la configuration `remote` avec le fournisseur OpenAI :

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Si vous ne voulez pas définir de clé API, utilisez `memorySearch.provider = "local"` ou définissez `memorySearch.fallback = "none"`.

Replis :

- `memorySearch.fallback` peut être `openai`, `gemini`, `voyage`, `mistral`, `local` ou `none`.
- Le fournisseur de repli est utilisé uniquement quand le fournisseur d'embeddings principal échoue.

Indexation par lots (OpenAI + Gemini + Voyage) :

- Désactivée par défaut. Définir `agents.defaults.memorySearch.remote.batch.enabled = true` pour activer pour l'indexation de grands corpus (OpenAI, Gemini et Voyage).
- Le comportement par défaut attend la complétion du lot ; ajuster `remote.batch.wait`, `remote.batch.pollIntervalMs` et `remote.batch.timeoutMinutes` si nécessaire.
- Définir `remote.batch.concurrency` pour contrôler le nombre de jobs de lot soumis en parallèle (défaut : 2).
- Le mode lot s'applique quand `memorySearch.provider = "openai"` ou `"gemini"` et utilise la clé API correspondante.
- Les jobs de lot Gemini utilisent le point d'accès de lot d'embeddings asynchrone et nécessitent la disponibilité de l'API Batch Gemini.

Pourquoi le lot OpenAI est rapide + économique :

- Pour les grands remplissages, OpenAI est typiquement l'option la plus rapide que nous supportons car nous pouvons soumettre de nombreuses requêtes d'embedding dans un seul job de lot et laisser OpenAI les traiter de manière asynchrone.
- OpenAI offre des tarifs réduits pour les charges de travail de l'API Batch, donc les grandes indexations sont généralement moins chères que l'envoi des mêmes requêtes de manière synchrone.
- Voir la documentation et les tarifs de l'API Batch OpenAI pour les détails :
  - [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
  - [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

Exemple de configuration :

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Outils :

- `memory_search` : retourne des extraits avec fichier + plages de lignes.
- `memory_get` : lire le contenu d'un fichier mémoire par chemin.

Mode local :

- Définir `agents.defaults.memorySearch.provider = "local"`.
- Fournir `agents.defaults.memorySearch.local.modelPath` (GGUF ou URI `hf:`).
- Optionnel : définir `agents.defaults.memorySearch.fallback = "none"` pour éviter le repli distant.

### Comment les outils mémoire fonctionnent

- `memory_search` recherche sémantiquement des morceaux Markdown (~400 tokens cible, 80 tokens de chevauchement) depuis `MEMORY.md` + `memory/**/*.md`. Il retourne le texte de l'extrait (plafonné ~700 caractères), le chemin du fichier, la plage de lignes, le score, le fournisseur/modèle et si nous avons replié depuis local vers des embeddings distants. Aucune charge utile de fichier complet n'est retournée.
- `memory_get` lit un fichier Markdown mémoire spécifique (relatif à l'espace de travail), optionnellement depuis une ligne de départ et pour N lignes. Les chemins en dehors de `MEMORY.md` / `memory/` sont rejetés.
- Les deux outils sont activés uniquement quand `memorySearch.enabled` est résolu à true pour l'agent.

### Ce qui est indexé (et quand)

- Type de fichier : Markdown uniquement (`MEMORY.md`, `memory/**/*.md`).
- Stockage de l'index : SQLite par agent à `~/.openclaw/memory/<agentId>.sqlite` (configurable via `agents.defaults.memorySearch.store.path`, supporte le token `{agentId}`).
- Fraîcheur : un watcher sur `MEMORY.md` + `memory/` marque l'index comme obsolète (anti-rebond 1,5s). La synchronisation est planifiée au démarrage de session, à la recherche ou à un intervalle et s'exécute de manière asynchrone. Les transcriptions de session utilisent des seuils de delta pour déclencher la synchronisation en arrière-plan.
- Déclencheurs de ré-indexation : l'index stocke le **fournisseur/modèle d'embedding + empreinte du point d'accès + paramètres de découpage**. Si l'un de ceux-ci change, OpenClaw réinitialise et ré-indexe automatiquement tout le magasin.

### Recherche hybride (BM25 + vecteur)

Quand activée, OpenClaw combine :

- **Similarité vectorielle** (correspondance sémantique, la formulation peut différer)
- **Pertinence par mots-clés BM25** (tokens exacts comme les identifiants, variables d'environnement, symboles de code)

Si la recherche plein texte n'est pas disponible sur votre plateforme, OpenClaw replie vers la recherche vectorielle uniquement.

#### Pourquoi hybride ?

La recherche vectorielle est excellente pour « ceci signifie la même chose » :

- « Mac Studio gateway host » vs « la machine qui exécute le Gateway »
- « debounce file updates » vs « éviter l'indexation à chaque écriture »

Mais elle peut être faible sur les tokens exacts à haute valeur :

- Identifiants (`a828e60`, `b3b9895a...`)
- Symboles de code (`memorySearch.query.hybrid`)
- Chaînes d'erreur (« sqlite-vec unavailable »)

BM25 (plein texte) est l'opposé : fort sur les tokens exacts, plus faible sur les paraphrases.
La recherche hybride est le compromis pragmatique : **utiliser les deux signaux de récupération** pour obtenir de bons résultats pour les requêtes en « langage naturel » et les requêtes « aiguille dans une botte de foin ».

#### Comment nous fusionnons les résultats (conception actuelle)

Schéma d'implémentation :

1. Récupérer un pool de candidats des deux côtés :

- **Vecteur** : top `maxResults * candidateMultiplier` par similarité cosinus.
- **BM25** : top `maxResults * candidateMultiplier` par rang BM25 FTS5 (plus bas est meilleur).

2. Convertir le rang BM25 en un score 0..1 :

- `textScore = 1 / (1 + max(0, bm25Rank))`

3. Unir les candidats par identifiant de morceau et calculer un score pondéré :

- `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Notes :

- `vectorWeight` + `textWeight` est normalisé à 1.0 dans la résolution de configuration, donc les poids se comportent comme des pourcentages.
- Si les embeddings ne sont pas disponibles (ou que le fournisseur retourne un vecteur nul), nous exécutons quand même BM25 et retournons les correspondances par mots-clés.
- Si FTS5 ne peut pas être créé, nous gardons la recherche vectorielle uniquement (pas d'échec dur).

Ce n'est pas « parfait en théorie RI », mais c'est simple, rapide et tend à améliorer le rappel/la précision sur de vraies notes.
Si nous voulons aller plus loin plus tard, les prochaines étapes courantes sont la Fusion de Rang Réciproque (RRF) ou la normalisation de score (min/max ou z-score) avant le mélange.

#### Pipeline de post-traitement

Après la fusion des scores vectoriels et par mots-clés, deux étapes de post-traitement optionnelles affinent la liste de résultats avant qu'elle n'atteigne l'agent :

```
Vecteur + Mots-clés -> Fusion pondérée -> Décroissance temporelle -> Tri -> MMR -> Résultats Top-K
```

Les deux étapes sont **désactivées par défaut** et peuvent être activées indépendamment.

#### Reclassement MMR (diversité)

Quand la recherche hybride retourne des résultats, plusieurs morceaux peuvent contenir du contenu similaire ou se chevauchant.
Par exemple, chercher « configuration réseau domestique » pourrait retourner cinq extraits presque identiques de différentes notes quotidiennes qui mentionnent toutes la même configuration de routeur.

**MMR (Maximal Marginal Relevance)** reclasse les résultats pour équilibrer pertinence et diversité, assurant que les premiers résultats couvrent différents aspects de la requête au lieu de répéter la même information.

Comment ça fonctionne :

1. Les résultats sont notés par leur pertinence originale (score pondéré vecteur + BM25).
2. MMR sélectionne itérativement les résultats qui maximisent : `lambda * pertinence - (1-lambda) * max_similarité_avec_sélectionnés`.
3. La similarité entre résultats est mesurée avec la similarité textuelle de Jaccard sur le contenu tokenisé.

Le paramètre `lambda` contrôle le compromis :

- `lambda = 1.0` -> pertinence pure (pas de pénalité de diversité)
- `lambda = 0.0` -> diversité maximale (ignoré la pertinence)
- Défaut : `0.7` (équilibré, léger biais de pertinence)

**Exemple : requête « configuration réseau domestique »**

Avec ces fichiers mémoire :

```
memory/2026-02-10.md  -> "Configured Omada router, set VLAN 10 for IoT devices"
memory/2026-02-08.md  -> "Configured Omada router, moved IoT to VLAN 10"
memory/2026-02-05.md  -> "Set up AdGuard DNS on 192.168.10.2"
memory/network.md     -> "Router: Omada ER605, AdGuard: 192.168.10.2, VLAN 10: IoT"
```

Sans MMR : top 3 résultats :

```
1. memory/2026-02-10.md  (score: 0.92)  <- routeur + VLAN
2. memory/2026-02-08.md  (score: 0.89)  <- routeur + VLAN (quasi-doublon !)
3. memory/network.md     (score: 0.85)  <- doc de référence
```

Avec MMR (lambda=0.7) : top 3 résultats :

```
1. memory/2026-02-10.md  (score: 0.92)  <- routeur + VLAN
2. memory/network.md     (score: 0.85)  <- doc de référence (divers !)
3. memory/2026-02-05.md  (score: 0.78)  <- AdGuard DNS (divers !)
```

Le quasi-doublon de février 8 disparaît, et l'agent obtient trois informations distinctes.

**Quand activer :** Si vous remarquez que `memory_search` retourne des extraits redondants ou quasi-doublons, surtout avec des notes quotidiennes qui répètent souvent des informations similaires d'un jour à l'autre.

#### Décroissance temporelle (boost de récence)

Les agents avec des notes quotidiennes accumulent des centaines de fichiers datés au fil du temps. Sans décroissance, une note bien formulée d'il y a six mois peut surclasser la mise à jour d'hier sur le même sujet.

La **décroissance temporelle** applique un multiplicateur exponentiel aux scores basé sur l'âge de chaque résultat, pour que les souvenirs récents se classent naturellement plus haut tandis que les anciens s'estompent :

```
decayedScore = score * e^(-lambda * ageInDays)
```

ou `lambda = ln(2) / halfLifeDays`.

Avec la demi-vie par défaut de 30 jours :

- Notes d'aujourd'hui : **100%** du score original
- Il y a 7 jours : **~84%**
- Il y a 30 jours : **50%**
- Il y a 90 jours : **12,5%**
- Il y a 180 jours : **~1,6%**

**Les fichiers permanents ne subissent jamais de décroissance :**

- `MEMORY.md` (fichier mémoire racine)
- Fichiers non datés dans `memory/` (par ex. `memory/projects.md`, `memory/network.md`)
- Ceux-ci contiennent des informations de référence durables qui doivent toujours se classer normalement.

**Les fichiers quotidiens datés** (`memory/YYYY-MM-DD.md`) utilisent la date extraite du nom de fichier. Les autres sources (par ex. transcriptions de session) utilisent par repli le temps de modification du fichier (`mtime`).

**Exemple : requête « quel est l'emploi du temps de Rod ? »**

Avec ces fichiers mémoire (aujourd'hui est le 10 février) :

```
memory/2025-09-15.md  -> "Rod works Mon-Fri, standup at 10am, pairing at 2pm"  (148 jours)
memory/2026-02-10.md  -> "Rod has standup at 14:15, 1:1 with Zeb at 14:45"    (aujourd'hui)
memory/2026-02-03.md  -> "Rod started new team, standup moved to 14:15"        (7 jours)
```

Sans décroissance :

```
1. memory/2025-09-15.md  (score: 0.91)  <- meilleure correspondance sémantique, mais obsolète !
2. memory/2026-02-10.md  (score: 0.82)
3. memory/2026-02-03.md  (score: 0.80)
```

Avec décroissance (halfLife=30) :

```
1. memory/2026-02-10.md  (score: 0.82 * 1.00 = 0.82)  <- aujourd'hui, pas de décroissance
2. memory/2026-02-03.md  (score: 0.80 * 0.85 = 0.68)  <- 7 jours, légère décroissance
3. memory/2025-09-15.md  (score: 0.91 * 0.03 = 0.03)  <- 148 jours, presque disparu
```

La note obsolète de septembre tombe au fond malgré la meilleure correspondance sémantique brute.

**Quand activer :** Si votre agent a des mois de notes quotidiennes et que vous constatez que les anciennes informations obsolètes surclassent le contexte récent. Une demi-vie de 30 jours fonctionne bien pour les workflows axés sur les notes quotidiennes ; augmentez-la (par ex. 90 jours) si vous référencez souvent des notes plus anciennes.

#### Configuration

Les deux fonctionnalités sont configurées sous `memorySearch.query.hybrid` :

```json5
agents: {
  defaults: {
    memorySearch: {
      query: {
        hybrid: {
          enabled: true,
          vectorWeight: 0.7,
          textWeight: 0.3,
          candidateMultiplier: 4,
          // Diversité : réduire les résultats redondants
          mmr: {
            enabled: true,    // défaut : false
            lambda: 0.7       // 0 = diversité max, 1 = pertinence max
          },
          // Récence : booster les souvenirs plus récents
          temporalDecay: {
            enabled: true,    // défaut : false
            halfLifeDays: 30  // le score diminue de moitié tous les 30 jours
          }
        }
      }
    }
  }
}
```

Vous pouvez activer chaque fonctionnalité indépendamment :

- **MMR seul** : utile quand vous avez beaucoup de notes similaires mais que l'âge n'a pas d'importance.
- **Décroissance temporelle seule** : utile quand la récence compte mais que vos résultats sont déjà divers.
- **Les deux** : recommandé pour les agents avec de longs historiques de notes quotidiennes.

### Cache d'embeddings

OpenClaw peut mettre en cache les **embeddings de morceaux** dans SQLite pour que la ré-indexation et les mises à jour fréquentes (surtout les transcriptions de session) ne ré-encodent pas le texte inchangé.

Configuration :

```json5
agents: {
  defaults: {
    memorySearch: {
      cache: {
        enabled: true,
        maxEntries: 50000
      }
    }
  }
}
```

### Recherche mémoire de session (expérimentale)

Vous pouvez optionnellement indexer les **transcriptions de session** et les afficher via `memory_search`. Ceci est protégé par un flag expérimental.

```json5
agents: {
  defaults: {
    memorySearch: {
      experimental: { sessionMemory: true },
      sources: ["memory", "sessions"]
    }
  }
}
```

Notes :

- L'indexation de session est **opt-in** (désactivée par défaut).
- Les mises à jour de session sont anti-rebondies et **indexées de manière asynchrone** une fois qu'elles dépassent les seuils de delta (meilleur effort).
- `memory_search` ne bloque jamais sur l'indexation ; les résultats peuvent être légèrement obsolètes jusqu'à ce que la synchronisation en arrière-plan se termine.
- Les résultats incluent toujours uniquement des extraits ; `memory_get` reste limité aux fichiers mémoire.
- L'indexation de session est isolée par agent (seuls les journaux de session de cet agent sont indexés).
- Les journaux de session résident sur le disque (`~/.openclaw/agents/<agentId>/sessions/*.jsonl`). Tout processus/utilisateur avec un accès au système de fichiers peut les lire, donc traitez l'accès disque comme la frontière de confiance. Pour une isolation plus stricte, exécutez les agents sous des utilisateurs OS ou hôtes séparés.

Seuils de delta (valeurs par défaut affichées) :

```json5
agents: {
  defaults: {
    memorySearch: {
      sync: {
        sessions: {
          deltaBytes: 100000,   // ~100 Ko
          deltaMessages: 50     // lignes JSONL
        }
      }
    }
  }
}
```

### Accélération vectorielle SQLite (sqlite-vec)

Quand l'extension sqlite-vec est disponible, OpenClaw stocke les embeddings dans une table virtuelle SQLite (`vec0`) et effectué les requêtes de distance vectorielle dans la base de données. Cela garde la recherche rapide sans charger chaque embedding en JS.

Configuration (optionnelle) :

```json5
agents: {
  defaults: {
    memorySearch: {
      store: {
        vector: {
          enabled: true,
          extensionPath: "/path/to/sqlite-vec"
        }
      }
    }
  }
}
```

Notes :

- `enabled` est à true par défaut ; quand désactivé, la recherche replie vers la similarité cosinus en processus sur les embeddings stockés.
- Si l'extension sqlite-vec est manquante ou échoue à se charger, OpenClaw journalise l'erreur et continue avec le repli JS (pas de table vectorielle).
- `extensionPath` surcharge le chemin sqlite-vec intégré (utile pour les builds personnalisés ou les emplacements d'installation non standards).

### Téléchargement automatique d'embeddings locaux

- Modèle d'embedding local par défaut : `hf:ggml-org/embeddinggemma-300m-qat-q8_0-GGUF/embeddinggemma-300m-qat-Q8_0.gguf` (~0,6 Go).
- Quand `memorySearch.provider = "local"`, `node-llama-cpp` résout `modelPath` ; si le GGUF est manquant il **télécharge automatiquement** vers le cache (ou `local.modelCacheDir` si défini), puis le charge. Les téléchargements reprennent en cas de nouvelle tentative.
- Exigence de build natif : exécuter `pnpm approve-builds`, choisir `node-llama-cpp`, puis `pnpm rebuild node-llama-cpp`.
- Repli : si la configuration locale échoue et `memorySearch.fallback = "openai"`, nous basculons automatiquement vers les embeddings distants (`openai/text-embedding-3-small` sauf surcharge) et enregistrons la raison.

### Exemple de point d'accès personnalisé compatible OpenAI

```json5
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_REMOTE_API_KEY",
        headers: {
          "X-Organization": "org-id",
          "X-Project": "project-id"
        }
      }
    }
  }
}
```

Notes :

- `remote.*` prend la priorité sur `models.providers.openai.*`.
- `remote.headers` fusionne avec les en-têtes OpenAI ; remote gagne en cas de conflit de clé. Omettre `remote.headers` pour utiliser les valeurs par défaut OpenAI.
