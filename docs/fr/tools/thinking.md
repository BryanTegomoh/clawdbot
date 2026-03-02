---
summary: "Syntaxe des directives pour /think + /verbose et leur effet sur le raisonnement du modèle"
read_when:
  - Ajustement de l'analyse des directives thinking ou verbose ou de leurs valeurs par défaut
title: "Niveaux de réflexion"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/thinking.md
  workflow: manual
---

# Niveaux de réflexion (directives /think)

## Ce que cela fait

- Directive en ligne dans tout corps de message entrant : `/t <niveau>`, `/think:<niveau>` ou `/thinking <niveau>`.
- Niveaux (alias) : `off | minimal | low | medium | high | xhigh` (modèles GPT-5.2 + Codex uniquement)
  - minimal → « think »
  - low → « think hard »
  - medium → « think harder »
  - high → « ultrathink » (budget maximal)
  - xhigh → « ultrathink+ » (modèles GPT-5.2 + Codex uniquement)
  - `x-high`, `x_high`, `extra-high`, `extra high` et `extra_high` correspondent à `xhigh`.
  - `highest`, `max` correspondent à `high`.
- Notes sur les fournisseurs :
  - Z.AI (`zai/*`) ne prend en charge que le thinking binaire (`on`/`off`). Tout niveau non-`off` est traité comme `on` (mappé à `low`).

## Ordre de résolution

1. Directive en ligne sur le message (s'applique uniquement à ce message).
2. Remplacement de session (défini en envoyant un message contenant uniquement la directive).
3. Valeur par défaut globale (`agents.defaults.thinkingDefault` dans la configuration).
4. Repli : low pour les modèles capables de raisonnement ; off sinon.

## Définir un défaut de session

- Envoyez un message qui est **uniquement** la directive (espaces autorisés), par ex. `/think:medium` ou `/t high`.
- Cela persiste pour la session courante (par expéditeur par défaut) ; effacé par `/think:off` ou réinitialisation de session inactive.
- Une réponse de confirmation est envoyée (`Thinking level set to high.` / `Thinking disabled.`). Si le niveau est invalide (par ex. `/thinking big`), la commande est rejetée avec une indication et l'état de la session reste inchangé.
- Envoyez `/think` (ou `/think:`) sans argument pour voir le niveau de réflexion actuel.

## Application par agent

- **Pi embarqué** : le niveau résolu est transmis au runtime de l'agent Pi en processus.

## Directives verbose (/verbose ou /v)

- Niveaux : `on` (minimal) | `full` | `off` (défaut).
- Un message contenant uniquement la directive bascule le verbose de session et répond `Verbose logging enabled.` / `Verbose logging disabled.` ; les niveaux invalides renvoient une indication sans changer l'état.
- `/verbose off` enregistre un remplacement de session explicite ; effacez-le via l'interface Sessions en choisissant `inherit`.
- Une directive en ligne n'affecte que ce message ; les valeurs par défaut de session/globales s'appliquent sinon.
- Envoyez `/verbose` (ou `/verbose:`) sans argument pour voir le niveau verbose actuel.
- Lorsque verbose est activé, les agents qui émettent des résultats d'outils structurés (Pi, autres agents JSON) envoient chaque appel d'outil comme son propre message de métadonnées, préfixé par `<emoji> <nom-outil>: <arg>` lorsque disponible (chemin/commande). Ces résumés d'outils sont envoyés dès que chaque outil démarre (bulles séparées), pas comme des deltas en streaming.
- Les résumés d'échec d'outils restent visibles en mode normal, mais les suffixes de détail d'erreur bruts sont masqués sauf si verbose est `on` ou `full`.
- Lorsque verbose est `full`, les sorties d'outils sont également transmises après achèvement (bulle séparée, tronquée à une longueur sûre). Si vous basculez `/verbose on|full|off` pendant qu'une exécution est en cours, les bulles d'outils suivantes respectent le nouveau paramètre.

## Visibilité du raisonnement (/reasoning)

- Niveaux : `on|off|stream`.
- Un message contenant uniquement la directive bascule l'affichage des blocs de réflexion dans les réponses.
- Lorsqu'activé, le raisonnement est envoyé comme un **message séparé** préfixé par `Reasoning:`.
- `stream` (Telegram uniquement) : diffuse le raisonnement dans la bulle brouillon Telegram pendant la génération de la réponse, puis envoie la réponse finale sans le raisonnement.
- Alias : `/reason`.
- Envoyez `/reasoning` (ou `/reasoning:`) sans argument pour voir le niveau de raisonnement actuel.

## Voir aussi

- La documentation du mode élevé se trouve dans [Mode élevé](/tools/elevated).

## Heartbeats

- Le corps de la sonde heartbeat est le prompt heartbeat configuré (défaut : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Les directives en ligne dans un message heartbeat s'appliquent comme d'habitude (mais évitez de changer les valeurs par défaut de session depuis les heartbeats).
- La livraison heartbeat est par défaut limitée au payload final uniquement. Pour envoyer également le message séparé `Reasoning:` (lorsque disponible), définissez `agents.defaults.heartbeat.includeReasoning: true` ou par agent `agents.list[].heartbeat.includeReasoning: true`.

## Interface web de chat

- Le sélecteur de réflexion du chat web reflète le niveau stocké de la session depuis le magasin de session entrant/configuration au chargement de la page.
- Choisir un autre niveau ne s'applique qu'au prochain message (`thinkingOnce`) ; après l'envoi, le sélecteur revient au niveau de session stocke.
- Pour changer la valeur par défaut de session, envoyez une directive `/think:<niveau>` (comme avant) ; le sélecteur la reflétera après le prochain rechargement.
