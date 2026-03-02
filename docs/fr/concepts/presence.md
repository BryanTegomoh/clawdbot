---
summary: "Comment les entrées de présence OpenClaw sont produites, fusionnées et affichées"
read_when:
  - Débogage de l'onglet Instances
  - Investigation de lignes d'instance en double ou obsolètes
  - Modification du connect WS du Gateway ou des balises d'événement système
title: "Présence"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/presence.md
  workflow: manual
---

# Présence

La « présence » d'OpenClaw est une vue légère, au mieux de ce qui est possible, de :

- le **Gateway** lui-même, et
- les **clients connectés au Gateway** (application mac, WebChat, CLI, etc.)

La présence est utilisée principalement pour afficher l'onglet **Instances** de l'application macOS et pour fournir une visibilité rapide à l'opérateur.

## Champs de présence (ce qui apparaît)

Les entrées de présence sont des objets structurés avec des champs comme :

- `instanceId` (optionnel mais fortement recommandé) : identité stable du client (généralement `connect.client.instanceId`)
- `host` : nom d'hôte lisible par l'humain
- `ip` : adresse IP au mieux de ce qui est possible
- `version` : chaîne de version du client
- `deviceFamily` / `modelIdentifier` : indices matériels
- `mode` : `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds` : « secondes depuis la dernière entrée utilisateur » (si connu)
- `reason` : `self`, `connect`, `node-connected`, `periodic`, ...
- `ts` : horodatage de dernière mise à jour (ms depuis epoch)

## Producteurs (d'ou vient la présence)

Les entrées de présence sont produites par plusieurs sources et **fusionnées**.

### 1) Entrée du Gateway lui-même

Le Gateway ensemence toujours une entrée « self » au démarrage pour que les interfaces montrent l'hôte du Gateway avant même qu'un client ne se connecte.

### 2) Connexion WebSocket

Chaque client WS commence avec une requête `connect`. Lors du handshake réussi, le Gateway met à jour/insère une entrée de présence pour cette connexion.

#### Pourquoi les commandes CLI ponctuelles n'apparaissent pas

Le CLI se connecte souvent pour des commandes courtes et ponctuelles. Pour éviter d'encombrer la liste Instances, `client.mode === "cli"` n'est **pas** transformé en entrée de présence.

### 3) Balises `system-event`

Les clients peuvent envoyer des balises périodiques plus riches via la méthode `system-event`. L'application mac utilisé cela pour rapporter le nom d'hôte, l'IP et `lastInputSeconds`.

### 4) Connexions de nœuds (rôle: node)

Quand un nœud se connecte via le WebSocket du Gateway avec `rôle: node`, le Gateway met à jour/insère une entrée de présence pour ce nœud (même flux que les autres clients WS).

## Règles de fusion + déduplication (pourquoi `instanceId` est important)

Les entrées de présence sont stockées dans une seule map en mémoire :

- Les entrées sont indexées par une **clé de présence**.
- La meilleure clé est un `instanceId` stable (depuis `connect.client.instanceId`) qui survit aux redémarrages.
- Les clés sont insensibles à la casse.

Si un client se reconnecte sans `instanceId` stable, il peut apparaître comme une ligne **en double**.

## TTL et taille bornée

La présence est intentionnellement éphémère :

- **TTL :** les entrées de plus de 5 minutes sont élaguées
- **Entrées max :** 200 (les plus anciennes supprimées en premier)

Cela garde la liste fraîche et évite une croissance mémoire non bornée.

## Avertissement distant/tunnel (IPs loopback)

Quand un client se connecte via un tunnel SSH / redirection de port local, le Gateway peut voir l'adresse distante comme `127.0.0.1`. Pour éviter d'écraser une bonne IP rapportée par le client, les adresses distantes loopback sont ignorées.

## Consommateurs

### Onglet Instances macOS

L'application macOS affiche la sortie de `system-presence` et applique un petit indicateur de statut (Actif/Inactif/Obsolète) basé sur l'âge de la dernière mise à jour.

## Conseils de débogage

- Pour voir la liste brute, appelez `system-presence` contre le Gateway.
- Si vous voyez des doublons :
  - confirmez que les clients envoient un `client.instanceId` stable dans le handshake
  - confirmez que les balises périodiques utilisent le même `instanceId`
  - vérifiez si l'entrée dérivée de la connexion est manquante d'`instanceId` (les doublons sont attendus)
