---
summary: "Commandes slash : texte vs natives, configuration et commandes supportées"
read_when:
  - Utilisation ou configuration des commandes de chat
  - Débogage du routage des commandes ou des permissions
title: "Commandes slash"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/slash-commands.md
  workflow: manual
---

# Commandes slash

Les commandes sont gérées par le Gateway. La plupart des commandes doivent être envoyées comme un message **autonome** commençant par `/`.
La commande bash du chat réservée à l'hôte utilisé `! <cmd>` (avec `/bash <cmd>` comme alias).

Il existe deux systèmes liés :

- **Commandes** : messages autonomes `/...`.
- **Directives** : `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
  - Les directives sont retirées du message avant que le modèle ne le voie.
  - Dans les messages de chat normaux (pas uniquement des directives), elles sont traitées comme des « indices en ligne » et ne persistent **pas** les paramètres de session.
  - Dans les messages uniquement de directives (le message ne contient que des directives), elles persistent dans la session et répondent avec un accusé de réception.
  - Les directives ne sont appliquées que pour les **expéditeurs autorisés**. Si `commands.allowFrom` est défini, c'est la seule
    liste d'autorisation utilisée ; sinon l'autorisation provient des listes d'autorisation/appariement de canaux plus `commands.useAccessGroups`.
    Les expéditeurs non autorisés voient les directives traitées comme du texte brut.

Il existe également quelques **raccourcis en ligne** (expéditeurs autorisés/liste d'autorisation uniquement) : `/help`, `/commands`, `/status`, `/whoami` (`/id`).
Ils s'exécutent immédiatement, sont retirés avant que le modèle ne voie le message, et le texte restant continue via le flux normal.

## Configuration

```json5
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

- `commands.text` (défaut `true`) activé l'analyse des `/...` dans les messages de chat.
  - Sur les surfaces sans commandes natives (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), les commandes texte fonctionnent même si vous définissez ceci à `false`.
- `commands.native` (défaut `"auto"`) enregistre les commandes natives.
  - Auto : activé pour Discord/Telegram ; désactivé pour Slack (jusqu'à ce que vous ajoutiez des commandes slash) ; ignoré pour les fournisseurs sans support natif.
  - Définissez `channels.discord.commands.native`, `channels.telegram.commands.native`, ou `channels.slack.commands.native` pour substituer par fournisseur (bool ou `"auto"`).
  - `false` efface les commandes précédemment enregistrées sur Discord/Telegram au démarrage. Les commandes Slack sont gérées dans l'application Slack et ne sont pas supprimées automatiquement.
- `commands.nativeSkills` (défaut `"auto"`) enregistre les commandes de **Skills** nativement quand supporté.
  - Auto : activé pour Discord/Telegram ; désactivé pour Slack (Slack nécessite la création d'une commande slash par Skill).
  - Définissez `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills`, ou `channels.slack.commands.nativeSkills` pour substituer par fournisseur (bool ou `"auto"`).
- `commands.bash` (défaut `false`) activé `! <cmd>` pour exécuter des commandes shell hôte (`/bash <cmd>` est un alias ; nécessite les listes d'autorisation `tools.elevated`).
- `commands.bashForegroundMs` (défaut `2000`) contrôle combien de temps bash attend avant de passer en mode arrière-plan (`0` passe en arrière-plan immédiatement).
- `commands.config` (défaut `false`) activé `/config` (lecture/écriture de `openclaw.json`).
- `commands.debug` (défaut `false`) activé `/debug` (substitutions runtime uniquement).
- `commands.allowFrom` (optionnel) définit une liste d'autorisation par fournisseur pour l'autorisation des commandes. Quand configuré, c'est la
  seule source d'autorisation pour les commandes et directives (les listes d'autorisation/appariement de canaux et `commands.useAccessGroups`
  sont ignorés). Utilisez `"*"` pour un défaut global ; les clés spécifiques au fournisseur le substituent.
- `commands.useAccessGroups` (défaut `true`) applique les listes d'autorisation/politiques pour les commandes quand `commands.allowFrom` n'est pas défini.

## Liste des commandes

Texte + natives (quand activées) :

- `/help`
- `/commands`
- `/skill <name> [input]` (exécuter un Skill par nom)
- `/status` (afficher le statut actuel ; inclut l'utilisation/quota du fournisseur pour le fournisseur de modèle actuel quand disponible)
- `/allowlist` (lister/ajouter/supprimer des entrées de liste d'autorisation)
- `/approve <id> allow-once|allow-always|deny` (résoudre les invites d'approbation exec)
- `/context [list|detail|json]` (expliquer le « contexte » ; `detail` affiche la taille par fichier + par outil + par Skill + prompt système)
- `/export-session [path]` (alias : `/export`) (exporter la session actuelle en HTML avec le prompt système complet)
- `/whoami` (afficher votre id d'expéditeur ; alias : `/id`)
- `/session ttl <duration|off>` (gérer les paramètres au niveau de la session, comme le TTL)
- `/subagents list|kill|log|info|send|steer|spawn` (inspecter, contrôler ou lancer des exécutions de sous-agents pour la session actuelle)
- `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions` (inspecter et contrôler les sessions du runtime ACP)
- `/agents` (lister les agents liés aux threads pour cette session)
- `/focus <target>` (Discord : lier ce thread, ou un nouveau thread, à une cible de session/sous-agent)
- `/unfocus` (Discord : supprimer la liaison du thread actuel)
- `/kill <id|#|all>` (arrêter immédiatement un ou tous les sous-agents en cours pour cette session ; pas de message de confirmation)
- `/steer <id|#> <message>` (diriger un sous-agent en cours immédiatement : en exécution quand possible, sinon abandonner le travail en cours et redémarrer sur le message de direction)
- `/tell <id|#> <message>` (alias pour `/steer`)
- `/config show|get|set|unset` (persister la configuration sur disque, propriétaire uniquement ; nécessite `commands.config: true`)
- `/debug show|set|unset|reset` (substitutions runtime, propriétaire uniquement ; nécessite `commands.debug: true`)
- `/usage off|tokens|full|cost` (pied de page d'utilisation par réponse ou résumé des coûts locaux)
- `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (contrôler le TTS ; voir [/tts](/tts))
  - Discord : la commande native est `/voice` (Discord réserve `/tts`) ; le texte `/tts` fonctionne toujours.
- `/stop`
- `/restart`
- `/dock-telegram` (alias : `/dock_telegram`) (basculer les réponses vers Telegram)
- `/dock-discord` (alias : `/dock_discord`) (basculer les réponses vers Discord)
- `/dock-slack` (alias : `/dock_slack`) (basculer les réponses vers Slack)
- `/activation mention|always` (groupes uniquement)
- `/send on|off|inherit` (propriétaire uniquement)
- `/reset` ou `/new [model]` (indication de modèle optionnelle ; le reste est transmis)
- `/think <off|minimal|low|medium|high|xhigh>` (choix dynamiques par modèle/fournisseur ; alias : `/thinking`, `/t`)
- `/verbose on|full|off` (alias : `/v`)
- `/reasoning on|off|stream` (alias : `/reason` ; quand activé, envoie un message séparé préfixé par `Reasoning:` ; `stream` = brouillon Telegram uniquement)
- `/elevated on|off|ask|full` (alias : `/elev` ; `full` ignoré les approbations exec)
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>` (envoyez `/exec` pour afficher l'état actuel)
- `/model <name>` (alias : `/models` ; ou `/<alias>` depuis `agents.defaults.models.*.alias`)
- `/queue <mode>` (plus des options comme `debounce:2s cap:25 drop:summarize` ; envoyez `/queue` pour voir les paramètres actuels)
- `/bash <command>` (hôte uniquement ; alias pour `! <command>` ; nécessite `commands.bash: true` + listes d'autorisation `tools.elevated`)

Texte uniquement :

- `/compact [instructions]` (voir [/concepts/compaction](/concepts/compaction))
- `! <command>` (hôte uniquement ; une à la fois ; utilisez `!poll` + `!stop` pour les travaux longs)
- `!poll` (vérifier la sortie / le statut ; accepte un `sessionId` optionnel ; `/bash poll` fonctionne aussi)
- `!stop` (arrêter le travail bash en cours ; accepte un `sessionId` optionnel ; `/bash stop` fonctionne aussi)

Notes :

- Les commandes acceptent un `:` optionnel entre la commande et les arguments (par ex. `/think: high`, `/send: on`, `/help:`).
- `/new <model>` accepte un alias de modèle, `provider/model`, ou un nom de fournisseur (correspondance floue) ; s'il n'y a pas de correspondance, le texte est traité comme le corps du message.
- Pour une ventilation complète de l'utilisation du fournisseur, utilisez `openclaw status --usage`.
- `/allowlist add|remove` nécessite `commands.config=true` et respecte `configWrites` du canal.
- `/usage` contrôle le pied de page d'utilisation par réponse ; `/usage cost` affiche un résumé des coûts locaux depuis les journaux de session OpenClaw.
- `/restart` est activé par défaut ; définissez `commands.restart: false` pour le désactiver.
- Commande native Discord uniquement : `/vc join|leave|status` contrôle les canaux vocaux (nécessite `channels.discord.voice` et les commandes natives ; non disponible en texte).
- Les commandes de liaison de threads Discord (`/focus`, `/unfocus`, `/agents`, `/session ttl`) nécessitent que les liaisons de threads effectives soient activées (`session.threadBindings.enabled` et/ou `channels.discord.threadBindings.enabled`).
- Référence de commande ACP et comportement du runtime : [Agents ACP](/tools/acp-agents).
- `/verbose` est prévu pour le débogage et la visibilité supplémentaire ; gardez-le **désactivé** en utilisation normale.
- Les résumés d'échec d'outils sont toujours affichés quand pertinent, mais le texte détaillé d'échec n'est inclus que quand `/verbose` est `on` ou `full`.
- `/reasoning` (et `/verbose`) sont risqués dans les contextes de groupe : ils peuvent révéler du raisonnement interne ou des sorties d'outils que vous n'aviez pas l'intention d'exposer. Préférez les laisser désactivés, surtout dans les chats de groupe.
- **Chemin rapide :** les messages de commande uniquement des expéditeurs autorisés sont traités immédiatement (contournent la file + le modèle).
- **Déclenchement par mention de groupe :** les messages de commande uniquement des expéditeurs autorisés contournent les exigences de mention.
- **Raccourcis en ligne (expéditeurs autorisés uniquement) :** certaines commandes fonctionnent aussi quand intégrées dans un message normal et sont retirées avant que le modèle ne voie le texte restant.
  - Exemple : `hey /status` déclenche une réponse de statut, et le texte restant continue via le flux normal.
- Actuellement : `/help`, `/commands`, `/status`, `/whoami` (`/id`).
- Les messages de commande uniquement non autorisés sont silencieusement ignorés, et les tokens `/...` en ligne sont traités comme du texte brut.
- **Commandes Skills :** les Skills `user-invocable` sont exposés comme commandes slash. Les noms sont nettoyés en `a-z0-9_` (max 32 caractères) ; les collisions reçoivent des suffixes numériques (par ex. `_2`).
  - `/skill <name> [input]` exécute un Skill par nom (utile quand les limités de commandes natives empêchent les commandes par Skill).
  - Par défaut, les commandes Skills sont transmises au modèle comme une requête normale.
  - Les Skills peuvent optionnellement déclarer `command-dispatch: tool` pour router la commande directement vers un outil (déterministe, pas de modèle).
  - Exemple : `/prose` (plugin OpenProse) — voir [OpenProse](/prose).
- **Arguments de commandes natives :** Discord utilisé l'autocomplétion pour les options dynamiques (et des menus de boutons quand vous omettez les arguments obligatoires). Telegram et Slack affichent un menu de boutons quand une commande supporte des choix et que vous omettez l'argument.

## Surfaces d'utilisation (ce qui s'affiche où)

- **Utilisation/quota du fournisseur** (exemple : « Claude 80% restant ») s'affiche dans `/status` pour le fournisseur de modèle actuel quand le suivi d'utilisation est activé.
- **Tokens/coût par réponse** est contrôlé par `/usage off|tokens|full` (ajouté aux réponses normales).
- `/model status` concerne les **modèles/auth/endpoints**, pas l'utilisation.

## Sélection de modèle (`/model`)

`/model` est implémenté comme une directive.

Exemples :

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notes :

- `/model` et `/model list` affichent un sélecteur compact et numéroté (famille de modèles + fournisseurs disponibles).
- Sur Discord, `/model` et `/models` ouvrent un sélecteur interactif avec des menus déroulants de fournisseur et modèle plus une étape de soumission.
- `/model <#>` sélectionne depuis ce sélecteur (et préfère le fournisseur actuel quand possible).
- `/model status` affiche la vue détaillée, y compris l'endpoint du fournisseur configuré (`baseUrl`) et le mode API (`api`) quand disponible.

## Substitutions de débogage

`/debug` vous permet de définir des substitutions de configuration **runtime uniquement** (mémoire, pas disque). Propriétaire uniquement. Désactivé par défaut ; activez avec `commands.debug: true`.

Exemples :

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Notes :

- Les substitutions s'appliquent immédiatement aux nouvelles lectures de configuration, mais n'écrivent **pas** dans `openclaw.json`.
- Utilisez `/debug reset` pour effacer toutes les substitutions et revenir à la configuration sur disque.

## Mises à jour de configuration

`/config` écrit dans votre configuration sur disque (`openclaw.json`). Propriétaire uniquement. Désactivé par défaut ; activez avec `commands.config: true`.

Exemples :

```
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Notes :

- La configuration est validée avant l'écriture ; les changements invalides sont rejetés.
- Les mises à jour `/config` persistent entre les redémarrages.

## Notes par surface

- **Commandes texte** s'exécutent dans la session de chat normale (les DM partagent `main`, les groupes ont leur propre session).
- **Commandes natives** utilisent des sessions isolées :
  - Discord : `agent:<agentId>:discord:slash:<userId>`
  - Slack : `agent:<agentId>:slack:slash:<userId>` (préfixe configurable via `channels.slack.slashCommand.sessionPrefix`)
  - Telegram : `telegram:slash:<userId>` (cible la session de chat via `CommandTargetSessionKey`)
- **`/stop`** cible la session de chat activé pour pouvoir interrompre l'exécution en cours.
- **Slack :** `channels.slack.slashCommand` est toujours supporté pour une seule commande style `/openclaw`. Si vous activez `commands.native`, vous devez créer une commande slash Slack par commande intégrée (mêmes noms que `/help`). Les menus d'arguments de commandes pour Slack sont délivrés comme des boutons Block Kit éphémères.
