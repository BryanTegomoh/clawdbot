---
summary: "Notes de recherche : système de mémoire hors-ligne pour les espaces de travail Clawd (Markdown comme source de vérité + index dérivé)"
read_when:
  - Conception de la mémoire d'espace de travail (~/.openclaw/workspace) au-delà des journaux Markdown quotidiens
  - Décision : CLI autonome vs intégration profonde dans OpenClaw
  - Ajout de rappel + réflexion hors-ligne (retain/recall/reflect)
title: "Recherche sur la mémoire d'espace de travail"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/experiments/research/memory.md
  workflow: manual
---

# Mémoire d'espace de travail v2 (hors-ligne) : notes de recherche

Cible : espace de travail de type Clawd (`agents.defaults.workspace`, par défaut `~/.openclaw/workspace`) où la « mémoire » est stockée comme un fichier Markdown par jour (`memory/YYYY-MM-DD.md`) plus un petit ensemble de fichiers stables (ex. `memory.md`, `SOUL.md`).

Ce document propose une architecture de mémoire **hors-ligne d'abord** qui conserve le Markdown comme source de vérité canonique et vérifiable, mais ajouté un **rappel structuré** (recherche, résumés d'entités, mises à jour de confiance) via un index dérivé.

## Pourquoi changer ?

La configuration actuelle (un fichier par jour) est excellente pour :

- le journaling « append-only »
- l'édition humaine
- la durabilité + l'auditabilité adossée à git
- la capture à faible friction (« écrivez-le simplement »)

Elle est faible pour :

- la récupération à haut rappel (« qu'avons-nous décidé à propos de X ? », « la dernière fois que nous avons essayé Y ? »)
- les réponses centrées sur les entités (« parle-moi d'Alice / du Castle / de warelay ») sans relire de nombreux fichiers
- la stabilité des opinions/préférences (et les preuves quand elles changent)
- les contraintes temporelles (« qu'est-ce qui était vrai pendant novembre 2025 ? ») et la résolution de conflits

## Objectifs de conception

- **Hors-ligne** : fonctionne sans réseau ; peut tourner sur laptop/Castle ; pas de dépendance cloud.
- **Explicable** : les éléments récupérés doivent être attribuables (fichier + emplacement) et séparables de l'inférence.
- **Faible cérémonie** : le journaling quotidien reste en Markdown, pas de travail de schéma lourd.
- **Incrémental** : la v1 est utile avec FTS uniquement ; le sémantique/vectoriel et les graphes sont des améliorations optionnelles.
- **Agent-friendly** : rend facile le « rappel dans les budgets de tokens » (retourner de petits paquets de faits).

## Modèle directeur (Hindsight x Letta)

Deux pièces à combiner :

1. **Boucle de contrôle de type Letta/MemGPT**

- conserver un petit « noyau » toujours en contexte (persona + faits clés de l'utilisateur)
- tout le reste est hors-contexte et récupère via des outils
- les écritures mémoire sont des appels d'outils explicites (append/replace/insert), persistés, puis ré-injectés au tour suivant

2. **Substrat mémoire de type Hindsight**

- séparer ce qui est observé vs ce qui est cru vs ce qui est résumé
- supporter retain/recall/reflect
- opinions portant une confiance qui peuvent évoluer avec les preuves
- récupération consciente des entités + requêtes temporelles (même sans graphes de connaissances complets)

## Architecture proposée (Markdown comme source de vérité + index dérivé)

### Store canonique (compatible git)

Conserver `~/.openclaw/workspace` comme mémoire canonique lisible par l'humain.

Organisation suggérée de l'espace de travail :

```
~/.openclaw/workspace/
  memory.md                    # petit : faits durables + préférences (noyau)
  memory/
    YYYY-MM-DD.md              # journal quotidien (append ; narratif)
  bank/                        # pages mémoire « typées » (stables, vérifiables)
    world.md                   # faits objectifs sur le monde
    experience.md              # ce que l'agent a fait (première personne)
    opinions.md                # prefs/jugements subjectifs + confiance + pointeurs de preuves
    entities/
      Peter.md
      The-Castle.md
      warelay.md
      ...
```

Notes :

- **Le journal quotidien reste un journal quotidien**. Pas besoin de le transformer en JSON.
- Les fichiers `bank/` sont **curatés**, produits par des jobs de réflexion, et peuvent toujours être édités manuellement.
- `memory.md` reste « petit + noyau » : les choses que vous voulez que Clawd voie à chaque session.

### Store dérivé (rappel machine)

Ajouter un index dérivé sous l'espace de travail (pas nécessairement suivi par git) :

```
~/.openclaw/workspace/.memory/index.sqlite
```

Adossé à :

- Un schéma SQLite pour les faits + liens d'entités + métadonnées d'opinion
- SQLite **FTS5** pour le rappel lexical (rapide, petit, hors-ligne)
- Table d'embeddings optionnelle pour le rappel sémantique (toujours hors-ligne)

L'index est toujours **reconstituable depuis le Markdown**.

## Retain / Recall / Reflect (boucle opérationnelle)

### Retain : normaliser les journaux quotidiens en « faits »

L'insight clé de Hindsight qui compte ici : stocker des **faits narratifs, autonomes**, pas de petits extraits.

Règle pratique pour `memory/YYYY-MM-DD.md` :

- en fin de journée (ou pendant), ajouter une section `## Retain` avec 2 a 5 puces qui sont :
  - narratives (contexte inter-tours préservé)
  - autonomes (compréhensibles seules plus tard)
  - taguées avec type + mentions d'entités

Exemple :

```
## Retain
- W @Peter: Currently in Marrakech (Nov 27–Dec 1, 2025) for Andy's birthday.
- B @warelay: I fixed the Baileys WS crash by wrapping connection.update handlers in try/catch (see memory/2025-11-27.md).
- O(c=0.95) @Peter: Prefers concise replies (<1500 chars) on WhatsApp; long content goes into files.
```

Parsing minimal :

- Préfixe de type : `W` (monde), `B` (expérience/biographique), `O` (opinion), `S` (observation/résumé ; généralement généré)
- Entités : `@Peter`, `@warelay`, etc (les slugs correspondent à `bank/entities/*.md`)
- Confiance d'opinion : `O(c=0.0..1.0)` optionnel

Si vous ne voulez pas que les auteurs y réfléchissent : le job de réflexion peut inférer ces puces à partir du reste du journal, mais avoir une section `## Retain` explicite est le « levier de qualité » le plus facile.

### Recall : requêtes sur l'index dérivé

Le rappel devrait supporter :

- **lexical** : « trouver des termes/noms/commandes exacts » (FTS5)
- **entité** : « parle-moi de X » (pages d'entité + faits liés aux entités)
- **temporel** : « qu'est-il arrivé autour du 27 novembre » / « depuis la semaine dernière »
- **opinion** : « que préfère Peter ? » (avec confiance + preuves)

Le format de retour devrait être agent-friendly et citer les sources :

- `kind` (`world|experience|opinion|observation`)
- `timestamp` (jour source, ou plage temporelle extraite si présente)
- `entities` (`["Peter","warelay"]`)
- `content` (le fait narratif)
- `source` (`memory/2025-11-27.md#L12` etc)

### Reflect : produire des pages stables + mettre à jour les croyances

La réflexion est un job planifié (quotidien ou heartbeat `ultrathink`) qui :

- met à jour les `bank/entities/*.md` à partir des faits récents (résumés d'entités)
- met à jour la confiance de `bank/opinions.md` basée sur le renforcement/contradiction
- propose optionnellement des éditions à `memory.md` (faits durables « noyau »)

Évolution des opinions (simple, explicable) :

- chaque opinion a :
  - un énoncé
  - une confiance `c ∈ [0,1]`
  - last_updated
  - des liens de preuves (IDs de faits supportant + contredisant)
- quand de nouveaux faits arrivent :
  - trouver les opinions candidates par chevauchement d'entités + similarité (FTS d'abord, embeddings ensuite)
  - mettre à jour la confiance par petits deltas ; les grands sauts nécessitent une contradiction forte + preuves répétées

## Intégration CLI : autonome vs intégration profonde

Recommandation : **intégration profonde dans OpenClaw**, mais conserver une bibliothèque centrale séparable.

### Pourquoi intégrer dans OpenClaw ?

- OpenClaw connaît déjà :
  - le chemin de l'espace de travail (`agents.defaults.workspace`)
  - le modèle de session + les heartbeats
  - les patterns de journalisation + dépannage
- Vous voulez que l'agent lui-même appelle les outils :
  - `openclaw memory recall "…" --k 25 --since 30d`
  - `openclaw memory reflect --since 7d`

### Pourquoi quand même séparer une bibliothèque ?

- conserver la logique mémoire testable sans gateway/runtime
- réutiliser depuis d'autres contextes (scripts locaux, future application de bureau, etc.)

Forme :
L'outillage mémoire est destiné à être une petite couche CLI + bibliothèque, mais ceci est exploratoire uniquement.

## « S-Collide » / SuCo : quand l'utiliser (recherche)

Si « S-Collide » fait référence à **SuCo (Subspace Collision)** : c'est une approche de récupération ANN qui cible de forts compromis rappel/latence en utilisant des collisions apprises/structurées dans des sous-espaces (article : arXiv 2411.14754, 2024).

Prise pragmatique pour `~/.openclaw/workspace` :

- **ne commencez pas** avec SuCo.
- commencez avec SQLite FTS + embeddings simples (optionnel) ; vous obtiendrez la plupart des gains UX immédiatement.
- considérez les solutions de classe SuCo/HNSW/ScaNN uniquement une fois que :
  - le corpus est grand (dizaines/centaines de milliers de chunks)
  - la recherche brute-force d'embeddings devient trop lente
  - la qualité du rappel est significativement goulottée par la recherche lexicale

Alternatives hors-ligne (en complexité croissante) :

- SQLite FTS5 + filtrés de métadonnées (zéro ML)
- Embeddings + force brute (fonctionne étonnamment loin si le nombre de chunks est faible)
- Index HNSW (courant, robuste ; nécessite un binding de bibliothèque)
- SuCo (niveau recherche ; attractif s'il existe une implémentation solide que vous pouvez embarquer)

Question ouverte :

- quel est le **meilleur** modèle d'embedding hors-ligne pour la « mémoire d'assistant personnel » sur vos machines (laptop + desktop) ?
  - si vous avez déjà Ollama : faites l'embedding avec un modèle local ; sinon, livrez un petit modèle d'embedding dans la chaîne d'outils.

## Pilote utile minimal

Si vous voulez une version minimale, mais quand même utile :

- Ajouter les pages d'entités `bank/` et une section `## Retain` dans les journaux quotidiens.
- Utiliser SQLite FTS pour le rappel avec citations (chemin + numéros de ligne).
- Ajouter les embeddings uniquement si la qualité du rappel ou l'échelle l'exige.

## Références

- Concepts Letta / MemGPT : « core memory blocks » + « archival memory » + mémoire auto-éditée pilotée par outils.
- Rapport technique Hindsight : « retain / recall / reflect », mémoire à quatre réseaux, extraction de faits narratifs, évolution de confiance d'opinion.
- SuCo : arXiv 2411.14754 (2024) : récupération de plus proches voisins approximatifs par « Subspace Collision ».
