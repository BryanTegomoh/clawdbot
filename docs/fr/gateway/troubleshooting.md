---
summary: "Runbook approfondi de dépannage pour le gateway, les canaux, l'automatisation, les nœuds et le navigateur"
read_when:
  - Le hub de dépannage vous a dirigé ici pour un diagnostic plus approfondi
  - Vous avez besoin de sections de runbook stables basées sur les symptômes avec des commandes exactes
title: "Dépannage"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/troubleshooting.md
  workflow: manual
---

# Dépannage du Gateway

Cette page est le runbook approfondi.
Commencez par [/help/troubleshooting](/help/troubleshooting) si vous souhaitez d'abord le flux de triage rapide.

## Échelle de commandes

Exécutez celles-ci en premier, dans cet ordre :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Signaux sains attendus :

- `openclaw gateway status` affiche `Runtime: running` et `RPC probe: ok`.
- `openclaw doctor` ne signale aucun problème bloquant de configuration/service.
- `openclaw channels status --probe` affiche des canaux connectés/prêts.

## Pas de réponses

Si les canaux sont actifs mais que rien ne répond, vérifiez le routage et la politique avant de reconnecter quoi que ce soit.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

Recherchez :

- Appairage en attente pour les expéditeurs en DM.
- Contrôle de mention de groupe (`requireMention`, `mentionPatterns`).
- Incompatibilités de liste d'autorisation de canal/groupe.

Signatures courantes :

- `drop guild message (mention required` → message de groupe ignoré jusqu'à la mention.
- `pairing request` → l'expéditeur a besoin d'une approbation.
- `blocked` / `allowlist` → l'expéditeur/canal a été filtré par la politique.

Connexe :

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/pairing](/channels/pairing)
- [/channels/groups](/channels/groups)

## Connectivité du tableau de bord / interface de contrôle

Lorsque le tableau de bord/l'interface de contrôle ne se connecte pas, validez l'URL, le mode d'authentification et les hypothèses de contexte sécurisé.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Recherchez :

- URL de sonde et URL du tableau de bord correctes.
- Incompatibilité de mode d'authentification/jeton entre le client et le gateway.
- Utilisation HTTP lorsque l'identité de l'appareil est requise.

Signatures courantes :

- `device identity required` → contexte non sécurisé ou authentification d'appareil manquante.
- `unauthorized` / boucle de reconnexion → incompatibilité jeton/mot de passe.
- `gateway connect failed:` → mauvais hôte/port/cible URL.

Connexe :

- [/web/control-ui](/web/control-ui)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/remote](/gateway/remote)

## Service du Gateway non démarré

Utilisez ceci lorsque le service est installé mais que le processus ne reste pas actif.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Recherchez :

- `Runtime: stopped` avec des indices de sortie.
- Incompatibilité de configuration du service (`Config (cli)` vs `Config (service)`).
- Conflits de port/écouteur.

Signatures courantes :

- `Gateway start blocked: set gateway.mode=local` → le mode gateway local n'est pas activé. Correction : définissez `gateway.mode="local"` dans votre configuration (ou exécutez `openclaw configuré`). Si vous exécutez OpenClaw via Podman avec l'utilisateur dédié `openclaw`, la configuration se trouve dans `~openclaw/.openclaw/openclaw.json`.
- `refusing to bind gateway ... without auth` → liaison non-loopback sans jeton/mot de passe.
- `another gateway instance is already listening` / `EADDRINUSE` → conflit de port.

Connexe :

- [/gateway/background-process](/gateway/background-process)
- [/gateway/configuration](/gateway/configuration)
- [/gateway/doctor](/gateway/doctor)

## Canal connecté mais messages ne transitant pas

Si l'état du canal est connecté mais que le flux de messages est mort, concentrez-vous sur la politique, les permissions et les règles de livraison spécifiques au canal.

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Recherchez :

- Politique DM (`pairing`, `allowlist`, `open`, `disabled`).
- Liste d'autorisation de groupe et exigences de mention.
- Permissions/scopes API du canal manquants.

Signatures courantes :

- `mention required` → message ignoré par la politique de mention du groupe.
- `pairing` / traces d'approbation en attente → l'expéditeur n'est pas approuvé.
- `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → problème d'authentification/permissions du canal.

Connexe :

- [/channels/troubleshooting](/channels/troubleshooting)
- [/channels/whatsapp](/channels/whatsapp)
- [/channels/telegram](/channels/telegram)
- [/channels/discord](/channels/discord)

## Livraison cron et heartbeat

Si le cron ou le heartbeat ne s'est pas exécuté ou n'a pas livré, vérifiez d'abord l'état du planificateur, puis la cible de livraison.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Recherchez :

- Cron active et prochain réveil présent.
- Historique d'exécution des tâches (`ok`, `skipped`, `error`).
- Raisons de saut du heartbeat (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Signatures courantes :

- `cron: scheduler disabled; jobs will not run automatically` → cron désactive.
- `cron: timer tick failed` → tick du planificateur échoué ; vérifiez les erreurs de fichier/log/exécution.
- `heartbeat skipped` avec `reason=quiet-hours` → en dehors de la fenêtre d'heures activés.
- `heartbeat: unknown accountId` → identifiant de compte invalide pour la cible de livraison du heartbeat.
- `heartbeat skipped` avec `reason=dm-blocked` → la cible du heartbeat a ete résolue vers une destination de type DM alors que `agents.defaults.heartbeat.directPolicy` (ou le remplacement par agent) est défini sur `block`.

Connexe :

- [/automation/troubleshooting](/automation/troubleshooting)
- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)

## Échec des outils d'un nœud appairé

Si un nœud est appairé mais que les outils échouent, isolez l'état de premier plan, les permissions et l'état d'approbation.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Recherchez :

- Nœud en ligne avec les capacités attendues.
- Autorisations OS pour caméra/micro/localisation/écran.
- Approbations exec et état de la liste d'autorisation.

Signatures courantes :

- `NODE_BACKGROUND_UNAVAILABLE` → l'application du nœud doit être au premier plan.
- `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → permission OS manquante.
- `SYSTEM_RUN_DENIED: approval required` → approbation exec en attente.
- `SYSTEM_RUN_DENIED: allowlist miss` → commande bloquée par la liste d'autorisation.

Connexe :

- [/nodes/troubleshooting](/nodes/troubleshooting)
- [/nodes/index](/nodes/index)
- [/tools/exec-approvals](/tools/exec-approvals)

## Échec de l'outil navigateur

Utilisez ceci lorsque les actions de l'outil navigateur échouent même si le gateway lui-même est sain.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Recherchez :

- Chemin d'exécutable du navigateur valide.
- Accèssibilité du profil CDP.
- Attachement de l'onglet relais d'extension pour `profile="chrome"`.

Signatures courantes :

- `Failed to start Chrome CDP on port` → le processus du navigateur n'a pas pu démarrer.
- `browser.executablePath not found` → le chemin configuré est invalide.
- `Chrome extension relay is running, but no tab is connected` → le relais d'extension n'est pas attaché.
- `Browser attachOnly is enabled ... not reachable` → le profil attach-only n'a pas de cible accessible.

Connexe :

- [/tools/browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
- [/tools/chrome-extension](/tools/chrome-extension)
- [/tools/browser](/tools/browser)

## Si vous avez mis à jour et que quelque chose s'est soudainement cassé

La plupart des cassures post-mise à jour sont dues à une dérive de configuration ou à des valeurs par défaut plus strictes désormais appliquées.

### 1) Le comportement d'authentification et de remplacement d'URL a changé

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

Ce qu'il faut vérifier :

- Si `gateway.mode=remote`, les appels CLI peuvent cibler le distant alors que votre service local fonctionne bien.
- Les appels explicites `--url` ne se rabattent pas sur les identifiants stockés.

Signatures courantes :

- `gateway connect failed:` → mauvaise cible URL.
- `unauthorized` → point de terminaison accessible mais mauvaise authentification.

### 2) Les garde-fous de liaison et d'authentification sont plus stricts

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

Ce qu'il faut vérifier :

- Les liaisons non-loopback (`lan`, `tailnet`, `custom`) nécessitent une authentification configurée.
- Les anciennes clés comme `gateway.token` ne remplacent pas `gateway.auth.token`.

Signatures courantes :

- `refusing to bind gateway ... without auth` → incompatibilité liaison+authentification.
- `RPC probe: failed` alors que le runtime est en cours d'exécution → gateway actif mais inaccessible avec l'authentification/URL actuelle.

### 3) L'état d'appairage et d'identité d'appareil a changé

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

Ce qu'il faut vérifier :

- Approbations d'appareil en attente pour le tableau de bord/les nœuds.
- Approbations d'appairage DM en attente après des changements de politique ou d'identité.

Signatures courantes :

- `device identity required` → authentification d'appareil non satisfaite.
- `pairing required` → l'expéditeur/l'appareil doit être approuvé.

Si la configuration du service et le runtime ne concordent toujours pas après les vérifications, réinstallez les métadonnées du service depuis le même répertoire de profil/état :

```bash
openclaw gateway install --force
openclaw gateway restart
```

Connexe :

- [/gateway/pairing](/gateway/pairing)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/background-process](/gateway/background-process)
