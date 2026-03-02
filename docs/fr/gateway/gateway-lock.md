---
summary: "Garde singleton du gateway utilisant le bind du listener WebSocket"
read_when:
  - Exécution ou débogage du processus gateway
  - Investigation de l'application d'instance unique
title: "Verrou du Gateway"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/gateway-lock.md
  workflow: manual
---

# Verrou du gateway

Dernière mise à jour : 2025-12-11

## Pourquoi

- Assurer qu'une seule instance gateway s'exécute par port de base sur le même hôte ; les gateways supplémentaires doivent utiliser des profils isolés et des ports uniques.
- Survivre aux crashs/SIGKILL sans laisser de fichiers verrou obsolètes.
- Échouer rapidement avec une erreur claire lorsque le port de contrôle est déjà occupé.

## Mécanisme

- Le gateway lie le listener WebSocket (par défaut `ws://127.0.0.1:18789`) immédiatement au démarrage en utilisant un listener TCP exclusif.
- Si le bind échoue avec `EADDRINUSE`, le démarrage lève `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- L'OS libère le listener automatiquement à toute sortie de processus, y compris les crashs et SIGKILL, aucun fichier verrou séparé ou étape de nettoyage n'est nécessaire.
- À l'arrêt, le gateway ferme le serveur WebSocket et le serveur HTTP sous-jacent pour libérer le port rapidement.

## Surface d'erreur

- Si un autre processus occupe le port, le démarrage lève `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")`.
- Les autres échecs de bind se manifestent comme `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")`.

## Notes opérationnelles

- Si le port est occupé par un _autre_ processus, l'erreur est la même ; libérez le port ou choisissez-en un autre avec `openclaw gateway --port <port>`.
- L'application macOS maintient toujours sa propre garde PID légère avant de lancer le gateway ; le verrou runtime est appliqué par le bind WebSocket.
