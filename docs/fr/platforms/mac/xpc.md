---
summary: "Architecture IPC macOS pour l'application OpenClaw, le transport du nœud gateway et PeekabooBridge"
read_when:
  - Modification des contrats IPC ou de l'IPC de l'application barre de menus
title: "IPC macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/xpc.md
  workflow: manual
---

# Architecture IPC macOS d'OpenClaw

**Modèle actuel :** un socket Unix local connecté le **service hôte de nœud** a l'**application macOS** pour les approbations d'exécution + `system.run`. Un CLI de debogage `openclaw-mac` existe pour les vérifications de decouverte/connexion ; les actions d'agent passent toujours par le WebSocket du Gateway et `node.invoke`. L'automatisation UI utilisé PeekabooBridge.

## Objectifs

- Une seule instance d'application GUI qui possède tout le travail TCC (notifications, enregistrement d'ecran, micro, parole, AppleScript).
- Une surface réduite pour l'automatisation : commandes Gateway + nœud, plus PeekabooBridge pour l'automatisation UI.
- Permissions previsibles : toujours le même bundle ID signe, lance par launchd, pour que les octrois TCC persistent.

## Fonctionnement

### Transport Gateway + nœud

- L'application exécute le Gateway (mode local) et s'y connecté en tant que nœud.
- Les actions d'agent sont effectuees via `node.invoke` (par exemple `system.run`, `system.notify`, `canvas.*`).

### Service de nœud + IPC de l'application

- Un service hôte de nœud headless se connecte au WebSocket du Gateway.
- Les requêtes `system.run` sont transmises a l'application macOS via un socket Unix local.
- L'application effectué l'exécution dans le contexte UI, affiche l'invite si nécessaire, et retourne la sortie.

Diagramme (SCI) :

```
Agent -> Gateway -> Service de noeud (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Application Mac (UI + TCC + system.run)
```

### PeekabooBridge (automatisation UI)

- L'automatisation UI utilisé un socket UNIX séparé nomme `bridge.sock` et le protocole JSON PeekabooBridge.
- Ordre de préférence de l'hôte (côté client) : Peekaboo.app → Claude.app → OpenClaw.app → exécution locale.
- Sécurité : les hôtes du bridge exigent un TeamID autorisé ; la soupape de sécurité DEBUG même-UID est protegee par `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (convention Peekaboo).
- Voir : [Utilisation de PeekabooBridge](/platforms/mac/peekaboo) pour les details.

## Flux operationnels

- Redémarrage/recompilation : `SIGN_IDENTITY="Apple Development: <Developer Name> (<TEAMID>)" scripts/restart-mac.sh`
  - Tue les instances existantes
  - Build Swift + empaquetage
  - Écrit/bootstrap/kickstart le LaunchAgent
- Instance unique : l'application se ferme rapidement si une autre instance avec le même bundle ID est en cours d'exécution.

## Notes de durcissement

- Préférez exiger une correspondance de TeamID pour toutes les surfaces privilegiees.
- PeekabooBridge : `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (DEBUG uniquement) peut autoriser les appelants même-UID pour le développement local.
- Toute communication reste locale uniquement ; aucun socket réseau n'est expose.
- Les invites TCC proviennent uniquement du bundle de l'application GUI ; gardez le bundle ID signe stable entre les recompilations.
- Durcissement IPC : mode socket `0600`, jeton, vérifications peer-UID, defi/réponse HMAC, TTL court.
