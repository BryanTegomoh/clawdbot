---
summary: "Dépannage de la planification et de la livraison cron et heartbeat"
read_when:
  - Cron ne s'est pas exécuté
  - Cron s'est exécuté mais aucun message n'a été livré
  - Le heartbeat semble silencieux ou ignoré
title: "Dépannage de l'automatisation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/troubleshooting.md
  workflow: manual
---

# Dépannage de l'automatisation

Utilisez cette page pour les problèmes de planification et de livraison (`cron` + `heartbeat`).

## Échelle de commandes

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis exécutez les vérifications d'automatisation :

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron ne se déclenche pas

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Une bonne sortie ressemble à :

- `cron status` rapporte activé et un `nextWakeAtMs` futur.
- La tâche est activée et a une planification/fuseau horaire valide.
- `cron runs` affiche `ok` ou une raison d'omission explicite.

Signatures courantes :

- `cron: scheduler disabled; jobs will not run automatically` : cron désactive dans la config/env.
- `cron: timer tick failed` : le tick du planificateur a planté ; inspectez la pile/le contexte de log environnant.
- `reason: not-due` dans la sortie d'exécution : exécution manuelle appelée sans `--force` et la tâche n'est pas encore due.

## Cron s'est déclenché mais pas de livraison

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

Une bonne sortie ressemble à :

- Le statut d'exécution est `ok`.
- Le mode/cible de livraison sont définis pour les tâches isolées.
- La sonde de canal rapporte le canal cible connecté.

Signatures courantes :

- L'exécution a réussi mais le mode de livraison est `none` : aucun message externe n'est attendu.
- Cible de livraison manquante/invalide (`channel`/`to`) : l'exécution peut réussir en interne mais ignorer la sortie.
- Erreurs d'authentification de canal (`unauthorized`, `missing_scope`, `Forbidden`) : livraison bloquée par les identifiants/permissions du canal.

## Heartbeat supprimé ou ignoré

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

Une bonne sortie ressemble à :

- Heartbeat activé avec un intervalle non nul.
- Le dernier résultat heartbeat est `ran` (ou la raison d'omission est comprise).

Signatures courantes :

- `heartbeat skipped` avec `reason=quiet-hours` : en dehors des `activeHours`.
- `requests-in-flight` : voie principale occupée ; heartbeat différé.
- `empty-heartbeat-file` : heartbeat d'intervalle ignoré parce que `HEARTBEAT.md` n'a pas de contenu actionnable et aucun événement cron tagué n'est en file d'attente.
- `alerts-disabled` : les paramètres de visibilité suppriment les messages heartbeat sortants.

## Pièges de fuseau horaire et activeHours

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

Règles rapides :

- `Config path not found: agents.defaults.userTimezone` signifie que la clé n'est pas définie ; le heartbeat se rabat sur le fuseau horaire de l'hôte (ou `activeHours.timezone` si défini).
- Cron sans `--tz` utilisé le fuseau horaire de l'hôte du Gateway.
- Les `activeHours` du heartbeat utilisent la résolution de fuseau horaire configurée (`user`, `local`, ou tz IANA explicite).
- Les horodatages ISO sans fuseau horaire sont traités comme UTC pour les planifications cron `at`.

Signatures courantes :

- Les tâches s'exécutent à la mauvaise heure murale après un changement de fuseau horaire de l'hôte.
- Le heartbeat est toujours ignoré pendant votre journée parce que `activeHours.timezone` est incorrect.

Liens connexes :

- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)
- [/automation/cron-vs-heartbeat](/automation/cron-vs-heartbeat)
- [/concepts/timezone](/concepts/timezone)
