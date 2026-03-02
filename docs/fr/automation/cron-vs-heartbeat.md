---
summary: "Guide pour choisir entre heartbeat et tâches cron pour l'automatisation"
read_when:
  - Choix de la méthode pour planifier des tâches récurrentes
  - Mise en place de surveillance de fond ou de notifications
  - Optimisation de l'utilisation des jetons pour les vérifications périodiques
title: "Cron vs Heartbeat"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/cron-vs-heartbeat.md
  workflow: manual
---

# Cron vs Heartbeat : quand utiliser chacun

Les heartbeats et les tâches cron permettent tous deux d'exécuter des tâches selon un planning. Ce guide vous aide à choisir le bon mécanisme pour votre cas d'utilisation.

## Guide de décision rapide

| Cas d'utilisation                          | Recommandé          | Pourquoi                                        |
| ------------------------------------------ | ------------------- | ----------------------------------------------- |
| Vérifier la boîte de réception toutes les 30 min | Heartbeat           | Regroupe avec d'autres vérifications, sensible au contexte |
| Envoyer un rapport quotidien à 9h pile    | Cron (isolé)        | Timing exact nécessaire                         |
| Surveiller le calendrier pour les événements à venir | Heartbeat           | Adapté naturellement à la conscience périodique  |
| Exécuter une analyse approfondie hebdomadaire | Cron (isolé)        | Tâche autonome, peut utiliser un modèle différent |
| Me rappeler dans 20 minutes               | Cron (principal, `--at`) | Ponctuel avec timing précis                 |
| Vérification de santé de projet en arrière-plan | Heartbeat           | Se greffe sur le cycle existant                 |

## Heartbeat : conscience périodique

Les heartbeats s'exécutent dans la **session principale** à un intervalle régulier (par défaut : 30 min). Ils sont conçus pour que l'agent vérifie les choses et fasse remonter tout ce qui est important.

### Quand utiliser le heartbeat

- **Vérifications périodiques multiples** : au lieu de 5 tâches cron séparées vérifiant la boîte de réception, le calendrier, la météo, les notifications et l'état du projet, un seul heartbeat peut regrouper tout cela.
- **Décisions contextuelles** : l'agent a le contexte complet de la session principale, donc il peut prendre des décisions intelligentes sur ce qui est urgent par rapport à ce qui peut attendre.
- **Continuité conversationnelle** : les exécutions heartbeat partagent la même session, donc l'agent se souvient des conversations récentes et peut faire un suivi naturellement.
- **Surveillance à faible surcharge** : un seul heartbeat remplace de nombreuses petites tâches de polling.

### Avantages du heartbeat

- **Regroupe plusieurs vérifications** : un seul tour d'agent peut examiner la boîte de réception, le calendrier et les notifications ensemble.
- **Réduit les appels API** : un seul heartbeat est moins coûteux que 5 tâches cron isolées.
- **Sensible au contexte** : l'agent sait sur quoi vous travaillez et peut prioriser en conséquence.
- **Suppression intelligente** : si rien ne nécessite d'attention, l'agent répond `HEARTBEAT_OK` et aucun message n'est livré.
- **Timing naturel** : dérive légèrement selon la charge de la file, ce qui convient à la plupart des surveillances.

### Exemple heartbeat : liste de contrôle HEARTBEAT.md

```md
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

L'agent lit ceci à chaque heartbeat et traite tous les éléments en un seul tour.

### Configuration du heartbeat

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // intervalle
        target: "last", // cible de livraison d'alerte explicite (par défaut "none")
        activeHours: { start: "08:00", end: "22:00" }, // optionnel
      },
    },
  },
}
```

Voir [Heartbeat](/gateway/heartbeat) pour la configuration complète.

## Cron : planification précise

Les tâches cron s'exécutent à des moments précis et peuvent s'exécuter dans des sessions isolées sans affecter le contexte principal.
Les planifications récurrentes en début d'heure sont automatiquement réparties par un
décalage déterministe par tâche dans une fenêtre de 0 à 5 minutes.

### Quand utiliser cron

- **Timing exact requis** : "Envoyer ceci à 9h00 chaque lundi" (pas "quelque part autour de 9h").
- **Tâches autonomes** : tâches qui n'ont pas besoin du contexte conversationnel.
- **Modèle/réflexion différent** : analyse lourde justifiant un modèle plus puissant.
- **Rappels ponctuels** : "Me rappeler dans 20 minutes" avec `--at`.
- **Tâches bruyantes/fréquentes** : tâches qui encombreraient l'historique de la session principale.
- **Déclencheurs externes** : tâches devant s'exécuter indépendamment de l'activité de l'agent.

### Avantages de cron

- **Timing précis** : expressions cron à 5 ou 6 champs (secondes) avec support des fuseaux horaires.
- **Répartition de charge intégrée** : les planifications récurrentes en début d'heure sont décalées jusqu'à 5 minutes par défaut.
- **Contrôle par tâche** : surcharger le décalage avec `--stagger <durée>` ou forcer le timing exact avec `--exact`.
- **Isolation de session** : s'exécute dans `cron:<jobId>` sans polluer l'historique principal.
- **Surcharges de modèle** : utiliser un modèle moins cher ou plus puissant par tâche.
- **Contrôle de livraison** : les tâches isolées utilisent par défaut `announce` (résumé) ; choisir `none` si nécessaire.
- **Livraison immédiate** : le mode announce publie directement sans attendre le heartbeat.
- **Pas de contexte d'agent nécessaire** : s'exécute même si la session principale est inactive ou compactée.
- **Support ponctuel** : `--at` pour des horodatages futurs précis.

### Exemple cron : briefing matinal quotidien

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Ceci s'exécute à exactement 7h00 heure de New York, utilisé Opus pour la qualité, et annonce un résumé directement sur WhatsApp.

### Exemple cron : rappel ponctuel

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

Voir [Tâches cron](/automation/cron-jobs) pour la référence CLI complète.

## Organigramme de décision

```
La tâche doit-elle s'exécuter à un moment EXACT ?
  OUI -> Utiliser cron
  NON -> Continuer...

La tâche nécessite-t-elle une isolation de la session principale ?
  OUI -> Utiliser cron (isolé)
  NON -> Continuer...

Cette tâche peut-elle être regroupée avec d'autres vérifications périodiques ?
  OUI -> Utiliser heartbeat (ajouter à HEARTBEAT.md)
  NON -> Utiliser cron

Est-ce un rappel ponctuel ?
  OUI -> Utiliser cron avec --at
  NON -> Continuer...

A-t-elle besoin d'un modèle ou niveau de réflexion différent ?
  OUI -> Utiliser cron (isolé) avec --model/--thinking
  NON -> Utiliser heartbeat
```

## Combiner les deux

La configuration la plus efficace utilise **les deux** :

1. **Heartbeat** gère la surveillance de routine (boîte de réception, calendrier, notifications) en un seul tour regroupé toutes les 30 minutes.
2. **Cron** gère les planifications précises (rapports quotidiens, revues hebdomadaires) et les rappels ponctuels.

### Exemple : configuration d'automatisation efficace

**HEARTBEAT.md** (vérifié toutes les 30 min) :

```md
# Heartbeat checklist

- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Tâches cron** (timing précis) :

```bash
# Briefing matinal quotidien à 7h
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --announce

# Revue de projet hebdomadaire le lundi à 9h
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# Rappel ponctuel
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

## Lobster : workflows déterministes avec approbations

Lobster est le moteur d'exécution de workflows pour les **pipelines d'outils multi-étapes** nécessitant une exécution déterministe et des approbations explicites.
Utilisez-le quand la tâche va au-delà d'un simple tour d'agent, et que vous voulez un workflow reprenables avec des points de contrôle humains.

### Quand Lobster convient

- **Automatisation multi-étapes** : vous avez besoin d'un pipeline fixe d'appels d'outil, pas d'un prompt ponctuel.
- **Portes d'approbation** : les effets secondaires doivent se mettre en pause jusqu'à votre approbation, puis reprendre.
- **Exécutions reprenables** : continuer un workflow en pause sans réexécuter les étapes précédentes.

### Comment ça se combine avec heartbeat et cron

- **Heartbeat/cron** décident _quand_ une exécution a lieu.
- **Lobster** définit _quelles étapes_ se produisent une fois l'exécution démarrée.

Pour les workflows planifiés, utilisez cron ou heartbeat pour déclencher un tour d'agent qui appelle Lobster.
Pour les workflows ad-hoc, appelez Lobster directement.

### Notes opérationnelles (depuis le code)

- Lobster s'exécute comme un **sous-processus local** (CLI `lobster`) en mode outil et renvoie une **enveloppe JSON**.
- Si l'outil renvoie `needs_approval`, vous reprenez avec un `resumeToken` et un flag `approve`.
- L'outil est un **plugin optionnel** ; activez-le de manière additive via `tools.alsoAllow: ["lobster"]` (recommandé).
- Lobster s'attend à ce que le CLI `lobster` soit disponible dans le `PATH`.

Voir [Lobster](/tools/lobster) pour l'utilisation complète et les exemples.

## Session principale vs session isolée

Les heartbeats et cron peuvent tous deux interagir avec la session principale, mais différemment :

|         | Heartbeat                       | Cron (principal)              | Cron (isolé)                   |
| ------- | ------------------------------- | ----------------------------- | ------------------------------ |
| Session | Principale                      | Principale (via événement système) | `cron:<jobId>`            |
| Historique | Partagé                      | Partagé                       | Nouveau à chaque exécution     |
| Contexte | Complet                       | Complet                       | Aucun (démarre propre)         |
| Modèle  | Modèle de session principale    | Modèle de session principale  | Peut être surchargé            |
| Sortie  | Livrée si pas `HEARTBEAT_OK`   | Prompt heartbeat + événement  | Résumé announce (par défaut)   |

### Quand utiliser cron en session principale

Utilisez `--session main` avec `--system-event` quand vous voulez :

- Que le rappel/événement apparaisse dans le contexte de la session principale
- Que l'agent le traite pendant le prochain heartbeat avec le contexte complet
- Pas d'exécution isolée séparée

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

### Quand utiliser cron isolé

Utilisez `--session isolated` quand vous voulez :

- Un environnement vierge sans contexte antérieur
- Des paramètres de modèle ou de réflexion différents
- Des résumés announce directement vers un canal
- Un historique qui n'encombre pas la session principale

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --announce
```

## Considérations de coût

| Mécanisme       | Profil de coût                                               |
| --------------- | ------------------------------------------------------------ |
| Heartbeat       | Un tour toutes les N minutes ; évolue avec la taille de HEARTBEAT.md |
| Cron (principal)| Ajouté un événement au prochain heartbeat (pas de tour isolé) |
| Cron (isolé)    | Tour d'agent complet par tâche ; peut utiliser un modèle moins cher |

**Conseils** :

- Garder `HEARTBEAT.md` petit pour minimiser la surcharge en jetons.
- Regrouper les vérifications similaires dans le heartbeat au lieu de multiples tâches cron.
- Utiliser `target: "none"` sur le heartbeat si vous ne voulez que du traitement interne.
- Utiliser cron isolé avec un modèle moins cher pour les tâches de routine.

## Liens connexes

- [Heartbeat](/gateway/heartbeat) - configuration complète du heartbeat
- [Tâches cron](/automation/cron-jobs) - référence complète CLI et API de cron
- [System](/cli/system) - événements système + contrôles heartbeat
