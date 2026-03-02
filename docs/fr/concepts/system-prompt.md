---
summary: "Contenu du prompt système OpenClaw et comment il est assemble"
read_when:
  - Modification du texte du prompt système, de la liste des outils ou des sections heure/heartbeat
  - Modification du comportement d'injection du bootstrap ou des Skills dans l'espace de travail
title: "Prompt Système"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/system-prompt.md
  workflow: manual
---

# Prompt Système

OpenClaw construit un prompt système personnalisé pour chaque exécution d'agent. Le prompt est **propriété d'OpenClaw** et n'utilisé pas le prompt par défaut de pi-coding-agent.

Le prompt est assemble par OpenClaw et injecte dans chaque exécution d'agent.

## Structure

Le prompt est intentionnellement compact et utilise des sections fixes :

- **Outillage** : liste des outils actuels + descriptions courtes.
- **Sécurité** : rappel court des garde-fous pour eviter les comportements de recherche de pouvoir ou le contournement de la supervision.
- **Skills** (lorsque disponibles) : indique au modèle comment charger les instructions de Skills a la demande.
- **Mise a jour automatique OpenClaw** : comment exécuter `config.apply` et `update.run`.
- **Espace de travail** : répertoire de travail (`agents.defaults.workspace`).
- **Documentation** : chemin local vers la documentation OpenClaw (depot ou package npm) et quand la consulter.
- **Fichiers de l'espace de travail (injectés)** : indique que les fichiers de bootstrap sont inclus ci-dessous.
- **Bac a sable** (lorsque activé) : indique l'environnement d'exécution en bac a sable, les chemins du bac a sable, et si l'exécution avec privileges élevés est disponible.
- **Date et heure actuelles** : heure locale de l'utilisateur, fuseau horaire et format horaire.
- **Balises de réponse** : syntaxe optionnelle de balises de réponse pour les fournisseurs compatibles.
- **Heartbeats** : prompt de heartbeat et comportement d'acquittement.
- **Environnement d'exécution** : hôte, OS, nœud, modèle, racine du depot (lorsque détectée), niveau de reflexion (une ligne).
- **Raisonnement** : niveau de visibilité actuel + indication de bascule /reasoning.

Les garde-fous de sécurité dans le prompt système sont consultatifs. Ils guident le comportement du modèle mais n'appliquent pas de politique. Utilisez la politique d'outils, les approbations d'exécution, le bac a sable et les listes d'autorisation de canal pour une application stricte ; les operateurs peuvent désactiver ces mecanismes par conception.

## Modes de prompt

OpenClaw peut générer des prompts système plus petits pour les sous-agents. L'environnement d'exécution définit un `promptMode` pour chaque exécution (ce n'est pas une configuration exposee a l'utilisateur) :

- `full` (par défaut) : inclut toutes les sections ci-dessus.
- `minimal` : utilisé pour les sous-agents ; omet **Skills**, **Rappel mémoire**, **Mise a jour automatique OpenClaw**, **Alias de modèles**, **Identité utilisateur**, **Balises de réponse**, **Messagerie**, **Réponses silencieuses** et **Heartbeats**. L'outillage, la **Sécurité**, l'espace de travail, le bac a sable, la date et heure actuelles (lorsque connues), l'environnement d'exécution et le contexte injecte restent disponibles.
- `none` : retourne uniquement la ligne d'identité de base.

Lorsque `promptMode=minimal`, les prompts injectés supplementaires sont etiquetes **Contexte de sous-agent** au lieu de **Contexte de chat de groupe**.

## Injection du bootstrap de l'espace de travail

Les fichiers de bootstrap sont tronques et ajoutés sous **Contexte du projet** afin que le modèle voie le contexte d'identité et de profil sans avoir besoin de lectures explicites :

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (uniquement sur les espaces de travail tout neufs)
- `MEMORY.md` et/ou `memory.md` (lorsque présents dans l'espace de travail ; l'un ou les deux peuvent être injectés)

Tous ces fichiers sont **injectés dans la fenêtre de contexte** a chaque tour, ce qui signifie qu'ils consomment des tokens. Gardez-les concis, en particulier `MEMORY.md`, qui peut grossir au fil du temps et entrainer une utilisation du contexte anormalement élevée et une compaction plus frequente.

> **Note :** les fichiers quotidiens `memory/*.md` ne sont **pas** injectés automatiquement. Ils sont accessibles a la demande via les outils `memory_search` et `memory_get`, et ne comptent donc pas dans la fenêtre de contexte sauf si le modèle les lit explicitement.

Les fichiers volumineux sont tronques avec un marqueur. La taille maximale par fichier est controlee par `agents.defaults.bootstrapMaxChars` (par défaut : 20000). Le contenu total de bootstrap injecte sur l'ensemble des fichiers est plafonne par `agents.defaults.bootstrapTotalMaxChars` (par défaut : 150000). Les fichiers manquants injectent un court marqueur de fichier absent.

Les sessions de sous-agents n'injectent que `AGENTS.md` et `TOOLS.md` (les autres fichiers de bootstrap sont filtrés pour garder le contexte du sous-agent réduit).

Des hooks internes peuvent intercepter cette étape via `agent:bootstrap` pour modifier ou remplacer les fichiers de bootstrap injectés (par exemple, remplacer `SOUL.md` par un persona alternatif).

Pour inspecter la contribution de chaque fichier injecte (brut vs injecte, troncature, plus la surcharge des schemas d'outils), utilisez `/context list` ou `/context detail`. Voir [Contexte](/concepts/context).

## Gestion de l'heure

Le prompt système inclut une section dédiée **Date et heure actuelles** lorsque le fuseau horaire de l'utilisateur est connu. Pour maintenir la stabilité du cache du prompt, il n'inclut désormais que le **fuseau horaire** (pas d'horloge dynamique ni de format horaire).

Utilisez `session_status` lorsque l'agent a besoin de l'heure actuelle ; la carte de statut inclut une ligne d'horodatage.

Configurez avec :

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Voir [Date et heure](/date-time) pour les details complets du comportement.

## Skills

Lorsque des Skills éligibles existent, OpenClaw injecte une **liste compacte des Skills disponibles** (`formatSkillsForPrompt`) qui inclut le **chemin du fichier** pour chaque Skill. Le prompt indique au modèle d'utiliser `read` pour charger le SKILL.md a l'emplacement indique (espace de travail, géré ou intégré). Si aucun Skill n'est éligible, la section Skills est omise.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Cela maintient le prompt de base petit tout en permettant l'utilisation ciblee des Skills.

## Documentation

Lorsque disponible, le prompt système inclut une section **Documentation** qui pointe vers le répertoire local de documentation OpenClaw (soit `docs/` dans l'espace de travail du depot, soit les docs du package npm intégré) et mentionne également le miroir public, le depot source, le Discord communautaire et ClawHub ([https://clawhub.com](https://clawhub.com)) pour la Decouverte de Skills. Le prompt indique au modèle de consulter d'abord la documentation locale pour le comportement, les commandes, la configuration ou l'architecture d'OpenClaw, et d'exécuter `openclaw status` lui-même lorsque possible (en demandant a l'utilisateur uniquement lorsqu'il n'a pas accès).
