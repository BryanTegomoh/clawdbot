---
summary: "Cycle de vie du Gateway sur macOS (launchd)"
read_when:
  - Intégration de l'application mac avec le cycle de vie du gateway
title: "Cycle de vie du Gateway"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/child-process.md
  workflow: manual
---

# Cycle de vie du Gateway sur macOS

L'application macOS **géré le Gateway via launchd** par défaut et ne lance pas
le Gateway en tant que processus enfant. Elle essaie d'abord de se connecter a un Gateway
déjà en cours d'exécution sur le port configuré ; si aucun n'est accessible, elle active le service launchd
via le CLI externe `openclaw` (pas de runtime intégré). Cela vous donne
un démarrage automatique fiable a la connexion et un redémarrage en cas de crash.

Le mode processus enfant (Gateway lance directement par l'application) **n'est pas utilisé** aujourd'hui.
Si vous avez besoin d'un couplage plus etroit avec l'UI, lancez le Gateway manuellement dans un terminal.

## Comportement par défaut (launchd)

- L'application installe un LaunchAgent par utilisateur labellise `ai.openclaw.gateway`
  (ou `ai.openclaw.<profile>` lors de l'utilisation de `--profile`/`OPENCLAW_PROFILE` ; l'ancien `com.openclaw.*` est supporte).
- Lorsque le mode Local est activé, l'application s'assuré que le LaunchAgent est charge et
  démarre le Gateway si nécessaire.
- Les logs sont écrits dans le chemin de log du gateway launchd (visible dans les paramètres de debogage).

Commandes courantes :

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Remplacez le label par `ai.openclaw.<profile>` lors de l'exécution d'un profil nomme.

## Builds de développement non signes

`scripts/restart-mac.sh --no-sign` est pour les builds locaux rapides lorsque vous n'avez pas
de clés de signature. Pour empecher launchd de pointer vers un binaire relay non signe, il :

- Écrit `~/.openclaw/disable-launchagent`.

Les executions signees de `scripts/restart-mac.sh` effacent ce marqueur s'il est
présent. Pour réinitialiser manuellement :

```bash
rm ~/.openclaw/disable-launchagent
```

## Mode connexion uniquement

Pour forcer l'application macOS a **ne jamais installer ou gérer launchd**, lancez-la avec
`--attach-only` (ou `--no-launchd`). Cela définit `~/.openclaw/disable-launchagent`,
pour que l'application se connecte uniquement a un Gateway déjà en cours d'exécution. Vous pouvez activer le même
comportement dans les paramètres de debogage.

## Mode distant

Le mode distant ne démarre jamais de Gateway local. L'application utilise un tunnel SSH vers
l'hôte distant et se connecte via ce tunnel.

## Pourquoi nous preferons launchd

- Démarrage automatique a la connexion.
- Semantiques de redémarrage/KeepAlive intégrées.
- Logs et supervision previsibles.

Si un vrai mode processus enfant est un jour nécessaire a nouveau, il devrait être documente comme un
mode distinct, explicitement réservé au développement.
