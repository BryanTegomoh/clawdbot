---
summary: "Refactorisation Clawnet : unifier le protocole réseau, les rôles, l'authentification, les approbations et l'identité"
read_when:
  - Planification d'un protocole réseau unifié pour les nœuds + clients opérateurs
  - Retravail des approbations, de l'appairage, du TLS et de la présence entre appareils
title: "Refactorisation Clawnet"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/refactor/clawnet.md
  workflow: manual
---

# Refactorisation Clawnet (unification protocole + authentification)

## Bonjour

Bonjour Peter, bonne direction ; cela débloque une UX plus simple + une sécurité renforcée.

## Objectif

Document unique et rigoureux pour :

- État actuel : protocoles, flux, frontières de confiance.
- Points de douleur : approbations, routage multi-sauts, duplication d'interface.
- Nouvel état proposé : un protocole, rôles scopés, authentification/appairage unifié, pinning TLS.
- Modèle d'identité : IDs stables + slugs mignons.
- Plan de migration, risques, questions ouvertes.

## Objectifs (issus de la discussion)

- Un protocole pour tous les clients (app mac, CLI, iOS, Android, nœud headless).
- Chaque participant au réseau authentifié + appairé.
- Clarté des rôles : nœuds vs opérateurs.
- Approbations centrales routées vers l'endroit où se trouve l'utilisateur.
- Chiffrement TLS + pinning optionnel pour tout le trafic distant.
- Duplication de code minimale.
- Une seule machine devrait apparaître une seule fois (pas d'entrée dupliquée UI/nœud).

## Hors périmètre (explicite)

- Supprimer la séparation de capacités (le moindre privilège reste nécessaire).
- Exposer le plan de contrôle complet du Gateway sans vérifications de scope.
- Faire dépendre l'authentification des labels humains (les slugs restent non-sécuritaires).

---

# État actuel (tel quel)

## Deux protocoles

### 1) WebSocket Gateway (plan de contrôle)

- Surface API complète : config, canaux, modèles, sessions, exécutions d'agent, logs, nœuds, etc.
- Bind par défaut : loopback. Accès distant via SSH/Tailscale.
- Authentification : token/mot de passe via `connect`.
- Pas de pinning TLS (s'appuie sur loopback/tunnel).
- Code :
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

### 2) Bridge (transport nœud)

- Surface allowlist restreinte, identité de nœud + appairage.
- JSONL sur TCP ; TLS optionnel + pinning d'empreinte de certificat.
- TLS annonce l'empreinte dans le TXT de découverte.
- Code :
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

## Clients du plan de contrôle aujourd'hui

- CLI -> Gateway WS via `callGateway` (`src/gateway/call.ts`).
- UI app macOS -> Gateway WS (`GatewayConnection`).
- UI web de contrôle -> Gateway WS.
- ACP -> Gateway WS.
- Le contrôle navigateur utilisé son propre serveur HTTP de contrôle.

## Nœuds aujourd'hui

- L'app macOS en mode nœud se connecte au bridge Gateway (`MacNodeBridgeSession`).
- Les apps iOS/Android se connectent au bridge Gateway.
- Appairage + token par nœud stockés sur le Gateway.

## Flux d'approbation actuel (exec)

- L'agent utilise `system.run` via le Gateway.
- Le Gateway invoque le nœud via le bridge.
- Le runtime du nœud décide de l'approbation.
- Le prompt UI est affiché par l'app mac (quand nœud == app mac).
- Le nœud retourne `invoke-res` au Gateway.
- Multi-sauts, UI liée à l'hôte du nœud.

## Présence + identité aujourd'hui

- Entrées de présence Gateway depuis les clients WS.
- Entrées de présence nœud depuis le bridge.
- L'app mac peut afficher deux entrées pour la même machine (UI + nœud).
- L'identité du nœud est stockée dans le store d'appairage ; l'identité UI est séparée.

---

# Problèmes / points de douleur

- Deux piles protocolaires à maintenir (WS + Bridge).
- Approbations sur les nœuds distants : le prompt apparaît sur l'hôte du nœud, pas là où l'utilisateur se trouve.
- Le pinning TLS n'existe que pour le bridge ; WS dépend de SSH/Tailscale.
- Duplication d'identité : la même machine apparaît comme plusieurs instances.
- Rôles ambigus : les capacités UI + nœud + CLI ne sont pas clairement séparées.

---

# Nouvel état proposé (Clawnet)

## Un protocole, deux rôles

Protocole WS unique avec rôle + scope.

- **Rôle : nœud** (hôte de capacités)
- **Rôle : opérateur** (plan de contrôle)
- **Scope** optionnel pour opérateur :
  - `operator.read` (statut + visualisation)
  - `operator.write` (exécution d'agent, envois)
  - `operator.admin` (config, canaux, modèles)

### Comportements de rôle

**Nœud**

- Peut enregistrer des capacités (`caps`, `commands`, permissions).
- Peut recevoir des commandes `invoke` (`system.run`, `camera.*`, `canvas.*`, `screen.record`, etc).
- Peut envoyer des événements : `voice.transcript`, `agent.request`, `chat.subscribe`.
- Ne peut pas appeler les API du plan de contrôle config/modèles/canaux/sessions/agent.

**Opérateur**

- API complète du plan de contrôle, gatée par scope.
- Reçoit toutes les approbations.
- N'exécute pas directement les actions OS ; route vers les nœuds.

### Règle clé

Le rôle est par connexion, pas par appareil. Un appareil peut ouvrir les deux rôles, séparément.

---

# Authentification + appairage unifiés

## Identité client

Chaque client fournit :

- `deviceId` (stable, dérivé de la clé de l'appareil).
- `displayName` (nom humain).
- `rôle` + `scope` + `caps` + `commands`.

## Flux d'appairage (unifié)

- Le client se connecte non authentifié.
- Le Gateway crée une **demande d'appairage** pour ce `deviceId`.
- L'opérateur reçoit le prompt ; approuve/refuse.
- Le Gateway émet des identifiants liés à :
  - la clé publique de l'appareil
  - le(s) rôle(s)
  - le(s) scope(s)
  - les capacités/commandes
- Le client persiste le token, se reconnecte authentifié.

## Authentification liée à l'appareil (éviter le rejeu de bearer token)

Préféré : paires de clés de l'appareil.

- L'appareil génère une paire de clés une fois.
- `deviceId = fingerprint(publicKey)`.
- Le Gateway envoie un nonce ; l'appareil signe ; le Gateway vérifie.
- Les tokens sont émis vers une clé publique (preuve de possession), pas une chaîne.

Alternatives :

- mTLS (certificats client) : le plus fort, plus de complexité opérationnelle.
- Bearer tokens de courte durée uniquement comme phase temporaire (rotation + révocation rapides).

## Approbation silencieuse (heuristique SSH)

La définir précisément pour éviter un maillon faible. Préférer un :

- **Local uniquement** : auto-appairage quand le client se connecte via loopback/socket Unix.
- **Challenge via SSH** : le Gateway émet un nonce ; le client prouve SSH en le récupérant.
- **Fenêtre de présence physique** : après une approbation locale sur l'UI hôte du Gateway, permettre l'auto-appairage pendant une courte fenêtre (ex. 10 minutes).

Toujours journaliser + enregistrer les auto-approbations.

---

# TLS partout (dev + prod)

## Réutiliser le TLS bridge existant

Utiliser le runtime TLS actuel + pinning d'empreinte :

- `src/infra/bridge/server/tls.ts`
- logique de vérification d'empreinte dans `src/node-host/bridge-client.ts`

## Appliquer au WS

- Le serveur WS supporte TLS avec le même cert/clé + empreinte.
- Les clients WS peuvent pinner l'empreinte (optionnel).
- La découverte annonce TLS + empreinte pour tous les endpoints.
  - La découverte est uniquement des indices de localisation ; jamais une ancre de confiance.

## Pourquoi

- Réduire la dépendance à SSH/Tailscale pour la confidentialité.
- Rendre les connexions mobiles distantes sûres par défaut.

---

# Refonte des approbations (centralisées)

## Actuel

L'approbation se fait sur l'hôte du nœud (runtime nœud app mac). Le prompt apparaît là où le nœud tourne.

## Proposé

L'approbation est **hébergée par le Gateway**, l'UI livrée aux clients opérateurs.

### Nouveau flux

1. Le Gateway reçoit l'intention `system.run` (agent).
2. Le Gateway crée un enregistrement d'approbation : `approval.requested`.
3. Les UI opérateur montrent le prompt.
4. La décision d'approbation est envoyée au Gateway : `approval.resolve`.
5. Le Gateway invoque la commande du nœud si approuvé.
6. Le nœud exécute, retourne `invoke-res`.

### Sémantique d'approbation (durcissement)

- Diffusion à tous les opérateurs ; seule l'UI activé montre une modale (les autres reçoivent un toast).
- La première résolution gagne ; le Gateway rejette les résolutions suivantes comme déjà réglées.
- Timeout par défaut : refus après N secondes (ex. 60s), journaliser la raison.
- La résolution requiert le scope `operator.approvals`.

## Bénéfices

- Le prompt apparaît là où l'utilisateur se trouve (mac/téléphone).
- Approbations cohérentes pour les nœuds distants.
- Le runtime du nœud reste headless ; pas de dépendance UI.

---

# Exemples de clarté des rôles

## App iPhone

- **Rôle nœud** pour : micro, caméra, chat vocal, localisation, push-to-talk.
- **operator.read** optionnel pour le statut et la vue chat.
- **operator.write/admin** optionnel uniquement quand explicitement activé.

## App macOS

- Rôle opérateur par défaut (UI de contrôle).
- Rôle nœud quand « Mac node » est activé (system.run, écran, caméra).
- Même deviceId pour les deux connexions -> entrée UI fusionnée.

## CLI

- Rôle opérateur toujours.
- Scope dérivé par sous-commande :
  - `status`, `logs` -> read
  - `agent`, `message` -> write
  - `config`, `channels` -> admin
  - approbations + appairage -> `operator.approvals` / `operator.pairing`

---

# Identité + slugs

## ID stable

Requis pour l'authentification ; ne change jamais.
Préféré :

- Empreinte de paire de clés (hash de clé publique).

## Slug mignon (thème homard)

Label humain uniquement.

- Exemple : `scarlet-claw`, `saltwave`, `mantis-pinch`.
- Stocké dans le registre Gateway, éditable.
- Gestion des collisions : `-2`, `-3`.

## Regroupement UI

Même `deviceId` entre les rôles -> une seule ligne « Instance » :

- Badge : `operator`, `node`.
- Affiche les capacités + dernière vue.

---

# Stratégie de migration

## Phase 0 : Documenter + aligner

- Publier ce document.
- Inventorier tous les appels protocole + flux d'approbation.

## Phase 1 : Ajouter rôles/scopes au WS

- Étendre les paramètres `connect` avec `rôle`, `scope`, `deviceId`.
- Ajouter le gating par allowlist pour le rôle nœud.

## Phase 2 : Compatibilité bridge

- Conserver le bridge en fonctionnement.
- Ajouter le support nœud WS en parallèle.
- Gater les fonctionnalités derrière un flag de config.

## Phase 3 : Approbations centrales

- Ajouter les événements de demande + résolution d'approbation dans le WS.
- Mettre à jour l'UI de l'app mac pour prompter + répondre.
- Le runtime du nœud arrête de prompter l'UI.

## Phase 4 : Unification TLS

- Ajouter la config TLS pour le WS en utilisant le runtime TLS du bridge.
- Ajouter le pinning aux clients.

## Phase 5 : Déprécier le bridge

- Migrer iOS/Android/mac nœud vers WS.
- Conserver le bridge comme fallback ; supprimer une fois stable.

## Phase 6 : Authentification liée à l'appareil

- Exiger l'identité basée sur clé pour toutes les connexions non locales.
- Ajouter l'UI de révocation + rotation.

---

# Notes de sécurité

- Rôle/allowlist appliqués à la frontière du Gateway.
- Aucun client n'obtient l'API « complète » sans scope opérateur.
- Appairage requis pour _toutes_ les connexions.
- TLS + pinning réduit le risque MITM pour le mobile.
- L'approbation silencieuse SSH est une commodité ; toujours enregistrée + révocable.
- La découverte n'est jamais une ancre de confiance.
- Les revendications de capacités sont vérifiées contre les allowlists serveur par plateforme/type.

# Streaming + gros payloads (médias nœud)

Le plan de contrôle WS est bien pour les petits messages, mais les nœuds font aussi :

- clips caméra
- enregistrements d'écran
- flux audio

Options :

1. Frames binaires WS + chunking + règles de backpressure.
2. Endpoint de streaming séparé (toujours TLS + authentification).
3. Conserver le bridge plus longtemps pour les commandes lourdes en médias, migrer en dernier.

Choisir un avant l'implémentation pour éviter la dérive.

# Politique de capacité + commande

- Les caps/commandes rapportés par le nœud sont traités comme des **revendications**.
- Le Gateway applique des allowlists par plateforme.
- Toute nouvelle commande nécessite l'approbation de l'opérateur ou un changement explicite d'allowlist.
- Auditer les changements avec des timestamps.

# Audit + limitation de débit

- Journaliser : demandes d'appairage, approbations/refus, émission/rotation/révocation de token.
- Limiter le débit du spam d'appairage et des prompts d'approbation.

# Hygiène du protocole

- Version de protocole explicite + codes d'erreur.
- Règles de reconnexion + politique de heartbeat.
- TTL de présence et sémantique de dernière vue.

---

# Questions ouvertes

1. Appareil unique exécutant les deux rôles : modèle de token
   - Recommandation : tokens séparés par rôle (nœud vs opérateur).
   - Même deviceId ; scopes différents ; révocation plus claire.

2. Granularité du scope opérateur
   - read/write/admin + approbations + appairage (minimum viable).
   - Considérer des scopes par fonctionnalité plus tard.

3. UX de rotation + révocation de token
   - Auto-rotation sur changement de rôle.
   - UI pour révoquer par deviceId + rôle.

4. Découverte
   - Étendre le TXT Bonjour actuel pour inclure l'empreinte TLS WS + indices de rôle.
   - Traiter comme des indices de localisation uniquement.

5. Approbation inter-réseau
   - Diffusion à tous les clients opérateurs ; l'UI activé montre la modale.
   - La première réponse gagne ; le Gateway assuré l'atomicité.

---

# Résumé (TL;DR)

- Aujourd'hui : plan de contrôle WS + transport nœud Bridge.
- Douleur : approbations + duplication + deux piles.
- Proposition : un protocole WS avec rôles + scopes explicites, appairage unifié + pinning TLS, approbations hébergées par le Gateway, IDs d'appareil stables + slugs mignons.
- Résultat : UX plus simple, sécurité renforcée, moins de duplication, meilleur routage mobile.
