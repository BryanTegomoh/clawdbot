---
summary: "Interface terminal (TUI) : se connecter au Gateway depuis n'importe quelle machine"
read_when:
  - Vous souhaitez un guide pas à pas pour débutants de la TUI
  - Vous avez besoin de la liste complète des fonctionnalités, commandes et raccourcis de la TUI
title: "TUI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/web/tui.md
  workflow: manual
---

# TUI (Interface terminal)

## Démarrage rapide

1. Démarrez le Gateway.

```bash
openclaw gateway
```

2. Ouvrez la TUI.

```bash
openclaw tui
```

3. Tapez un message et appuyez sur Entrée.

Gateway distant :

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Utilisez `--password` si votre Gateway utilisé l'authentification par mot de passe.

## Ce que vous voyez

- En-tête : URL de connexion, agent actuel, session actuelle.
- Journal de chat : messages utilisateur, réponses de l'assistant, notices système, cartes d'outils.
- Ligne de statut : état de connexion/exécution (connexion, en cours, diffusion, inactif, erreur).
- Pied de page : état de connexion + agent + session + modèle + pensée/verbeux/raisonnement + compteurs de jetons + livraison.
- Saisie : éditeur de texte avec autocomplétion.

## Modèle mental : agents + sessions

- Les agents sont des identifiants uniques (ex. `main`, `research`). Le Gateway expose la liste.
- Les sessions appartiennent à l'agent actuel.
- Les clés de session sont stockées sous la forme `agent:<agentId>:<sessionKey>`.
  - Si vous tapez `/session main`, la TUI l'étend en `agent:<currentAgent>:main`.
  - Si vous tapez `/session agent:other:main`, vous basculez explicitement vers cette session d'agent.
- Portée de la session :
  - `per-sender` (par défaut) : chaque agent a plusieurs sessions.
  - `global` : la TUI utilisé toujours la session `global` (le sélecteur peut être vide).
- L'agent actuel et la session sont toujours visibles dans le pied de page.

## Envoi + livraison

- Les messages sont envoyés au Gateway ; la livraison aux fournisseurs est désactivée par défaut.
- Activer la livraison :
  - `/deliver on`
  - ou le panneau Paramètres
  - ou démarrer avec `openclaw tui --deliver`

## Sélecteurs + superpositions

- Sélecteur de modèle : lister les modèles disponibles et définir la surcharge de session.
- Sélecteur d'agent : choisir un agent différent.
- Sélecteur de session : affiche uniquement les sessions de l'agent actuel.
- Paramètres : basculer livraison, expansion de la sortie d'outils et visibilité de la pensée.

## Raccourcis clavier

- Entrée : envoyer le message
- Échap : annuler l'exécution active
- Ctrl+C : effacer la saisie (appuyer deux fois pour quitter)
- Ctrl+D : quitter
- Ctrl+L : sélecteur de modèle
- Ctrl+G : sélecteur d'agent
- Ctrl+P : sélecteur de session
- Ctrl+O : basculer l'expansion de la sortie d'outils
- Ctrl+T : basculer la visibilité de la pensée (recharge l'historique)

## Commandes slash

Principales :

- `/help`
- `/status`
- `/agent <id>` (ou `/agents`)
- `/session <key>` (ou `/sessions`)
- `/model <provider/model>` (ou `/models`)

Contrôles de session :

- `/think <off|minimal|low|medium|high>`
- `/verbose <on|full|off>`
- `/reasoning <on|off|stream>`
- `/usage <off|tokens|full>`
- `/elevated <on|off|ask|full>` (alias : `/elev`)
- `/activation <mention|always>`
- `/deliver <on|off>`

Cycle de vie de la session :

- `/new` ou `/reset` (réinitialiser la session)
- `/abort` (annuler l'exécution active)
- `/settings`
- `/exit`

Les autres commandes slash du Gateway (par exemple, `/context`) sont transmises au Gateway et affichées comme sortie système. Voir [Commandes slash](/tools/slash-commands).

## Commandes shell locales

- Préfixez une ligne avec `!` pour exécuter une commande shell locale sur l'hôte de la TUI.
- La TUI demande une fois par session d'autoriser l'exécution locale ; refuser désactive `!` pour la session.
- Les commandes s'exécutent dans un shell neuf, non interactif, dans le répertoire de travail de la TUI (pas de `cd`/env persistant).
- Un `!` seul est envoyé comme message normal ; les espaces en début de ligne ne déclenchent pas l'exécution locale.

## Sortie d'outils

- Les appels d'outils s'affichent sous forme de cartes avec arguments + résultats.
- Ctrl+O bascule entre les vues repliées/dépliées.
- Pendant l'exécution des outils, les mises à jour partielles sont diffusées dans la même carte.

## Historique + diffusion

- À la connexion, la TUI charge le dernier historique (200 messages par défaut).
- Les réponses en diffusion se mettent à jour sur place jusqu'à finalisation.
- La TUI écoute également les événements d'outils de l'agent pour des cartes d'outils plus riches.

## Détails de connexion

- La TUI s'enregistre auprès du Gateway avec `mode: "tui"`.
- Les reconnexions affichent un message système ; les lacunes d'événements sont signalées dans le journal.

## Options

- `--url <url>` : URL WebSocket du Gateway (par défaut selon la configuration ou `ws://127.0.0.1:<port>`)
- `--token <token>` : jeton du Gateway (si requis)
- `--password <password>` : mot de passe du Gateway (si requis)
- `--session <key>` : clé de session (par défaut : `main`, ou `global` lorsque la portée est globale)
- `--deliver` : livrer les réponses de l'assistant au fournisseur (désactivé par défaut)
- `--thinking <level>` : surcharger le niveau de pensée pour les envois
- `--timeout-ms <ms>` : délai d'expiration de l'agent en ms (par défaut selon `agents.defaults.timeoutSeconds`)

Note : lorsque vous définissez `--url`, la TUI ne se rabat pas sur les identifiants de configuration ou d'environnement.
Passez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

## Dépannage

Pas de sortie après l'envoi d'un message :

- Exécutez `/status` dans la TUI pour confirmer que le Gateway est connecté et inactif/occupé.
- Vérifiez les journaux du Gateway : `openclaw logs --follow`.
- Confirmez que l'agent peut s'exécuter : `openclaw status` et `openclaw models status`.
- Si vous attendez des messages dans un canal de chat, activez la livraison (`/deliver on` ou `--deliver`).
- `--history-limit <n>` : entrées d'historique à charger (par défaut 200)

## Dépannage de connexion

- `disconnected` : assurez-vous que le Gateway est en cours d'exécution et que vos `--url/--token/--password` sont corrects.
- Pas d'agents dans le sélecteur : vérifiez `openclaw agents list` et votre configuration de routage.
- Sélecteur de session vide : vous êtes peut-être en portée globale ou n'avez pas encore de sessions.
