---
summary: "Dépannage de l'appairage des nœuds, exigences de premier plan, permissions et échecs d'outils"
read_when:
  - Le nœud est connecté mais les outils caméra/canvas/écran/exec échouent
  - Vous avez besoin du modèle mental appairage de nœud versus approbations
title: "Dépannage des nœuds"
sidebarTitle: "Dépannage des nœuds"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/troubleshooting.md
  workflow: manual
---

# Dépannage des nœuds

Utilisez cette page lorsqu'un nœud est visible dans le statut mais que les outils du nœud échouent.

## Escalade de commandes

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis exécutez les vérifications spécifiques au nœud :

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Signaux de bon fonctionnement :

- Le nœud est connecté et appairé pour le rôle `node`.
- `nodes describe` inclut la capacité que vous appelez.
- Les approbations exec affichent le mode/liste blanche attendu.

## Exigences de premier plan

`canvas.*`, `camera.*` et `screen.*` fonctionnent uniquement au premier plan sur les nœuds iOS/Android.

Vérification rapide et correction :

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

Si vous voyez `NODE_BACKGROUND_UNAVAILABLE`, mettez l'application du nœud au premier plan et réessayez.

## Matrice des permissions

| Capacité                     | iOS                                            | Android                                               | Application nœud macOS        | Code d'erreur typique          |
| ---------------------------- | ---------------------------------------------- | ----------------------------------------------------- | ----------------------------- | ------------------------------ |
| `camera.snap`, `camera.clip` | Caméra (+ micro pour l'audio du clip)          | Caméra (+ micro pour l'audio du clip)                 | Caméra (+ micro pour l'audio du clip) | `*_PERMISSION_REQUIRED`        |
| `screen.record`              | Enregistrement d'écran (+ micro optionnel)     | Invite de capture d'écran (+ micro optionnel)         | Enregistrement d'écran        | `*_PERMISSION_REQUIRED`        |
| `location.get`               | En cours d'utilisation ou Toujours (selon le mode) | Localisation premier plan/arrière-plan selon le mode | Permission de localisation    | `LOCATION_PERMISSION_REQUIRED` |
| `system.run`                 | n/a (chemin hôte de nœud)                      | n/a (chemin hôte de nœud)                             | Approbations exec requises    | `SYSTEM_RUN_DENIED`            |

## Appairage versus approbations

Ce sont des portes différentes :

1. **Appairage d'appareil** : ce nœud peut-il se connecter au gateway ?
2. **Approbations exec** : ce nœud peut-il exécuter une commande shell spécifique ?

Vérifications rapides :

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

Si l'appairage est manquant, approuvez d'abord l'appareil du nœud.
Si l'appairage est correct mais que `system.run` échoue, corrigez les approbations exec/la liste blanche.

## Codes d'erreur courants des nœuds

- `NODE_BACKGROUND_UNAVAILABLE` → l'application est en arrière-plan ; mettez-la au premier plan.
- `CAMERA_DISABLED` → la bascule caméra est désactivée dans les paramètres du nœud.
- `*_PERMISSION_REQUIRED` → permission système manquante/refusée.
- `LOCATION_DISABLED` → le mode de localisation est désactivé.
- `LOCATION_PERMISSION_REQUIRED` → le mode de localisation demandé n'est pas accordé.
- `LOCATION_BACKGROUND_UNAVAILABLE` → l'application est en arrière-plan mais seule la permission « En cours d'utilisation » existe.
- `SYSTEM_RUN_DENIED: approval required` → la requête exec nécessite une approbation explicite.
- `SYSTEM_RUN_DENIED: allowlist miss` → la commande est bloquée par le mode liste blanche.
  Sur les hôtes de nœud Windows, les formes wrapper shell comme `cmd.exe /c ...` sont traitées comme des échecs de liste blanche en
  mode liste blanche sauf si approuvées via le flux de demande.

## Boucle de récupération rapide

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

Si toujours bloqué :

- Réapprouvez l'appairage d'appareil.
- Rouvrez l'application du nœud (premier plan).
- Réaccordez les permissions du système.
- Recréez/ajustez la politique d'approbation exec.

Références :

- [/nodes/index](/nodes/index)
- [/nodes/camera](/nodes/camera)
- [/nodes/location-command](/nodes/location-command)
- [/tools/exec-approvals](/tools/exec-approvals)
- [/gateway/pairing](/gateway/pairing)
