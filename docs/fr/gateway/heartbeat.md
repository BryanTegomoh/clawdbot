---
summary: "Messages de sondage heartbeat et règles de notification"
read_when:
  - Ajustement de la cadence ou de la messagerie heartbeat
  - Décision entre heartbeat et cron pour les tâches planifiées
title: "Heartbeat"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/heartbeat.md
  workflow: manual
---

# Heartbeat (Gateway)

> **Heartbeat vs Cron ?** Voir [Cron vs Heartbeat](/automation/cron-vs-heartbeat) pour des conseils sur quand utiliser chacun.

Heartbeat exécute des **tours d'agent périodiques** dans la session principale afin que le modèle puisse remonter tout ce qui nécessite de l'attention sans vous spammer.

Dépannage : [/automation/troubleshooting](/automation/troubleshooting)

## Démarrage rapide (débutant)

1. Laissez les heartbeats activés (par défaut `30m`, ou `1h` pour Anthropic OAuth/setup-token) ou définissez votre propre cadence.
2. Créez une petite checklist `HEARTBEAT.md` dans le workspace agent (optionnel mais recommandé).
3. Décidez où les messages heartbeat doivent aller (`target: "none"` est la valeur par défaut ; définissez `target: "last"` pour router vers le dernier contact).
4. Optionnel : activez la livraison du raisonnement heartbeat pour la transparence.
5. Optionnel : restreignez les heartbeats aux heures activés (heure locale).

Exemple de configuration :

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // livraison explicite au dernier contact (par défaut "none")
        directPolicy: "allow", // par défaut : autoriser les cibles direct/DM ; mettre "block" pour supprimer
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // optionnel : envoyer aussi un message séparé `Reasoning:`
      },
    },
  },
}
```

## Valeurs par défaut

- Intervalle : `30m` (ou `1h` lorsque Anthropic OAuth/setup-token est le mode d'authentification détecté). Définissez `agents.defaults.heartbeat.every` ou par agent `agents.list[].heartbeat.every` ; utilisez `0m` pour désactiver.
- Corps du prompt (configurable via `agents.defaults.heartbeat.prompt`) :
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- Le prompt heartbeat est envoyé **tel quel** comme message utilisateur. Le prompt système inclut une section « Heartbeat » et l'exécution est marquée en interne.
- Les heures activés (`heartbeat.activeHours`) sont vérifiées dans le fuseau horaire configuré.
  En dehors de la fenêtre, les heartbeats sont ignorés jusqu'au prochain tick dans la fenêtre.

## À quoi sert le prompt heartbeat

Le prompt par défaut est intentionnellement large :

- **Tâches en arrière-plan** : « Consider outstanding tasks » pousse l'agent à examiner les suivis (boîte de réception, calendrier, rappels, travail en file d'attente) et à remonter tout ce qui est urgent.
- **Vérification humaine** : « Checkup sometimes on your human during day time » pousse un message léger occasionnel « anything you need? », mais évite le spam nocturne en utilisant votre fuseau horaire local configuré (voir [/concepts/timezone](/concepts/timezone)).

Si vous voulez qu'un heartbeat fasse quelque chose de très spécifique (par exemple « vérifier les stats Gmail PubSub » ou « vérifier la santé du gateway »), définissez `agents.defaults.heartbeat.prompt` (ou `agents.list[].heartbeat.prompt`) avec un corps personnalisé (envoyé tel quel).

## Contrat de réponse

- Si rien ne nécessite d'attention, répondez avec **`HEARTBEAT_OK`**.
- Pendant les exécutions heartbeat, OpenClaw traite `HEARTBEAT_OK` comme un acquittement lorsqu'il apparaît au **début ou à la fin** de la réponse. Le jeton est supprimé et la réponse est abandonnée si le contenu restant est **≤ `ackMaxChars`** (par défaut : 300).
- Si `HEARTBEAT_OK` apparaît au **milieu** d'une réponse, il n'est pas traité spécialement.
- Pour les alertes, **n'incluez pas** `HEARTBEAT_OK` ; renvoyez uniquement le texte de l'alerte.

En dehors des heartbeats, un `HEARTBEAT_OK` isolé au début/fin d'un message est supprimé et journalise ; un message qui est uniquement `HEARTBEAT_OK` est abandonné.

## Configuration

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // par défaut : 30m (0m désactive)
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // par défaut : false (livrer un message séparé Reasoning: lorsque disponible)
        target: "last", // par défaut : none | options : last | none | <channel id> (core ou plugin, par ex. "bluebubbles")
        to: "+15551234567", // remplacement optionnel spécifique au canal
        accountId: "ops-bot", // identifiant de canal multi-compte optionnel
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // max chars autorisés après HEARTBEAT_OK
      },
    },
  },
}
```

### Portée et précédence

- `agents.defaults.heartbeat` définit le comportement heartbeat global.
- `agents.list[].heartbeat` se fusionne par-dessus ; si un agent a un bloc `heartbeat`, **seuls ces agents** exécutent les heartbeats.
- `channels.defaults.heartbeat` définit les valeurs par défaut de visibilité pour tous les canaux.
- `channels.<channel>.heartbeat` remplace les valeurs par défaut du canal.
- `channels.<channel>.accounts.<id>.heartbeat` (canaux multi-comptes) remplace les paramètres par canal.

### Heartbeats par agent

Si une entrée `agents.list[]` inclut un bloc `heartbeat`, **seuls ces agents** exécutent les heartbeats. Le bloc par agent se fusionne par-dessus `agents.defaults.heartbeat` (vous pouvez donc définir les valeurs par défaut partagées une fois et remplacer par agent).

Exemple : deux agents, seul le second agent exécute les heartbeats.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // livraison explicite au dernier contact (par défaut "none")
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### Exemple d'heures activés

Restreindre les heartbeats aux heures de bureau dans un fuseau horaire spécifique :

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // livraison explicite au dernier contact (par défaut "none")
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // optionnel ; utilise votre userTimezone si défini, sinon le fuseau de l'hôte
        },
      },
    },
  },
}
```

En dehors de cette fenêtre (avant 9h ou après 22h Eastern), les heartbeats sont ignorés. Le prochain tick planifié dans la fenêtre s'exécutera normalement.

### Configuration 24/7

Si vous voulez que les heartbeats s'exécutent toute la journée, utilisez l'un de ces patterns :

- Omettez `activeHours` entièrement (pas de restriction de fenêtre temporelle ; c'est le comportement par défaut).
- Définissez une fenêtre complète : `activeHours: { start: "00:00", end: "24:00" }`.

Ne définissez pas les mêmes heures `start` et `end` (par exemple `08:00` à `08:00`).
C'est traité comme une fenêtre de largeur zéro, donc les heartbeats sont toujours ignorés.

### Exemple multi-comptes

Utilisez `accountId` pour cibler un compte spécifique sur les canaux multi-comptes comme Telegram :

```json5
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // optionnel : router vers un sujet/fil spécifique
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### Notes sur les champs

- `every` : intervalle heartbeat (chaîne de durée ; unité par défaut = minutes).
- `model` : remplacement de modèle optionnel pour les exécutions heartbeat (`provider/model`).
- `includeReasoning` : lorsque activé, livre aussi le message séparé `Reasoning:` lorsque disponible (même forme que `/reasoning on`).
- `session` : clé de session optionnelle pour les exécutions heartbeat.
  - `main` (par défaut) : session principale de l'agent.
  - Clé de session explicite (copiez depuis `openclaw sessions --json` ou le [CLI sessions](/cli/sessions)).
  - Formats de clé de session : voir [Sessions](/concepts/session) et [Groupes](/channels/groups).
- `target` :
  - `last` : livrer au dernier canal externe utilise.
  - canal explicite : `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  - `none` (par défaut) : exécuter le heartbeat mais **ne pas livrer** en externe.
- `directPolicy` : contrôle le comportement de livraison direct/DM :
  - `allow` (par défaut) : autoriser la livraison heartbeat direct/DM.
  - `block` : supprimer la livraison direct/DM (`reason=dm-blocked`).
- `to` : remplacement de destinataire optionnel (identifiant spécifique au canal, par ex. E.164 pour WhatsApp ou un identifiant de chat Telegram). Pour les sujets/fils Telegram, utilisez `<chatId>:topic:<messageThreadId>`.
- `accountId` : identifiant de compte optionnel pour les canaux multi-comptes. Lorsque `target: "last"`, l'identifiant de compte s'applique au dernier canal résolu s'il prend en charge les comptes ; sinon il est ignoré. Si l'identifiant de compte ne correspond pas à un compte configuré pour le canal résout, la livraison est ignorée.
- `prompt` : remplace le corps du prompt par défaut (non fusionne).
- `ackMaxChars` : max chars autorisés après `HEARTBEAT_OK` avant livraison.
- `suppressToolErrorWarnings` : lorsque true, supprimé les charges utiles d'avertissement d'erreur d'outil pendant les exécutions heartbeat.
- `activeHours` : restreint les exécutions heartbeat à une fenêtre temporelle. Objet avec `start` (HH:MM, inclusif ; utilisez `00:00` pour le début de journée), `end` (HH:MM exclusif ; `24:00` autorisé pour la fin de journée), et optionnellement `timezone`.
  - Omis ou `"user"` : utilisé votre `agents.defaults.userTimezone` si défini, sinon le fuseau horaire système de l'hôte.
  - `"local"` : utilisé toujours le fuseau horaire système de l'hôte.
  - Tout identifiant IANA (par ex. `America/New_York`) : utilisé directement ; si invalide, revient au comportement `"user"` ci-dessus.
  - `start` et `end` ne doivent pas être égaux pour une fenêtre activé ; des valeurs égales sont traitées comme largeur zéro (toujours en dehors de la fenêtre).
  - En dehors de la fenêtre activé, les heartbeats sont ignorés jusqu'au prochain tick dans la fenêtre.

## Comportement de livraison

- Les heartbeats s'exécutent dans la session principale de l'agent par défaut (`agent:<id>:<mainKey>`),
  ou `global` lorsque `session.scope = "global"`. Définissez `session` pour remplacer vers une session de canal spécifique (Discord/WhatsApp/etc.).
- `session` n'affecte que le contexte d'exécution ; la livraison est contrôlée par `target` et `to`.
- Pour livrer à un canal/destinataire spécifique, définissez `target` + `to`. Avec
  `target: "last"`, la livraison utilise le dernier canal externe pour cette session.
- Les livraisons heartbeat autorisent les cibles direct/DM par défaut. Définissez `directPolicy: "block"` pour supprimer les envois vers des cibles directes tout en exécutant le tour heartbeat.
- Si la file d'attente principale est occupée, le heartbeat est ignoré et réessaie plus tard.
- Si `target` ne résout aucune destination externe, l'exécution a toujours lieu mais aucun message sortant n'est envoyé.
- Les réponses heartbeat uniquement **ne maintiennent pas** la session active ; le dernier `updatedAt` est restauré pour que l'expiration par inactivité se comporte normalement.

## Contrôles de visibilité

Par défaut, les acquittements `HEARTBEAT_OK` sont supprimés tandis que le contenu d'alerte est livré. Vous pouvez ajuster cela par canal ou par compte :

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false # Masquer HEARTBEAT_OK (par défaut)
      showAlerts: true # Afficher les messages d'alerte (par défaut)
      useIndicator: true # Émettre des événements indicateur (par défaut)
  telegram:
    heartbeat:
      showOk: true # Afficher les acquittements OK sur Telegram
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Supprimer la livraison d'alerte pour ce compte
```

Précédence : par-compte → par-canal → valeurs par défaut du canal → valeurs par défaut intégrées.

### Ce que fait chaque flag

- `showOk` : envoie un acquittement `HEARTBEAT_OK` lorsque le modèle renvoie une réponse OK uniquement.
- `showAlerts` : envoie le contenu d'alerte lorsque le modèle renvoie une réponse non-OK.
- `useIndicator` : émet des événements indicateur pour les surfaces de statut UI.

Si **les trois** sont false, OpenClaw ignore entièrement l'exécution heartbeat (pas d'appel modèle).

### Exemples par-canal vs par-compte

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # tous les comptes Slack
    accounts:
      ops:
        heartbeat:
          showAlerts: false # supprimer les alertes pour le compte ops uniquement
  telegram:
    heartbeat:
      showOk: true
```

### Patterns courants

| Objectif                                          | Configuration                                                                                    |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Comportement par défaut (OKs silencieux, alertes activées) | _(aucune configuration nécessaire)_                                                              |
| Totalement silencieux (pas de messages, pas d'indicateur) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Indicateur uniquement (pas de messages)           | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }`  |
| OKs dans un seul canal                            | `channels.telegram.heartbeat: { showOk: true }`                                          |

## HEARTBEAT.md (optionnel)

Si un fichier `HEARTBEAT.md` existe dans le workspace, le prompt par défaut dit à l'agent de le lire. Considérez-le comme votre « checklist heartbeat » : petit, stable et sûr à inclure toutes les 30 minutes.

Si `HEARTBEAT.md` existe mais est effectivement vide (uniquement des lignes vides et des en-têtes markdown comme `# Heading`), OpenClaw ignore l'exécution heartbeat pour économiser les appels API.
Si le fichier est absent, le heartbeat s'exécute toujours et le modèle décide quoi faire.

Gardez-le petit (courte checklist ou rappels) pour éviter le gonflement du prompt.

Exemple `HEARTBEAT.md` :

```md
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it's daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### L'agent peut-il mettre à jour HEARTBEAT.md ?

Oui, si vous le lui demandez.

`HEARTBEAT.md` est juste un fichier normal dans le workspace agent, donc vous pouvez dire à l'agent (dans un chat normal) quelque chose comme :

- « Update `HEARTBEAT.md` to add a daily calendar check. »
- « Rewrite `HEARTBEAT.md` so it's shorter and focused on inbox follow-ups. »

Si vous voulez que cela se fasse de manière proactive, vous pouvez aussi inclure une ligne explicite dans votre prompt heartbeat comme : « If the checklist becomes stale, update HEARTBEAT.md with a better one. »

Note de sécurité : ne mettez pas de secrets (clés API, numéros de téléphone, jetons privés) dans `HEARTBEAT.md` : il fait partie du contexte du prompt.

## Réveil manuel (à la demande)

Vous pouvez mettre en file d'attente un événement système et déclencher un heartbeat immédiat avec :

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

Si plusieurs agents ont `heartbeat` configuré, un réveil manuel exécute chacun de ces heartbeats d'agents immédiatement.

Utilisez `--mode next-heartbeat` pour attendre le prochain tick planifié.

## Livraison du raisonnement (optionnel)

Par défaut, les heartbeats ne livrent que la charge utile « réponse » finale.

Si vous voulez de la transparence, activez :

- `agents.defaults.heartbeat.includeReasoning: true`

Lorsque activé, les heartbeats livreront aussi un message séparé préfixé `Reasoning:` (même forme que `/reasoning on`). Cela peut être utile lorsque l'agent gère plusieurs sessions/codex et que vous voulez voir pourquoi il a décidé de vous contacter, mais cela peut aussi exposer plus de détails internes que souhaité. Préférez le garder désactivé dans les discussions de groupe.

## Conscience des coûts

Les heartbeats exécutent des tours d'agent complets. Des intervalles plus courts consomment plus de jetons. Gardez `HEARTBEAT.md` petit et envisagez un `model` moins cher ou `target: "none"` si vous voulez uniquement des mises à jour d'état internes.
