---
summary: "Plan de refactorisation : routage d'hôte exec, approbations de nœud et runner headless"
read_when:
  - Conception du routage d'hôte exec ou des approbations exec
  - Implémentation du runner de nœud + IPC UI
  - Ajout de modes de sécurité d'hôte exec et commandes slash
title: "Refactorisation de l'hôte exec"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/refactor/exec-host.md
  workflow: manual
---

# Plan de refactorisation de l'hôte exec

## Objectifs

- Ajouter `exec.host` + `exec.security` pour router l'exécution entre **bac à sable**, **Gateway** et **nœud**.
- Conserver les valeurs par défaut **sûres** : pas d'exécution inter-hôtes sauf si explicitement activée.
- Séparer l'exécution en un **service runner headless** avec UI optionnelle (app macOS) via IPC local.
- Fournir une politique **par agent**, une allowlist, un mode ask et une liaison de nœud.
- Supporter des **modes ask** qui fonctionnent _avec_ ou _sans_ allowlists.
- Multi-plateforme : socket Unix + authentification par token (parité macOS/Linux/Windows).

## Hors périmètre

- Pas de migration d'allowlist legacy ni de support de schéma legacy.
- Pas de PTY/streaming pour l'exec nœud (sortie agrégée uniquement).
- Pas de nouvelle couche réseau au-delà du Bridge + Gateway existants.

## Décisions (verrouillées)

- **Clés de config :** `exec.host` + `exec.security` (surcharge par agent autorisée).
- **Élévation :** conserver `/elevated` comme alias pour l'accès complet au Gateway.
- **Ask par défaut :** `on-miss`.
- **Store d'approbations :** `~/.openclaw/exec-approvals.json` (JSON, pas de migration legacy).
- **Runner :** service système headless ; l'app UI héberge un socket Unix pour les approbations.
- **Identité de nœud :** utiliser le `nodeId` existant.
- **Authentification socket :** socket Unix + token (multi-plateforme) ; séparer plus tard si nécessaire.
- **État de l'hôte nœud :** `~/.openclaw/node.json` (id de nœud + token d'appairage).
- **Hôte exec macOS :** exécuter `system.run` dans l'app macOS ; le service hôte de nœud transfère les requêtes via IPC local.
- **Pas de helper XPC :** rester avec socket Unix + token + vérifications de pair.

## Concepts clés

### Hôte

- `sandbox` : exec Docker (comportement actuel).
- `gateway` : exec sur l'hôte Gateway.
- `node` : exec sur le runner de nœud via Bridge (`system.run`).

### Mode de sécurité

- `deny` : toujours bloquer.
- `allowlist` : autoriser uniquement les correspondances.
- `full` : tout autoriser (équivalent à elevated).

### Mode ask

- `off` : ne jamais demander.
- `on-miss` : demander uniquement quand l'allowlist ne correspond pas.
- `always` : demander à chaque fois.

Ask est **indépendant** de l'allowlist ; l'allowlist peut être utilisée avec `always` ou `on-miss`.

### Résolution de politique (par exec)

1. Résoudre `exec.host` (paramètre outil -> surcharge agent -> valeur par défaut globale).
2. Résoudre `exec.security` et `exec.ask` (même précédence).
3. Si l'hôte est `sandbox`, procéder avec l'exec bac à sable local.
4. Si l'hôte est `gateway` ou `node`, appliquer la politique sécurité + ask sur cet hôte.

## Sécurité par défaut

- Par défaut `exec.host = sandbox`.
- Par défaut `exec.security = deny` pour `gateway` et `node`.
- Par défaut `exec.ask = on-miss` (pertinent uniquement si la sécurité autorisé).
- Si aucune liaison de nœud n'est définie, **l'agent peut cibler n'importe quel nœud**, mais uniquement si la politique le permet.

## Surface de configuration

### Paramètres d'outil

- `exec.host` (optionnel) : `sandbox | gateway | node`.
- `exec.security` (optionnel) : `deny | allowlist | full`.
- `exec.ask` (optionnel) : `off | on-miss | always`.
- `exec.node` (optionnel) : id/nom de nœud à utiliser quand `host=node`.

### Clés de config (globales)

- `tools.exec.host`
- `tools.exec.security`
- `tools.exec.ask`
- `tools.exec.node` (liaison de nœud par défaut)

### Clés de config (par agent)

- `agents.list[].tools.exec.host`
- `agents.list[].tools.exec.security`
- `agents.list[].tools.exec.ask`
- `agents.list[].tools.exec.node`

### Alias

- `/elevated on` = définir `tools.exec.host=gateway`, `tools.exec.security=full` pour la session d'agent.
- `/elevated off` = restaurer les paramètres exec précédents pour la session d'agent.

## Store d'approbations (JSON)

Chemin : `~/.openclaw/exec-approvals.json`

Objectif :

- Politique locale + allowlists pour l'**hôte d'exécution** (Gateway ou runner de nœud).
- Repli ask quand aucune UI n'est disponible.
- Identifiants IPC pour les clients UI.

Schéma proposé (v1) :

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

Notes :

- Pas de formats d'allowlist legacy.
- `askFallback` s'applique uniquement quand `ask` est requis et qu'aucune UI n'est joignable.
- Permissions de fichier : `0600`.

## Service runner (headless)

### Rôle

- Appliquer `exec.security` + `exec.ask` localement.
- Exécuter les commandes système et retourner la sortie.
- Émettre des événements Bridge pour le cycle de vie exec (optionnel mais recommandé).

### Cycle de vie du service

- Launchd/daemon sur macOS ; service système sur Linux/Windows.
- Le JSON d'approbations est local à l'hôte d'exécution.
- L'UI héberge un socket Unix local ; les runners se connectent à la demande.

## Intégration UI (app macOS)

### IPC

- Socket Unix à `~/.openclaw/exec-approvals.sock` (0600).
- Token stocké dans `exec-approvals.json` (0600).
- Vérifications de pair : même UID uniquement.
- Challenge/réponse : nonce + HMAC(token, request-hash) pour prévenir le rejeu.
- TTL court (ex. 10s) + payload max + limitation de débit.

### Flux ask (hôte exec app macOS)

1. Le service nœud reçoit `system.run` du Gateway.
2. Le service nœud se connecte au socket local et envoie le prompt/requête exec.
3. L'app valide pair + token + HMAC + TTL, puis affiche le dialogue si nécessaire.
4. L'app exécute la commande dans le contexte UI et retourne la sortie.
5. Le service nœud retourne la sortie au Gateway.

Si l'UI est absente :

- Appliquer `askFallback` (`deny|allowlist|full`).

### Diagramme (SCI)

```
Agent -> Gateway -> Bridge -> Service nœud (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     App Mac (UI + TCC + system.run)
```

## Identité de nœud + liaison

- Utiliser le `nodeId` existant de l'appairage Bridge.
- Modèle de liaison :
  - `tools.exec.node` restreint l'agent à un nœud spécifique.
  - Si non défini, l'agent peut choisir n'importe quel nœud (la politique applique toujours les valeurs par défaut).
- Résolution de sélection de nœud :
  - correspondance exacte `nodeId`
  - `displayName` (normalisé)
  - `remoteIp`
  - préfixe `nodeId` (>= 6 caractères)

## Événements

### Qui voit les événements

- Les événements système sont **par session** et montrés à l'agent au prochain prompt.
- Stockés dans la file en mémoire du Gateway (`enqueueSystemEvent`).

### Texte des événements

- `Exec started (node=<id>, id=<runId>)`
- `Exec finished (node=<id>, id=<runId>, code=<code>)` + queue de sortie optionnelle
- `Exec denied (node=<id>, id=<runId>, <reason>)`

### Transport

Option A (recommandée) :

- Le runner envoie des frames Bridge `event` : `exec.started` / `exec.finished`.
- Le `handleBridgeEvent` du Gateway mappe ceux-ci en `enqueueSystemEvent`.

Option B :

- L'outil `exec` du Gateway gère le cycle de vie directement (synchrone uniquement).

## Flux exec

### Hôte bac à sable

- Comportement `exec` existant (Docker ou hôte quand non sandboxé).
- PTY supporté en mode non-bac à sable uniquement.

### Hôte Gateway

- Le processus Gateway exécute sur sa propre machine.
- Applique le `exec-approvals.json` local (sécurité/ask/allowlist).

### Hôte nœud

- Le Gateway appelle `node.invoke` avec `system.run`.
- Le runner applique les approbations locales.
- Le runner retourne stdout/stderr agrégés.
- Événements Bridge optionnels pour start/finish/deny.

## Limités de sortie

- Limiter stdout+stderr combinés à **200k** ; conserver les **20k de queue** pour les événements.
- Tronquer avec un suffixe clair (ex. `"… (truncated)"`).

## Commandes slash

- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- Surcharges par agent, par session ; non persistantes sauf sauvegarde via config.
- `/elevated on|off|ask|full` reste un raccourci pour `host=gateway security=full` (avec `full` sautant les approbations).

## Histoire multi-plateforme

- Le service runner est la cible d'exécution portable.
- L'UI est optionnelle ; si absente, `askFallback` s'applique.
- Windows/Linux supportent le même protocole JSON d'approbations + socket.

## Phases d'implémentation

### Phase 1 : config + routage exec

- Ajouter le schéma de config pour `exec.host`, `exec.security`, `exec.ask`, `exec.node`.
- Mettre à jour la plomberie d'outil pour respecter `exec.host`.
- Ajouter la commande slash `/exec` et conserver l'alias `/elevated`.

### Phase 2 : store d'approbations + application au Gateway

- Implémenter le lecteur/écriveur de `exec-approvals.json`.
- Appliquer les modes allowlist + ask pour l'hôte `gateway`.
- Ajouter les limités de sortie.

### Phase 3 : application au runner de nœud

- Mettre à jour le runner de nœud pour appliquer allowlist + ask.
- Ajouter le pont de prompt par socket Unix vers l'UI app macOS.
- Connecter `askFallback`.

### Phase 4 : événements

- Ajouter les événements Bridge nœud -> Gateway pour le cycle de vie exec.
- Mapper en `enqueueSystemEvent` pour les prompts d'agent.

### Phase 5 : polish UI

- App Mac : éditeur d'allowlist, sélecteur par agent, UI de politique ask.
- Contrôles de liaison de nœud (optionnel).

## Plan de test

- Tests unitaires : correspondance d'allowlist (glob + insensible à la casse).
- Tests unitaires : précédence de résolution de politique (paramètre outil -> surcharge agent -> global).
- Tests d'intégration : flux deny/allow/ask du runner de nœud.
- Tests d'événements Bridge : routage événement nœud -> événement système.

## Risques ouverts

- Indisponibilité de l'UI : s'assurer que `askFallback` est respecté.
- Commandes longues : s'appuyer sur timeout + limités de sortie.
- Ambiguïté multi-nœuds : erreur sauf si liaison de nœud ou paramètre nœud explicite.

## Documentation associée

- [Outil exec](/tools/exec)
- [Approbations exec](/tools/exec-approvals)
- [Nœuds](/nodes)
- [Mode élevé](/tools/elevated)
