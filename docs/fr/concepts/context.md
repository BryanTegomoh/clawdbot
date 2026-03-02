---
summary: "Contexte : ce que le modèle voit, comment il est construit, et comment l'inspecter"
read_when:
  - Vous souhaitez comprendre ce que « contexte » signifie dans OpenClaw
  - Vous déboguez pourquoi le modèle « sait » quelque chose (ou l'a oublié)
  - Vous souhaitez réduire la surcharge du contexte (/context, /status, /compact)
title: "Contexte"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/context.md
  workflow: manual
---

# Contexte

Le « contexte » est **tout ce qu'OpenClaw envoie au modèle pour une exécution**. Il est limité par la **fenêtre de contexte** du modèle (limité de tokens).

Modèle mental pour débutant :

- **Prompt système** (construit par OpenClaw) : règles, outils, liste de skills, heure/runtime, et fichiers de l'espace de travail injectés.
- **Historique de conversation** : vos messages + les messages de l'assistant pour cette session.
- **Appels d'outils/résultats + pièces jointes** : sortie de commandes, lectures de fichiers, images/audio, etc.

Le contexte _n'est pas la même chose_ que la « mémoire » : la mémoire peut être stockée sur disque et rechargée plus tard ; le contexte est ce qui se trouve à l'intérieur de la fenêtre actuelle du modèle.

## Démarrage rapide (inspecter le contexte)

- `/status` → vue rapide « quelle est l'occupation de ma fenêtre ? » + paramètres de session.
- `/context list` → ce qui est injecté + tailles approximatives (par fichier + totaux).
- `/context detail` → décomposition détaillée : par fichier, par taille de schéma d'outil, par taille d'entrée de skill, et taille du prompt système.
- `/usage tokens` → ajouter un pied de page d'utilisation par réponse aux réponses normales.
- `/compact` → résumer l'historique ancien en une entrée compacte pour libérer de l'espace dans la fenêtre.

Voir aussi : [Commandes slash](/tools/slash-commands), [Utilisation de tokens et coûts](/reference/token-use), [Compaction](/concepts/compaction).

## Exemple de sortie

Les valeurs varient selon le modèle, le fournisseur, la politique d'outils et le contenu de votre espace de travail.

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Ce qui compte dans la fenêtre de contexte

Tout ce que le modèle reçoit compte, y compris :

- Prompt système (toutes les sections).
- Historique de conversation.
- Appels d'outils + résultats d'outils.
- Pièces jointes/transcriptions (images/audio/fichiers).
- Résumés de compaction et artefacts d'élagage.
- « Wrappers » du fournisseur ou en-têtes cachés (non visibles, mais comptés).

## Comment OpenClaw construit le prompt système

Le prompt système est **propriété d'OpenClaw** et reconstruit à chaque exécution. Il inclut :

- Liste d'outils + descriptions courtes.
- Liste de skills (métadonnées uniquement ; voir ci-dessous).
- Emplacement de l'espace de travail.
- Heure (UTC + heure utilisateur convertie si configurée).
- Métadonnées du runtime (hôte/OS/modèle/thinking).
- Fichiers bootstrap de l'espace de travail injectés sous **Project Context**.

Décomposition complète : [Prompt système](/concepts/system-prompt).

## Fichiers de l'espace de travail injectés (Project Context)

Par défaut, OpenClaw injecte un ensemble fixe de fichiers de l'espace de travail (s'ils sont présents) :

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (premier lancement uniquement)

Les fichiers volumineux sont tronqués par fichier en utilisant `agents.defaults.bootstrapMaxChars` (défaut `20000` caractères). OpenClaw applique également un plafond total d'injection bootstrap sur l'ensemble des fichiers avec `agents.defaults.bootstrapTotalMaxChars` (défaut `150000` caractères). `/context` affiche les tailles **brutes vs injectées** et si la troncature a eu lieu.

## Skills : ce qui est injecté vs chargé à la demande

Le prompt système inclut une **liste compacte de skills** (nom + description + emplacement). Cette liste a une surcharge réelle.

Les instructions de skill ne sont _pas_ incluses par défaut. Le modèle est censé utiliser `read` pour lire le `SKILL.md` du skill **uniquement quand c'est nécessaire**.

## Outils : il y a deux coûts

Les outils affectent le contexte de deux manières :

1. **Texte de la liste d'outils** dans le prompt système (ce que vous voyez comme « Tooling »).
2. **Schémas d'outils** (JSON). Ceux-ci sont envoyés au modèle pour qu'il puisse appeler les outils. Ils comptent dans le contexte même si vous ne les voyez pas comme du texte brut.

`/context detail` décompose les plus gros schémas d'outils pour que vous puissiez voir ce qui domine.

## Commandes, directives et « raccourcis en ligne »

Les commandes slash sont gérées par le Gateway. Il y a quelques comportements différents :

- **Commandes autonomes** : un message qui n'est que `/...` s'exécute comme une commande.
- **Directives** : `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` sont supprimées avant que le modèle ne voie le message.
  - Les messages ne contenant que des directives persistent les paramètres de session.
  - Les directives en ligne dans un message normal agissent comme des indices par message.
- **Raccourcis en ligne** (expéditeurs autorisés uniquement) : certains tokens `/...` dans un message normal peuvent s'exécuter immédiatement (exemple : « hey /status »), et sont supprimés avant que le modèle ne voie le texte restant.

Détails : [Commandes slash](/tools/slash-commands).

## Sessions, compaction et élagage (ce qui persiste)

Ce qui persiste entre les messages dépend du mécanisme :

- **L'historique normal** persiste dans la transcription de session jusqu'à compaction/élagage par politique.
- **La compaction** persiste un résumé dans la transcription et conserve les messages récents intacts.
- **L'élagage** supprimé les anciens résultats d'outils du prompt _en mémoire_ pour une exécution, mais ne réécrit pas la transcription.

Docs : [Session](/concepts/session), [Compaction](/concepts/compaction), [Élagage de session](/concepts/session-pruning).

## Ce que `/context` rapporte réellement

`/context` préfère le dernier rapport du prompt système **construit à l'exécution** lorsqu'il est disponible :

- `System prompt (run)` = capturé depuis la dernière exécution intégrée (capable d'outils) et persisté dans le magasin de sessions.
- `System prompt (estimate)` = calculé à la volée quand aucun rapport d'exécution n'existe (ou lors de l'exécution via un backend CLI qui ne génère pas le rapport).

Dans les deux cas, il rapporte les tailles et les principaux contributeurs ; il ne **dump pas** le prompt système complet ni les schémas d'outils.
