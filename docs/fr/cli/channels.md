---
summary: "Référence CLI pour `openclaw channels` (comptes, statut, connexion/déconnexion, logs)"
read_when:
  - Vous souhaitez ajouter/supprimer des comptes de canaux (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage)
  - Vous souhaitez vérifier le statut des canaux ou suivre les logs des canaux
title: "channels"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/channels.md
  workflow: manual
---

# `openclaw channels`

Gérer les comptes de canaux de chat et leur statut d'exécution sur le Gateway.

Documentation liée :

- Guides des canaux : [Channels](/channels/index)
- Configuration du Gateway : [Configuration](/gateway/configuration)

## Commandes courantes

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## Ajouter / supprimer des comptes

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Astuce : `openclaw channels add --help` affiche les options par canal (jeton, jeton d'application, chemins signal-cli, etc).

Quand vous exécutez `openclaw channels add` sans drapeaux, l'assistant interactif peut demander :

- les ids de compte par canal sélectionné
- les noms d'affichage optionnels pour ces comptes
- `Bind configured channel accounts to agents now?`

Si vous confirmez le binding maintenant, l'assistant demande quel agent doit posséder chaque compte de canal configuré et écrit les liaisons de routage scopées au compte.

Vous pouvez aussi gérer les mêmes règles de routage plus tard avec `openclaw agents bindings`, `openclaw agents bind` et `openclaw agents unbind` (voir [agents](/cli/agents)).

Quand vous ajoutez un compte non par défaut à un canal qui utilise encore des paramètres de niveau supérieur mono-compte (pas encore d'entrées `channels.<channel>.accounts`), OpenClaw déplace les valeurs mono-compte scopées au compte dans `channels.<channel>.accounts.default`, puis écrit le nouveau compte. Cela préserve le comportement du compte original tout en passant à la forme multi-comptes.

Le comportement de routage reste cohérent :

- Les liaisons canal uniquement existantes (sans `accountId`) continuent de correspondre au compte par défaut.
- `channels add` ne crée pas automatiquement et ne réécrit pas les liaisons en mode non interactif.
- La configuration interactive peut optionnellement ajouter des liaisons scopées au compte.

Si votre configuration était déjà dans un état mixte (comptes nommés présents, `default` manquant, et valeurs mono-compte de niveau supérieur toujours définies), exécutez `openclaw doctor --fix` pour déplacer les valeurs scopées au compte dans `accounts.default`.

## Connexion / déconnexion (interactif)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## Dépannage

- Exécutez `openclaw status --deep` pour un sondage large.
- Utilisez `openclaw doctor` pour des corrections guidées.
- `openclaw channels list` affiche `Claude: HTTP 403 ... user:profile` -> l'instantané d'utilisation nécessite la portée `user:profile`. Utilisez `--no-usage`, ou fournissez une clé de session claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), ou réauthentifiez-vous via le CLI Claude Code.

## Sonde de capacités

Récupérer les indices de capacité du fournisseur (intentions/portées lorsque disponibles) ainsi que le support de fonctionnalités statiques :

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notes :

- `--channel` est optionnel ; omettez-le pour lister tous les canaux (extensions incluses).
- `--target` accepte `channel:<id>` ou un identifiant de canal numérique brut et ne s'applique qu'a Discord.
- Les sondes sont spécifiques au fournisseur : intentions Discord + permissions de canal optionnelles ; portées bot + utilisateur Slack ; drapeaux bot Telegram + webhook ; version daemon Signal ; jeton d'application MS Teams + rôles/portées Graph (annotes lorsque connus). Les canaux sans sondes signalent `Probe: unavailable`.

## Résoudre des noms en identifiants

Résoudre les noms de canaux/utilisateurs en identifiants en utilisant le répertoire du fournisseur :

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Notes :

- Utilisez `--kind user|group|auto` pour forcer le type de cible.
- La résolution préfère les correspondances activés lorsque plusieurs entrées partagent le même nom.
