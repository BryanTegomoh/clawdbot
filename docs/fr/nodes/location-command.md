---
summary: "Commande de localisation pour les nœuds (location.get), modes de permission et comportement en arrière-plan"
read_when:
  - Ajout du support de localisation pour les nœuds ou de l'interface de permissions
  - Conception de la localisation en arrière-plan + flux push
title: "Commande de localisation"
sidebarTitle: "Commande de localisation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/location-command.md
  workflow: manual
---

# Commande de localisation (nœuds)

## En bref

- `location.get` est une commande de nœud (via `node.invoke`).
- Désactivée par défaut.
- Les paramètres utilisent un sélecteur : Désactivé / En cours d'utilisation / Toujours.
- Bascule séparée : Localisation précise.

## Pourquoi un sélecteur (et pas juste un interrupteur)

Les permissions du système sont multi-niveaux. Nous pouvons exposer un sélecteur dans l'application, mais le système décide toujours de l'autorisation réelle.

- iOS/macOS : l'utilisateur peut choisir **En cours d'utilisation** ou **Toujours** dans les invites système/Paramètres. L'application peut demander une mise à niveau, mais le système peut exiger les Paramètres.
- Android : la localisation en arrière-plan est une permission séparée ; sur Android 10+, elle nécessite souvent un passage par les Paramètres.
- La localisation précise est une autorisation séparée (iOS 14+ « Précis », Android « fin » vs « approximatif »).

Le sélecteur dans l'interface détermine notre mode demandé ; l'autorisation réelle se trouve dans les paramètres du système.

## Modèle de paramètres

Par appareil nœud :

- `location.enabledMode` : `off | whileUsing | always`
- `location.preciseEnabled` : booléen

Comportement de l'interface :

- Sélectionner `whileUsing` demande la permission de premier plan.
- Sélectionner `always` s'assuré d'abord de `whileUsing`, puis demande l'arrière-plan (ou envoie l'utilisateur dans les Paramètres si nécessaire).
- Si le système refuse le niveau demandé, revenir au plus haut niveau accordé et afficher le statut.

## Correspondance des permissions (node.permissions)

Optionnel. Le nœud macOS signale `location` via la carte de permissions ; iOS/Android peuvent l'omettre.

## Commande : `location.get`

Appelée via `node.invoke`.

Paramètres (suggérés) :

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

Charge utile de réponse :

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

Erreurs (codes stables) :

- `LOCATION_DISABLED` : le sélecteur est désactivé.
- `LOCATION_PERMISSION_REQUIRED` : permission manquante pour le mode demandé.
- `LOCATION_BACKGROUND_UNAVAILABLE` : l'application est en arrière-plan mais seul le mode « En cours d'utilisation » est autorisé.
- `LOCATION_TIMEOUT` : pas de fix dans le délai imparti.
- `LOCATION_UNAVAILABLE` : défaillance système / pas de fournisseur disponible.

## Comportement en arrière-plan (futur)

Objectif : le modèle peut demander la localisation même lorsque le nœud est en arrière-plan, mais uniquement lorsque :

- L'utilisateur a sélectionné **Toujours**.
- Le système accorde la localisation en arrière-plan.
- L'application est autorisée à s'exécuter en arrière-plan pour la localisation (mode arrière-plan iOS / service de premier plan Android ou autorisation spéciale).

Flux déclenché par push (futur) :

1. Le Gateway envoie un push au nœud (push silencieux ou données FCM).
2. Le nœud se réveille brièvement et demande la localisation à l'appareil.
3. Le nœud transmet la charge utile au Gateway.

Notes :

- iOS : permission « Toujours » + mode de localisation en arrière-plan requis. Le push silencieux peut être limité ; attendez-vous à des échecs intermittents.
- Android : la localisation en arrière-plan peut nécessiter un service de premier plan ; sinon, attendez-vous à un refus.

## Intégration modèle/outils

- Surface d'outil : l'outil `nodes` ajouté l'action `location_get` (nœud requis).
- CLI : `openclaw nodes location get --node <id>`.
- Directives pour l'agent : n'appeler que lorsque l'utilisateur a activé la localisation et comprend la portée.

## Texte d'interface (suggestions)

- Désactivé : « Le partage de localisation est désactivé. »
- En cours d'utilisation : « Uniquement lorsque OpenClaw est ouvert. »
- Toujours : « Autoriser la localisation en arrière-plan. Nécessite une permission système. »
- Précis : « Utiliser la localisation GPS précise. Désactiver pour partager une localisation approximative. »
