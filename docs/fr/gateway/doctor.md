---
summary: "Commande Doctor : vérifications de santé, migrations de configuration et étapes de réparation"
read_when:
  - Ajout ou modification des migrations doctor
  - Introduction de changements de configuration incompatibles
title: "Doctor"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/doctor.md
  workflow: manual
---

# Doctor

`openclaw doctor` est l'outil de réparation et de migration pour OpenClaw. Il corrige les configurations/états obsolètes, vérifie la santé et fournit des étapes de réparation exploitables.

## Démarrage rapide

```bash
openclaw doctor
```

### Mode headless / automatisation

```bash
openclaw doctor --yes
```

Accepte les valeurs par défaut sans invitation (y compris les étapes de réparation de redémarrage/service/bac à sable le cas échéant).

```bash
openclaw doctor --repair
```

Applique les réparations recommandées sans invitation (réparations + redémarrages lorsque c'est sûr).

```bash
openclaw doctor --repair --force
```

Applique également les réparations agressives (écrase les configurations superviseur personnalisées).

```bash
openclaw doctor --non-interactive
```

Exécute sans invitations et n'applique que les migrations sûres (normalisation de configuration + déplacements d'état sur disque). Ignoré les actions de redémarrage/service/bac à sable nécessitant une confirmation humaine.
Les migrations d'état legacy s'exécutent automatiquement lorsqu'elles sont détectées.

```bash
openclaw doctor --deep
```

Analyse les services système à la recherche d'installations gateway supplémentaires (launchd/systemd/schtasks).

Si vous souhaitez examiner les changements avant l'écriture, ouvrez d'abord le fichier de configuration :

```bash
cat ~/.openclaw/openclaw.json
```

## Ce qu'il fait (résumé)

- Mise à jour pré-vol optionnelle pour les installations git (interactif uniquement).
- Vérification de fraîcheur du protocole UI (reconstruit l'Interface de contrôle lorsque le schéma de protocole est plus récent).
- Vérification de santé + invite de redémarrage.
- Résumé du statut des skills (éligibles/manquants/bloqués).
- Normalisation de configuration pour les valeurs legacy.
- Avertissements de remplacement de fournisseur OpenCode Zen (`models.providers.opencode`).
- Migration d'état legacy sur disque (sessions/répertoire agent/authentification WhatsApp).
- Vérifications d'intégrité d'état et de permissions (sessions, transcriptions, répertoire d'état).
- Vérifications des permissions du fichier de configuration (chmod 600) en mode local.
- Santé de l'authentification des modèles : vérifie l'expiration OAuth, peut rafraîchir les jetons expirants et signale les états cooldown/désactivé des profils d'authentification.
- Détection de répertoire workspace supplémentaire (`~/openclaw`).
- Réparation d'image bac à sable lorsque l'isolation en bac à sable est activée.
- Migration de service legacy et détection de gateway supplémentaire.
- Vérifications du runtime gateway (service installé mais non en cours d'exécution ; label launchd en cache).
- Avertissements de statut de canal (sondés depuis le gateway en cours d'exécution).
- Audit de configuration superviseur (launchd/systemd/schtasks) avec réparation optionnelle.
- Vérifications des bonnes pratiques du runtime gateway (Node vs Bun, chemins de gestionnaire de versions).
- Diagnostics de collision de port gateway (par défaut `18789`).
- Avertissements de sécurité pour les politiques DM ouvertes.
- Avertissements d'authentification gateway lorsque aucun `gateway.auth.token` n'est défini (mode local ; propose la génération de jeton).
- Vérification linger systemd sous Linux.
- Vérifications d'installation source (incompatibilité workspace pnpm, assets UI manquants, binaire tsx manquant).
- Écrit la configuration mise à jour + métadonnées de l'assistant.

## Comportement détaillé et justification

### 0) Mise à jour optionnelle (installations git)

Si c'est un checkout git et que doctor s'exécute de manière interactive, il propose de mettre à jour (fetch/rebase/build) avant d'exécuter doctor.

### 1) Normalisation de configuration

Si la configuration contient des formes de valeurs legacy (par exemple `messages.ackReaction` sans remplacement spécifique au canal), doctor les normalise selon le schéma actuel.

### 2) Migrations de clés de configuration legacy

Lorsque la configuration contient des clés obsolètes, les autres commandes refusent de s'exécuter et vous demandent d'exécuter `openclaw doctor`.

Doctor va :

- Expliquer quelles clés legacy ont été trouvées.
- Montrer la migration qu'il a appliquée.
- Réécrire `~/.openclaw/openclaw.json` avec le schéma mis à jour.

Le Gateway exécute aussi automatiquement les migrations doctor au démarrage lorsqu'il détecte un format de configuration legacy, de sorte que les configurations obsolètes sont réparées sans intervention manuelle.

Migrations actuelles :

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → `bindings` de niveau supérieur
- `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.média.audio.models`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- Pour les canaux avec des `accounts` nommes mais sans `accounts.default`, déplacer les valeurs de canal mono-compte de niveau supérieur dans `channels.<channel>.accounts.default` lorsqu'elles sont présentés
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
- `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

### 2b) Remplacements de fournisseur OpenCode Zen

Si vous avez ajouté `models.providers.opencode` (ou `opencode-zen`) manuellement, cela remplace le catalogue OpenCode Zen intégré de `@mariozechner/pi-ai`. Cela peut forcer chaque modèle sur une seule API ou mettre les coûts à zéro. Doctor avertit pour que vous puissiez supprimer le remplacement et restaurer le routage API par modèle + les coûts.

### 3) Migrations d'état legacy (disposition sur disque)

Doctor peut migrer les dispositions sur disque plus anciennes vers la structure actuelle :

- Stockage de sessions + transcriptions :
  - de `~/.openclaw/sessions/` vers `~/.openclaw/agents/<agentId>/sessions/`
- Répertoire agent :
  - de `~/.openclaw/agent/` vers `~/.openclaw/agents/<agentId>/agent/`
- État d'authentification WhatsApp (Baileys) :
  - depuis le legacy `~/.openclaw/credentials/*.json` (sauf `oauth.json`)
  - vers `~/.openclaw/credentials/whatsapp/<accountId>/...` (identifiant de compte par défaut : `default`)

Ces migrations sont au mieux et idempotentes ; doctor émettra des avertissements lorsqu'il laisse des dossiers legacy en tant que sauvegardes. Le Gateway/CLI migre aussi automatiquement les sessions legacy + répertoire agent au démarrage afin que l'historique/authentification/modèles atterrissent dans le chemin par agent sans exécution manuelle de doctor. L'authentification WhatsApp est intentionnellement migrée uniquement via `openclaw doctor`.

### 4) Vérifications d'intégrité d'état (persistance de session, routage et sécurité)

Le répertoire d'état est le tronc cérébral opérationnel. S'il disparaît, vous perdez les sessions, identifiants, logs et la configuration (sauf si vous avez des sauvegardes ailleurs).

Doctor vérifie :

- **Répertoire d'état manquant** : avertit de la perte d'état catastrophique, propose de recréer le répertoire et rappelle qu'il ne peut pas récupérer les données manquantes.
- **Permissions du répertoire d'état** : vérifie l'écriture ; propose de réparer les permissions (et émet un indice `chown` lorsqu'une incompatibilité propriétaire/groupe est détectée).
- **Répertoires de session manquants** : `sessions/` et le répertoire du stockage de session sont requis pour persister l'historique et éviter les crashs `ENOENT`.
- **Incompatibilité de transcription** : avertit lorsque des entrées de session récentes ont des fichiers de transcription manquants.
- **Session principale « JSONL 1 ligne »** : signale lorsque la transcription principale n'a qu'une seule ligne (l'historique ne s'accumule pas).
- **Répertoires d'état multiples** : avertit lorsque plusieurs dossiers `~/.openclaw` existent à travers les répertoires home ou lorsque `OPENCLAW_STATE_DIR` pointe ailleurs (l'historique peut se diviser entre les installations).
- **Rappel mode remote** : si `gateway.mode=remote`, doctor rappelle d'exécuter la commande sur l'hôte distant (l'état s'y trouve).
- **Permissions du fichier de configuration** : avertit si `~/.openclaw/openclaw.json` est lisible par le groupe/monde et propose de resserrer à `600`.

### 5) Santé de l'authentification des modèles (expiration OAuth)

Doctor inspecte les profils OAuth dans le stockage d'authentification, avertit lorsque les jetons expirent/sont expirés et peut les rafraîchir lorsque c'est sûr. Si le profil Anthropic Claude Code est obsolète, il suggère d'exécuter `claude setup-token` (ou de coller un setup-token).
Les invites de rafraîchissement n'apparaissent que lors d'une exécution interactive (TTY) ; `--non-interactive` ignoré les tentatives de rafraîchissement.

Doctor signale aussi les profils d'authentification temporairement inutilisables en raison de :

- cooldowns courts (limités de débit/timeouts/échecs d'authentification)
- désactivations plus longues (échecs de facturation/crédit)

### 6) Validation du modèle des hooks

Si `hooks.gmail.model` est défini, doctor valide la référence de modèle par rapport au catalogue et à la liste autorisée et avertit lorsqu'il ne résoudra pas ou est interdit.

### 7) Réparation d'image bac à sable

Lorsque l'isolation en bac à sable est activée, doctor vérifie les images Docker et propose de construire ou de basculer vers les noms legacy si l'image actuelle est manquante.

### 8) Migrations de service gateway et indices de nettoyage

Doctor détecte les services gateway legacy (launchd/systemd/schtasks) et propose de les supprimer et d'installer le service OpenClaw en utilisant le port gateway actuel. Il peut aussi rechercher des services de type gateway supplémentaires et afficher des indices de nettoyage. Les services gateway OpenClaw nommés par profil sont considérés comme de premier ordre et ne sont pas signalés comme « supplémentaires ».

### 9) Avertissements de sécurité

Doctor émet des avertissements lorsqu'un fournisseur est ouvert aux DMs sans liste autorisée, ou lorsqu'une politique est configurée de manière dangereuse.

### 10) linger systemd (Linux)

Si le gateway s'exécute en tant que service utilisateur systemd, doctor s'assuré que le lingering est activé pour que le gateway reste actif après la déconnexion.

### 11) Statut des skills

Doctor affiche un résumé rapide des skills éligibles/manquants/bloqués pour le workspace actuel.

### 12) Vérifications d'authentification gateway (jeton local)

Doctor avertit lorsque `gateway.auth` est absent sur un gateway local et propose de générer un jeton. Utilisez `openclaw doctor --generate-gateway-token` pour forcer la création de jeton en automatisation.

### 13) Vérification de santé gateway + redémarrage

Doctor exécute une vérification de santé et propose de redémarrer le gateway lorsqu'il semble défaillant.

### 14) Avertissements de statut de canal

Si le gateway est sain, doctor exécute un sondage de statut de canal et signale les avertissements avec des corrections suggérées.

### 15) Audit + réparation de configuration superviseur

Doctor vérifie la configuration superviseur installée (launchd/systemd/schtasks) pour les valeurs par défaut manquantes ou obsolètes (par exemple, les dépendances network-online systemd et le délai de redémarrage). Lorsqu'il trouve une incompatibilité, il recommandé une mise à jour et peut réécrire le fichier de service/tâche avec les valeurs par défaut actuelles.

Notes :

- `openclaw doctor` invite avant de réécrire la configuration superviseur.
- `openclaw doctor --yes` accepte les invites de réparation par défaut.
- `openclaw doctor --repair` applique les corrections recommandées sans invitation.
- `openclaw doctor --repair --force` écrase les configurations superviseur personnalisées.
- Vous pouvez toujours forcer une réécriture complète via `openclaw gateway install --force`.

### 16) Diagnostics runtime + port du gateway

Doctor inspecte le runtime du service (PID, dernier statut de sortie) et avertit lorsque le service est installé mais n'est pas réellement en cours d'exécution. Il vérifie aussi les collisions de port sur le port gateway (par défaut `18789`) et signale les causes probables (gateway déjà en cours d'exécution, tunnel SSH).

### 17) Bonnes pratiques du runtime gateway

Doctor avertit lorsque le service gateway s'exécute sur Bun ou un chemin Node géré par un gestionnaire de versions (`nvm`, `fnm`, `volta`, `asdf`, etc.). Les canaux WhatsApp + Telegram nécessitent Node, et les chemins de gestionnaire de versions peuvent casser après les mises à jour car le service ne charge pas votre init shell. Doctor propose de migrer vers une installation Node système lorsqu'elle est disponible (Homebrew/apt/choco).

### 18) Écriture de configuration + métadonnées de l'assistant

Doctor persiste tous les changements de configuration et horodate les métadonnées de l'assistant pour enregistrer l'exécution de doctor.

### 19) Conseils workspace (sauvegarde + système de mémoire)

Doctor suggère un système de mémoire workspace lorsqu'il est absent et affiche un conseil de sauvegarde si le workspace n'est pas déjà sous git.

Voir [/concepts/agent-workspace](/concepts/agent-workspace) pour un guide complet sur la structure du workspace et la sauvegarde git (GitHub ou GitLab privé recommandé).
