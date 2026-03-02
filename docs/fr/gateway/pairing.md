---
summary: "Appairage de nœuds géré par le Gateway (Option B) pour iOS et autres nœuds distants"
read_when:
  - Implémentation des approbations d'appairage de nœuds sans interface macOS
  - Ajout de flux CLI pour approuver les nœuds distants
  - Extension du protocole gateway avec la gestion des nœuds
title: "Appairage géré par le Gateway"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/pairing.md
  workflow: manual
---

# Appairage géré par le Gateway (Option B)

Dans l'appairage géré par le Gateway, le **Gateway** est la source de vérité pour déterminer quels nœuds sont autorisés à se connecter. Les interfaces (application macOS, futurs clients) ne sont que des frontends qui approuvent ou rejettent les requêtes en attente.

**Important :** Les nœuds WS utilisent l'**appairage de périphériques** (rôle `node`) lors du `connect`. `node.pair.*` est un store d'appairage séparé et ne contrôle **pas** le handshake WS. Seuls les clients qui appellent explicitement `node.pair.*` utilisent ce flux.

## Concepts

- **Requête en attente** : un nœud a demandé à rejoindre ; nécessite une approbation.
- **Nœud appairé** : nœud approuvé avec un jeton d'authentification émis.
- **Transport** : l'endpoint WS du Gateway transmet les requêtes mais ne décide pas de l'appartenance. (Le support du bridge TCP legacy est obsolète/supprimé.)

## Fonctionnement de l'appairage

1. Un nœud se connecte au WS du Gateway et demande l'appairage.
2. Le Gateway stocke une **requête en attente** et émet `node.pair.requested`.
3. Vous approuvez ou rejetez la requête (CLI ou interface).
4. À l'approbation, le Gateway émet un **nouveau jeton** (les jetons sont renouvelés lors d'un ré-appairage).
5. Le nœud se reconnecte en utilisant le jeton et est désormais « appairé ».

Les requêtes en attente expirent automatiquement après **5 minutes**.

## Flux CLI (compatible headless)

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` affiche les nœuds appairés/connectés et leurs capacités.

## Surface API (protocole gateway)

Événements :

- `node.pair.requested` : émis lorsqu'une nouvelle requête en attente est créée.
- `node.pair.resolved` : émis lorsqu'une requête est approuvée/rejetée/expirée.

Méthodes :

- `node.pair.request` : créer ou réutiliser une requête en attente.
- `node.pair.list` : lister les nœuds en attente + appairés.
- `node.pair.approve` : approuver une requête en attente (émet un jeton).
- `node.pair.reject` : rejeter une requête en attente.
- `node.pair.verify` : vérifier `{ nodeId, token }`.

Notes :

- `node.pair.request` est idempotent par nœud : les appels répétés retournent la même requête en attente.
- L'approbation génère **toujours** un jeton frais ; aucun jeton n'est jamais retourné par `node.pair.request`.
- Les requêtes peuvent inclure `silent: true` comme indication pour les flux d'auto-approbation.

## Auto-approbation (application macOS)

L'application macOS peut optionnellement tenter une **approbation silencieuse** lorsque :

- la requête est marquée `silent`, et
- l'application peut vérifier une connexion SSH vers l'hôte gateway en utilisant le même utilisateur.

Si l'approbation silencieuse échoue, elle revient au prompt normal « Approuver/Rejeter ».

## Stockage (local, privé)

L'état d'appairage est stocké dans le répertoire d'état du Gateway (par défaut `~/.openclaw`) :

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

Si vous redéfinissez `OPENCLAW_STATE_DIR`, le dossier `nodes/` se déplace avec.

Notes de sécurité :

- Les jetons sont des secrets ; traitez `paired.json` comme sensible.
- La rotation d'un jeton nécessite une ré-approbation (ou la suppression de l'entrée du nœud).

## Comportement du transport

- Le transport est **sans état** ; il ne stocke pas l'appartenance.
- Si le Gateway est hors ligne ou que l'appairage est désactivé, les nœuds ne peuvent pas s'appairer.
- Si le Gateway est en mode distant, l'appairage se fait toujours contre le store du Gateway distant.
