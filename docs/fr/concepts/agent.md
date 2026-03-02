---
summary: "Runtime de l'agent (pi-mono intégré), contrat de l'espace de travail et amorçage de session"
read_when:
  - Modification du runtime de l'agent, de l'amorçage de l'espace de travail ou du comportement des sessions
title: "Runtime de l'agent"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/agent.md
  workflow: manual
---

# Runtime de l'agent

OpenClaw exécute un seul runtime d'agent intégré dérivé de **pi-mono**.

## Espace de travail (requis)

OpenClaw utilise un seul répertoire d'espace de travail (`agents.defaults.workspace`) comme **unique** répertoire de travail (`cwd`) de l'agent pour les outils et le contexte.

Recommandé : utilisez `openclaw setup` pour créer `~/.openclaw/openclaw.json` s'il est manquant et initialiser les fichiers de l'espace de travail.

Disposition complète de l'espace de travail et guide de sauvegarde : [Espace de travail de l'agent](/concepts/agent-workspace)

Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent remplacer cela avec des espaces de travail par session sous `agents.defaults.sandbox.workspaceRoot` (voir [Configuration du Gateway](/gateway/configuration)).

## Fichiers d'amorçage (injectés)

Dans `agents.defaults.workspace`, OpenClaw attend ces fichiers modifiables par l'utilisateur :

- `AGENTS.md` — instructions de fonctionnement + « mémoire »
- `SOUL.md` — personnalité, limités, ton
- `TOOLS.md` — notes d'outils maintenues par l'utilisateur (par ex. `imsg`, `sag`, conventions)
- `BOOTSTRAP.md` — rituel de premier lancement unique (supprimé après complétion)
- `IDENTITY.md` — nom/style/emoji de l'agent
- `USER.md` — profil utilisateur + mode d'adresse préféré

Au premier tour d'une nouvelle session, OpenClaw injecte le contenu de ces fichiers directement dans le contexte de l'agent.

Les fichiers vides sont ignorés. Les fichiers volumineux sont réduits et tronqués avec un marqueur afin que les prompts restent légers (lire le fichier pour le contenu complet).

Si un fichier est manquant, OpenClaw injecte une seule ligne de marqueur « fichier manquant » (et `openclaw setup` créera un modèle par défaut sûr).

`BOOTSTRAP.md` est créé uniquement pour un **tout nouvel espace de travail** (aucun autre fichier d'amorçage présent). Si vous le supprimez après avoir terminé le rituel, il ne devrait pas être recréé lors des redémarrages ultérieurs.

Pour désactiver entièrement la création des fichiers d'amorçage (pour les espaces de travail pré-remplis), définissez :

```json5
{ agent: { skipBootstrap: true } }
```

## Outils intégrés

Les outils de base (read/exec/edit/write et outils système associés) sont toujours disponibles, sous réserve de la politique d'outils. `apply_patch` est optionnel et contrôle par `tools.exec.applyPatch`. `TOOLS.md` ne contrôle **pas** quels outils existent ; c'est un guide pour la façon dont _vous_ souhaitez qu'ils soient utilisés.

## Skills

OpenClaw charge les skills depuis trois emplacements (l'espace de travail a la priorité en cas de conflit de nom) :

- Intégrés (livrés avec l'installation)
- Gérés/locaux : `~/.openclaw/skills`
- Espace de travail : `<workspace>/skills`

Les skills peuvent être contrôlés par la configuration/l'environnement (voir `skills` dans [Configuration du Gateway](/gateway/configuration)).

## Intégration pi-mono

OpenClaw réutilise des éléments de la base de code pi-mono (modèles/outils), mais **la gestion des sessions, la découverte et le câblage des outils appartiennent à OpenClaw**.

- Pas de runtime pi-coding agent.
- Aucun paramètre `~/.pi/agent` ou `<workspace>/.pi` n'est consulté.

## Sessions

Les transcriptions de session sont stockées en JSONL à :

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

L'identifiant de session est stable et choisi par OpenClaw.
Les dossiers de session Pi/Tau hérités ne sont **pas** lus.

## Pilotage pendant le streaming

Lorsque le mode de file d'attente est `steer`, les messages entrants sont injectés dans l'exécution en cours. La file d'attente est vérifiée **après chaque appel d'outil** ; si un message en attente est présent, les appels d'outils restants du message assistant actuel sont ignorés (résultats d'outil en erreur avec « Skipped due to queued user message. »), puis le message utilisateur en attente est injecté avant la prochaine réponse assistant.

Lorsque le mode de file d'attente est `followup` ou `collect`, les messages entrants sont retenus jusqu'à la fin du tour actuel, puis un nouveau tour d'agent démarre avec les charges utiles en attente. Voir [File d'attente](/concepts/queue) pour le mode + comportement de debounce/cap.

Le streaming par blocs envoie les blocs assistant complétés dès qu'ils sont terminés ; il est **désactivé par défaut** (`agents.defaults.blockStreamingDefault: "off"`).
Réglez la limité via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end` ; par défaut text_end).
Contrôlez le découpage souple des blocs avec `agents.defaults.blockStreamingChunk` (par défaut 800-1200 caractères ; préfère les sauts de paragraphe, puis les retours à la ligne ; les phrases en dernier).
Fusionnez les fragments diffusés avec `agents.defaults.blockStreamingCoalesce` pour réduire le spam de lignes uniques (fusion basée sur l'inactivité avant l'envoi). Les canaux non-Telegram nécessitent un `*.blockStreaming: true` explicite pour activer les réponses par blocs.
Les résumés d'outils détaillés sont émis au démarrage de l'outil (sans debounce) ; l'interface Control diffuse en continu la sortie des outils via les événements agent lorsque disponible.
Plus de détails : [Streaming + découpage](/concepts/streaming).

## Références de modèles

Les références de modèles dans la configuration (par exemple `agents.defaults.model` et `agents.defaults.models`) sont analysées en divisant sur le **premier** `/`.

- Utilisez `provider/model` lors de la configuration des modèles.
- Si l'identifiant du modèle lui-même contient `/` (style OpenRouter), incluez le préfixe du fournisseur (exemple : `openrouter/moonshotai/kimi-k2`).
- Si vous omettez le fournisseur, OpenClaw traite l'entrée comme un alias ou un modèle pour le **fournisseur par défaut** (fonctionne uniquement lorsqu'il n'y a pas de `/` dans l'identifiant du modèle).

## Configuration (minimale)

Au minimum, définissez :

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (fortement recommandé)

---

_Suivant : [Conversations de groupe](/channels/group-messages)_
