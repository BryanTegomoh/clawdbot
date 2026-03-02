---
summary: "Intégration de PeekabooBridge pour l'automatisation de l'UI macOS"
read_when:
  - Hebergement de PeekabooBridge dans OpenClaw.app
  - Intégration de Peekaboo via Swift Package Manager
  - Modification du protocole/chemins de PeekabooBridge
title: "Peekaboo Bridge"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/peekaboo.md
  workflow: manual
---

# Peekaboo Bridge (automatisation de l'UI macOS)

OpenClaw peut hébergér **PeekabooBridge** en tant que broker d'automatisation UI local et conscient des permissions. Cela permet au CLI `peekaboo` de piloter l'automatisation UI en reutilisant les
permissions TCC de l'application macOS.

## Ce que c'est (et ce que ce n'est pas)

- **Hôte** : OpenClaw.app peut agir comme un hôte PeekabooBridge.
- **Client** : utilisez le CLI `peekaboo` (pas de surface `openclaw ui ...` séparée).
- **UI** : les superpositions visuelles restent dans Peekaboo.app ; OpenClaw est un hôte broker leger.

## Activer le bridge

Dans l'application macOS :

- Paramètres → **Enable Peekaboo Bridge**

Lorsqu'il est activé, OpenClaw démarre un serveur socket UNIX local. S'il est désactivé, l'hôte
est arrêté et `peekaboo` se rabattra sur d'autres hôtes disponibles.

## Ordre de decouverte du client

Les clients Peekaboo essaient généralement les hôtes dans cet ordre :

1. Peekaboo.app (UX complète)
2. Claude.app (si installe)
3. OpenClaw.app (broker leger)

Utilisez `peekaboo bridge status --verbose` pour voir quel hôte est actif et quel
chemin de socket est utilisé. Vous pouvez surcharger avec :

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## Sécurité et permissions

- Le bridge valide les **signatures de code de l'appelant** ; une liste blanche de TeamID est
  appliquee (TeamID de l'hôte Peekaboo + TeamID de l'application OpenClaw).
- Les requêtes expirent après ~10 secondes.
- Si les permissions requises sont manquantes, le bridge retourne un message d'erreur clair
  plutot que de lancer les Préférences Système.

## Comportement des instantanes (automatisation)

Les instantanes sont stockes en mémoire et expirent automatiquement après une courte fenêtre.
Si vous avez besoin d'une retention plus longue, recapturez depuis le client.

## Dépannage

- Si `peekaboo` rapporte « bridge client is not authorized », assurez-vous que le client est
  correctement signe ou exécutez l'hôte avec `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`
  en mode **debug** uniquement.
- Si aucun hôte n'est trouve, ouvrez l'une des applications hôtes (Peekaboo.app ou OpenClaw.app)
  et confirmez que les permissions sont accordees.
