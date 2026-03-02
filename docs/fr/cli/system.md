---
summary: "Référence CLI pour `openclaw system` (événements système, heartbeat, présence)"
read_when:
  - Vous souhaitez mettre en file d'attente un événement système sans créer un job cron
  - Vous avez besoin d'activer ou désactiver les heartbeats
  - Vous souhaitez inspecter les entrées de présence système
title: "system"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/system.md
  workflow: manual
---

# `openclaw system`

Utilitaires au niveau système pour le Gateway : mettre en file d'attente des événements système, contrôler les heartbeats
et afficher la présence.

## Commandes courantes

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

Met en file d'attente un événement système sur la session **principale**. Le prochain heartbeat l'injectera
en tant que ligne `System:` dans le prompt. Utilisez `--mode now` pour déclencher le heartbeat
immédiatement ; `next-heartbeat` attend le prochain tick programmé.

Options :

- `--text <text>` : texte de l'événement système requis.
- `--mode <mode>` : `now` ou `next-heartbeat` (par défaut).
- `--json` : sortie lisible par machine.

## `system heartbeat last|enable|disable`

Contrôles du heartbeat :

- `last` : afficher le dernier événement heartbeat.
- `enable` : réactiver les heartbeats (utilisez ceci s'ils ont été désactivés).
- `disable` : mettre en pause les heartbeats.

Options :

- `--json` : sortie lisible par machine.

## `system presence`

Lister les entrées de présence système actuelles connues du Gateway (nœuds,
instances et lignes de statut similaires).

Options :

- `--json` : sortie lisible par machine.

## Notes

- Nécessite un Gateway en cours d'exécution accessible par votre configuration actuelle (locale ou distante).
- Les événements système sont éphémères et ne sont pas persistés entre les redémarrages.
