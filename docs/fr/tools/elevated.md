---
summary: "Mode élevé d'exécution et directives /elevated"
read_when:
  - Ajustement des valeurs par défaut du mode élevé, des listes d'autorisation ou du comportement des commandes slash
title: "Mode élevé"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/elevated.md
  workflow: manual
---

# Mode élevé (directives /elevated)

## Ce que cela fait

- `/elevated on` s'exécute sur l'hôte du Gateway et conserve les approbations exec (identique à `/elevated ask`).
- `/elevated full` s'exécute sur l'hôte du Gateway **et** approuve automatiquement les exec (ignoré les approbations exec).
- `/elevated ask` s'exécute sur l'hôte du Gateway mais conserve les approbations exec (identique à `/elevated on`).
- `on`/`ask` ne forcent **pas** `exec.security=full` ; la politique security/ask configurée s'applique toujours.
- Ne change le comportement que lorsque l'agent est **sandboxé** (sinon exec s'exécute déjà sur l'hôte).
- Formes de directive : `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- Seuls `on|off|ask|full` sont acceptés ; toute autre valeur renvoie une indication et ne change pas l'état.

## Ce que cela contrôle (et ce que cela ne contrôle pas)

- **Portes de disponibilité** : `tools.elevated` est la base globale. `agents.list[].tools.elevated` peut davantage restreindre elevated par agent (les deux doivent autoriser).
- **État par session** : `/elevated on|off|ask|full` définit le niveau élevé pour la clé de session courante.
- **Directive en ligne** : `/elevated on|ask|full` dans un message s'applique uniquement à ce message.
- **Groupes** : dans les chats de groupe, les directives elevated ne sont honorées que lorsque l'agent est mentionné. Les messages contenant uniquement des commandes qui contournent les exigences de mention sont traités comme mentionnés.
- **Exécution sur l'hôte** : elevated force `exec` sur l'hôte du Gateway ; `full` définit également `security=full`.
- **Approbations** : `full` ignoré les approbations exec ; `on`/`ask` les respectent lorsque les règles allowlist/ask l'exigent.
- **Agents non sandboxés** : sans effet sur la localisation ; affecte uniquement le gating, la journalisation et le statut.
- **La politique d'outils s'applique toujours** : si `exec` est refusé par la politique d'outils, elevated ne peut pas être utilisé.
- **Séparé de `/exec`** : `/exec` ajuste les valeurs par défaut par session pour les expéditeurs autorisés et ne nécessite pas elevated.

## Ordre de résolution

1. Directive en ligne sur le message (s'applique uniquement à ce message).
2. Remplacement de session (défini en envoyant un message contenant uniquement la directive).
3. Valeur par défaut globale (`agents.defaults.elevatedDefault` dans la configuration).

## Définir un défaut de session

- Envoyez un message qui est **uniquement** la directive (espaces autorisés), par ex. `/elevated full`.
- Une réponse de confirmation est envoyée (`Elevated mode set to full...` / `Elevated mode disabled.`).
- Si l'accès elevated est désactivé ou si l'expéditeur n'est pas dans la liste d'autorisation approuvée, la directive répond avec une erreur actionnable et ne change pas l'état de la session.
- Envoyez `/elevated` (ou `/elevated:`) sans argument pour voir le niveau élevé actuel.

## Disponibilité + listes d'autorisation

- Porte de fonctionnalité : `tools.elevated.enabled` (la valeur par défaut peut être désactivée via la configuration même si le code le supporte).
- Liste d'autorisation des expéditeurs : `tools.elevated.allowFrom` avec des listes d'autorisation par fournisseur (par ex. `discord`, `whatsapp`).
- Les entrées de liste d'autorisation non préfixées ne correspondent qu'aux valeurs d'identité limitées à l'expéditeur (`SenderId`, `SenderE164`, `From`) ; les champs de routage du destinataire ne sont jamais utilisés pour l'autorisation elevated.
- Les métadonnées mutables de l'expéditeur nécessitent des préfixes explicites :
  - `name:<valeur>` correspond à `SenderName`
  - `username:<valeur>` correspond à `SenderUsername`
  - `tag:<valeur>` correspond à `SenderTag`
  - `id:<valeur>`, `from:<valeur>`, `e164:<valeur>` sont disponibles pour le ciblage explicite d'identité
- Porte par agent : `agents.list[].tools.elevated.enabled` (optionnel ; ne peut que restreindre davantage).
- Liste d'autorisation par agent : `agents.list[].tools.elevated.allowFrom` (optionnel ; lorsqu'elle est définie, l'expéditeur doit correspondre aux **deux** listes globale + par agent).
- Repli Discord : si `tools.elevated.allowFrom.discord` est omis, la liste `channels.discord.allowFrom` est utilisée en repli (ancien : `channels.discord.dm.allowFrom`). Définissez `tools.elevated.allowFrom.discord` (même `[]`) pour remplacer. Les listes par agent n'utilisent **pas** le repli.
- Toutes les portes doivent être franchies ; sinon elevated est traité comme non disponible.

## Journalisation + statut

- Les appels exec élevés sont journalisés au niveau info.
- Le statut de session inclut le mode élevé (par ex. `elevated=ask`, `elevated=full`).
