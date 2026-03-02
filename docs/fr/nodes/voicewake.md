---
summary: "Mots d'activation vocaux globaux (propriété du Gateway) et synchronisation entre les nœuds"
read_when:
  - Modification du comportement ou des valeurs par défaut des mots d'activation vocaux
  - Ajout de nouvelles plateformes de nœud nécessitant la synchronisation des mots d'activation
title: "Activation vocale"
sidebarTitle: "Activation vocale"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/nodes/voicewake.md
  workflow: manual
---

# Activation vocale (mots d'activation globaux)

OpenClaw traite les **mots d'activation comme une liste globale unique** appartenant au **Gateway**.

- Il n'y a **pas de mots d'activation personnalisés par nœud**.
- **Toute interface de nœud/application peut modifier** la liste ; les modifications sont conservées par le Gateway et diffusées à tous.
- Chaque appareil conserve sa propre bascule **Activation vocale activée/désactivée** (l'interface et les permissions locales diffèrent).

## Stockage (hôte du Gateway)

Les mots d'activation sont stockés sur la machine du gateway à :

- `~/.openclaw/settings/voicewake.json`

Structure :

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## Protocole

### Méthodes

- `voicewake.get` → `{ triggers: string[] }`
- `voicewake.set` avec paramètres `{ triggers: string[] }` → `{ triggers: string[] }`

Notes :

- Les déclencheurs sont normalisés (tronqués, entrées vides supprimées). Les listes vides se rabattent sur les valeurs par défaut.
- Des limités sont appliquées pour la sécurité (plafonds de nombre/longueur).

### Événements

- `voicewake.changed` charge utile `{ triggers: string[] }`

Qui le reçoit :

- Tous les clients WebSocket (application macOS, WebChat, etc.)
- Tous les nœuds connectés (iOS/Android), et également lors de la connexion du nœud comme push initial de « l'état actuel ».

## Comportement côté client

### Application macOS

- Utilisé la liste globale pour contrôler les déclencheurs de `VoiceWakeRuntime`.
- Modifier les « Mots déclencheurs » dans les paramètres d'activation vocale appelle `voicewake.set` puis s'appuie sur la diffusion pour maintenir les autres clients synchronisés.

### Nœud iOS

- Utilisé la liste globale pour la détection de déclencheurs de `VoiceWakeManager`.
- Modifier les mots d'activation dans les Paramètres appelle `voicewake.set` (via le WS du Gateway) et maintient également la détection locale des mots d'activation réactive.

### Nœud Android

- Expose un éditeur de mots d'activation dans les Paramètres.
- Appelle `voicewake.set` via le WS du Gateway pour que les modifications soient synchronisées partout.
