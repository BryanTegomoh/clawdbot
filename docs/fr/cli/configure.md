---
summary: "Référence CLI pour `openclaw configuré` (invites de configuration interactives)"
read_when:
  - Vous souhaitez ajuster les identifiants, appareils ou paramètres par défaut des agents de manière interactive
title: "configuré"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/configuré.md
  workflow: manual
---

# `openclaw configuré`

Invite interactive pour configurer les identifiants, appareils et paramètres par défaut des agents.

Note : La section **Modèle** inclut désormais une sélection multiple pour la liste d'autorisation `agents.defaults.models` (ce qui apparaît dans `/model` et le sélecteur de modèles).

Astuce : `openclaw config` sans sous-commande ouvre le même assistant. Utilisez `openclaw config get|set|unset` pour les modifications non interactives.

Liens :

- Référence de configuration du Gateway : [Configuration](/gateway/configuration)
- CLI Config : [Config](/cli/config)

Notes :

- Le choix de l'emplacement d'exécution du Gateway met toujours à jour `gateway.mode`. Vous pouvez sélectionner "Continuer" sans autres sections si c'est tout ce dont vous avez besoin.
- Les services orientes canaux (Slack/Discord/Matrix/Microsoft Teams) demandent les listes d'autorisation de canaux/salons pendant la configuration. Vous pouvez saisir des noms ou des identifiants ; l'assistant résout les noms en identifiants lorsque possible.

## Exemples

```bash
openclaw configure
openclaw configure --section model --section channels
```
