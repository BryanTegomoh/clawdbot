---
title: Vérification formelle (modèles de sécurité)
summary: Modèles de sécurité vérifiés par machine pour les chemins a plus haut risque d'OpenClaw.
read_when:
  - Examen des garanties ou limités des modèles de sécurité formels
  - Reproduction ou mise a jour des vérifications de modèles de sécurité TLA+/TLC
permalink: /security/formal-verification/
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/security/formal-verification.md
  workflow: manual
---

# Vérification formelle (modèles de sécurité)

Cette page suit les **modèles de sécurité formels** d'OpenClaw (TLA+/TLC aujourd'hui ; d'autres selon les besoins).

> Note: certains liens anciens peuvent faire référence au nom précédent du projet.

**Objectif (etoile du nord) :** fournir un argument vérifié par machine qu'OpenClaw applique sa
politique de sécurité prévue (autorisation, isolation des sessions, contrôle d'accès aux outils et
sécurité de mauvaise configuration), sous des hypotheses explicites.

**Ce que c'est (aujourd'hui) :** une **suite de regression de sécurité** executable, pilotee par l'attaquant :

- Chaque affirmation a une vérification de modèle executable sur un espace d'états fini.
- De nombreuses affirmations ont un **modèle negatif** apparié qui produit une trace de contre-exemple pour une classe de bugs realiste.

**Ce que ce n'est pas (encore) :** une preuve qu'"OpenClaw est sécurisé a tous egards" ou que l'implementation TypeScript complète est correcte.

## Ou se trouvent les modèles

Les modèles sont maintenus dans un dépôt séparé : [vignesh07/openclaw-formal-models](https://github.com/vignesh07/openclaw-formal-models).

## Mises en garde importantes

- Ce sont des **modèles**, pas l'implementation TypeScript complète. Une derive entre le modèle et le code est possible.
- Les résultats sont bornes par l'espace d'états explore par TLC ; "vert" n'implique pas la sécurité au-dela des hypotheses et bornes modelisees.
- Certaines affirmations reposent sur des hypotheses environnementales explicites (par exemple, déploiement correct, entrées de configuration correctes).

## Reproduction des résultats

Aujourd'hui, les résultats sont reproduits en clonant le dépôt des modèles localement et en exécutant TLC (voir ci-dessous). Une iteration future pourrait offrir :

- Des modèles exécutes en CI avec des artefacts publics (traces de contre-exemples, journaux d'exécution)
- Un workflow hébergé "exécuter ce modèle" pour des vérifications petites et bornees

Pour commencer :

```bash
git clone https://github.com/vignesh07/openclaw-formal-models
cd openclaw-formal-models

# Java 11+ required (TLC runs on the JVM).
# The repo vendors a pinned `tla2tools.jar` (TLA+ tools) and provides `bin/tlc` + Make targets.

make <target>
```

### Exposition du Gateway et mauvaise configuration du Gateway ouvert

**Affirmation :** un binding au-dela du loopback sans authentification peut rendre la compromission distante possible / augmente l'exposition ; le jeton/mot de passe bloque les attaquants non authentifiés (selon les hypotheses du modèle).

- Exécutions vertes :
  - `make gateway-exposure-v2`
  - `make gateway-exposure-v2-protected`
- Rouge (attendu) :
  - `make gateway-exposure-v2-negative`

Voir aussi : `docs/gateway-exposure-matrix.md` dans le dépôt des modèles.

### Pipeline nodes.run (capacité a plus haut risque)

**Affirmation :** `nodes.run` nécessite (a) une liste d'autorisation de commandes de nœud plus les commandes declarees et (b) une approbation en direct lorsqu'elle est configurée ; les approbations sont tokenisees pour empêcher la relecture (dans le modèle).

- Exécutions vertes :
  - `make nodes-pipeline`
  - `make approvals-token`
- Rouge (attendu) :
  - `make nodes-pipeline-negative`
  - `make approvals-token-negative`

### Magasin d'appairage (contrôle d'accès DM)

**Affirmation :** les demandes d'appairage respectent le TTL et les limités de demandes en attente.

- Exécutions vertes :
  - `make pairing`
  - `make pairing-cap`
- Rouge (attendu) :
  - `make pairing-negative`
  - `make pairing-cap-negative`

### Contrôle d'accès entrant (mentions + contournement de commande de contrôle)

**Affirmation :** dans les contextes de groupe nécessitant une mention, une "commande de contrôle" non autorisée ne peut pas contourner le contrôle d'accès par mention.

- Vert :
  - `make ingress-gating`
- Rouge (attendu) :
  - `make ingress-gating-negative`

### Isolation routage/clé de session

**Affirmation :** les DM de pairs distincts ne se regroupent pas dans la même session sauf si explicitement liés/configurés.

- Vert :
  - `make routing-isolation`
- Rouge (attendu) :
  - `make routing-isolation-negative`

## v1++ : modèles bornes supplementaires (concurrence, nouvelles tentatives, correction des traces)

Ce sont des modèles complementaires qui renforcent la fidelite autour des modes de defaillance reels (mises a jour non atomiques, nouvelles tentatives et diffusion de messages).

### Concurrence / idempotence du magasin d'appairage

**Affirmation :** un magasin d'appairage doit appliquer `MaxPending` et l'idempotence même sous entrelacements (c'est-a-dire que "vérifier-puis-écrire" doit être atomique / verrouille ; le rafraichissement ne devrait pas créer de doublons).

Ce que cela signifie :

- Sous des requêtes concurrentes, vous ne pouvez pas dépasser `MaxPending` pour un canal.
- Les requêtes/rafraichissements repêtes pour le même `(channel, sender)` ne devraient pas créer de lignes en attente dupliquees.

- Exécutions vertes :
  - `make pairing-race` (vérification atomique/verrouillee du plafond)
  - `make pairing-idempotency`
  - `make pairing-refresh`
  - `make pairing-refresh-race`
- Rouge (attendu) :
  - `make pairing-race-negative` (condition de course non atomique begin/commit du plafond)
  - `make pairing-idempotency-negative`
  - `make pairing-refresh-negative`
  - `make pairing-refresh-race-negative`

### Correlation de trace / idempotence d'ingestion

**Affirmation :** l'ingestion doit preserver la correlation de trace lors de la diffusion et être idempotente lors des nouvelles tentatives du fournisseur.

Ce que cela signifie :

- Lorsqu'un événement externe devient plusieurs messages internes, chaque partie conserve la même identité de trace/événement.
- Les nouvelles tentatives ne provoquent pas de double traitement.
- Si les identifiants d'événement du fournisseur sont manquants, la déduplication se rabat sur une clé sûre (par exemple, l'identifiant de trace) pour éviter de supprimer des événements distincts.

- Vert :
  - `make ingress-trace`
  - `make ingress-trace2`
  - `make ingress-idempotency`
  - `make ingress-dedupe-fallback`
- Rouge (attendu) :
  - `make ingress-trace-negative`
  - `make ingress-trace2-negative`
  - `make ingress-idempotency-negative`
  - `make ingress-dedupe-fallback-negative`

### Précédence dmScope du routage + identityLinks

**Affirmation :** le routage doit garder les sessions DM isolees par défaut, et ne regrouper les sessions que lorsqu'elles sont explicitement configurées (précédence de canal + liens d'identité).

Ce que cela signifie :

- Les remplacements dmScope spécifiques au canal doivent primer sur les valeurs par défaut globales.
- Les identityLinks ne devraient regrouper que dans les groupes explicitement liés, pas entre des pairs non liés.

- Vert :
  - `make routing-precedence`
  - `make routing-identitylinks`
- Rouge (attendu) :
  - `make routing-precedence-negative`
  - `make routing-identitylinks-negative`
