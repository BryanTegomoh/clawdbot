---
summary: "Exécution en arrière-plan et gestion des processus"
read_when:
  - Ajout ou modification du comportement d'exécution en arrière-plan
  - Débogage de tâches d'exécution longue durée
title: "Exécution en arrière-plan et outil Process"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/background-process.md
  workflow: manual
---

# Exécution en arrière-plan + outil Process

OpenClaw exécute les commandes shell via l'outil `exec` et conserve les tâches longue durée en mémoire. L'outil `process` gère ces sessions en arrière-plan.

## Outil exec

Paramètres clés :

- `command` (obligatoire)
- `yieldMs` (défaut 10000) : passage automatique en arrière-plan après ce délai
- `background` (bool) : passage immédiat en arrière-plan
- `timeout` (secondes, défaut 1800) : arrêt du processus après ce délai
- `elevated` (bool) : exécution sur l'hôte si le mode élevé est activé/autorisé
- Besoin d'un vrai TTY ? Définissez `pty: true`.
- `workdir`, `env`

Comportement :

- Les exécutions au premier plan renvoient la sortie directement.
- Lors du passage en arrière-plan (explicite ou par timeout), l'outil renvoie `status: "running"` + `sessionId` et une courte fin de sortie.
- La sortie est conservée en mémoire jusqu'à ce que la session soit interrogée ou effacée.
- Si l'outil `process` est désactivé, `exec` s'exécute de manière synchrone et ignore `yieldMs`/`background`.

## Pont de processus enfant

Lors du lancement de processus enfants longue durée en dehors des outils exec/process (par exemple, relances CLI ou assistants gateway), attachez l'aide au pont de processus enfant afin que les signaux de terminaison soient transmis et les listeners détachés à la sortie/erreur. Cela évite les processus orphelins sous systemd et maintient un comportement d'arrêt cohérent sur toutes les plateformes.

Remplacements d'environnement :

- `PI_BASH_YIELD_MS` : délai de yield par défaut (ms)
- `PI_BASH_MAX_OUTPUT_CHARS` : limité de sortie en mémoire (caractères)
- `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS` : limité stdout/stderr en attente par flux (caractères)
- `PI_BASH_JOB_TTL_MS` : TTL pour les sessions terminées (ms, borné entre 1 min et 3 h)

Configuration (préférée) :

- `tools.exec.backgroundMs` (défaut 10000)
- `tools.exec.timeoutSec` (défaut 1800)
- `tools.exec.cleanupMs` (défaut 1800000)
- `tools.exec.notifyOnExit` (défaut true) : met en file d'attente un événement système + demande un heartbeat lorsqu'une exécution en arrière-plan se termine.
- `tools.exec.notifyOnExitEmptySuccess` (défaut false) : lorsque activé, met également en file d'attente les événements de complétion pour les exécutions en arrière-plan réussies sans sortie.

## Outil process

Actions :

- `list` : sessions en cours + terminées
- `poll` : drainer la nouvelle sortie d'une session (rapporte aussi le statut de sortie)
- `log` : lire la sortie agrégée (prend en charge `offset` + `limit`)
- `write` : envoyer des données stdin (`data`, `eof` optionnel)
- `kill` : terminer une session en arrière-plan
- `clear` : supprimer une session terminée de la mémoire
- `remove` : arrêter si en cours d'exécution, sinon effacer si terminée

Notes :

- Seules les sessions en arrière-plan sont listées/conservées en mémoire.
- Les sessions sont perdues au redémarrage du processus (pas de persistance sur disque).
- Les logs de session ne sont sauvegardés dans l'historique de chat que si vous exécutez `process poll/log` et que le résultat de l'outil est enregistré.
- `process` est limité par agent ; il ne voit que les sessions démarrées par cet agent.
- `process list` inclut un `name` dérivé (verbe de commande + cible) pour un aperçu rapide.
- `process log` utilise un `offset`/`limit` basé sur les lignes.
- Lorsque `offset` et `limit` sont tous deux omis, les 200 dernières lignes sont renvoyées avec un indice de pagination.
- Lorsque `offset` est fourni et `limit` est omis, la sortie va de `offset` jusqu'à la fin (sans plafond de 200).

## Exemples

Exécuter une tâche longue et interroger plus tard :

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

Démarrer immédiatement en arrière-plan :

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

Envoyer des données stdin :

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```
