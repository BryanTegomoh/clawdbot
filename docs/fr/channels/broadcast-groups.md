---
summary: "Diffuser un message WhatsApp à plusieurs agents"
read_when:
  - Configuration des groupes de diffusion
  - Débogage des réponses multi-agents dans WhatsApp
status: experimental
title: "Groupes de diffusion"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/broadcast-groups.md
  workflow: manual
---

# Groupes de diffusion

**Statut :** Expérimental
**Version :** Ajouté dans 2026.1.9

## Vue d'ensemble

Les groupes de diffusion permettent à plusieurs agents de traiter et de répondre au même message simultanément. Cela vous permet de créer des équipes d'agents spécialisés qui travaillent ensemble dans un seul groupe WhatsApp ou Message privé, le tout en utilisant un seul numéro de téléphone.

Périmètre actuel : **WhatsApp uniquement** (canal web).

Les groupes de diffusion sont évalués après les listes autorisées de canal et les règles d'activation de groupe. Dans les groupes WhatsApp, cela signifie que les diffusions se produisent quand OpenClaw répondrait normalement (par exemple : lors d'une mention, selon vos paramètres de groupe).

## Cas d'utilisation

### 1. Équipes d'agents spécialisés

Déployez plusieurs agents avec des responsabilités atomiques et ciblées :

```
Groupe : "Development Team"
Agents :
  - CodeReviewer (revue de code)
  - DocumentationBot (génère la documentation)
  - SecurityAuditor (vérifie les vulnérabilités)
  - TestGenerator (suggère des cas de test)
```

Chaque agent traite le même message et fournit sa perspective spécialisée.

### 2. Support multi-langue

```
Groupe : "International Support"
Agents :
  - Agent_EN (répond en anglais)
  - Agent_DE (répond en allemand)
  - Agent_ES (répond en espagnol)
```

### 3. Workflows d'assurance qualité

```
Groupe : "Customer Support"
Agents :
  - SupportAgent (fournit la réponse)
  - QAAgent (vérifie la qualité, ne répond que si des problèmes sont trouvés)
```

### 4. Automatisation de tâches

```
Groupe : "Project Management"
Agents :
  - TaskTracker (met à jour la base de tâches)
  - TimeLogger (enregistre le temps passé)
  - ReportGenerator (crée des résumés)
```

## Configuration

### Configuration de base

Ajoutez une section `broadcast` de niveau supérieur (à côté de `bindings`). Les clés sont des identifiants de pair WhatsApp :

- chats de groupe : JID de groupe (par exemple `120363403215116621@g.us`)
- Messages privés : numéro de téléphone E.164 (par exemple `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Résultat :** Quand OpenClaw répondrait dans ce chat, il exécutera les trois agents.

### Stratégie de traitement

Contrôlez comment les agents traitent les messages :

#### Parallèle (par défaut)

Tous les agents traitent simultanément :

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Séquentiel

Les agents traitent dans l'ordre (chacun attend que le précédent termine) :

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Exemple complet

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Fonctionnement

### Flux de messages

1. **Message entrant** arrive dans un groupe WhatsApp
2. **Vérification de diffusion** : le système vérifie si l'identifiant du pair est dans `broadcast`
3. **Si dans la liste de diffusion** :
   - Tous les agents listés traitent le message
   - Chaque agent a sa propre clé de session et un contexte isolé
   - Les agents traitent en parallèle (par défaut) ou séquentiellement
4. **Si pas dans la liste de diffusion** :
   - Le routage normal s'applique (premier binding correspondant)

Note : les groupes de diffusion ne contournent pas les listes autorisées de canal ou les règles d'activation de groupe (mentions/commandes/etc). Ils changent uniquement _quels agents s'exécutent_ quand un message est éligible au traitement.

### Isolation des sessions

Chaque agent dans un groupe de diffusion maintient séparément :

- **Clés de session** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **Historique de conversation** (l'agent ne voit pas les messages des autres agents)
- **Espace de travail** (bacs à sable séparés si configuré)
- **Accès aux outils** (différentes listes allow/deny)
- **Mémoire/contexte** (IDENTITY.md, SOUL.md, etc. séparés)
- **Tampon de contexte de groupe** (messages récents du groupe utilisés pour le contexte) est partagé par pair, donc tous les agents de diffusion voient le même contexte quand ils sont déclenchés

Cela permet à chaque agent d'avoir :

- Différentes personnalités
- Différents accès aux outils (par exemple, lecture seule vs lecture-écriture)
- Différents modèles (par exemple, opus vs sonnet)
- Différentes Skills installées

### Exemple : Sessions isolées

Dans le groupe `120363403215116621@g.us` avec les agents `["alfred", "baerbel"]` :

**Contexte d'Alfred :**

```
Session : agent:alfred:whatsapp:group:120363403215116621@g.us
Historique : [message utilisateur, réponses précédentes d'alfred]
Espace de travail : /Users/pascal/openclaw-alfred/
Outils : read, write, exec
```

**Contexte de Bärbel :**

```
Session : agent:baerbel:whatsapp:group:120363403215116621@g.us
Historique : [message utilisateur, réponses précédentes de baerbel]
Espace de travail : /Users/pascal/openclaw-baerbel/
Outils : lecture seule
```

## Bonnes pratiques

### 1. Gardez les agents ciblés

Concevez chaque agent avec une responsabilité unique et claire :

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

**Bon :** Chaque agent a une seule tâche
**Mauvais :** Un agent générique "dev-helper"

### 2. Utilisez des noms descriptifs

Rendez clair ce que fait chaque agent :

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Configurez différents accès aux outils

Donnez aux agents uniquement les outils dont ils ont besoin :

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Lecture seule
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Lecture-écriture
    }
  }
}
```

### 4. Surveillez les performances

Avec de nombreux agents, considérez :

- Utiliser `"strategy": "parallel"` (par défaut) pour la vitesse
- Limiter les groupes de diffusion à 5-10 agents
- Utiliser des modèles plus rapides pour les agents plus simples

### 5. Gérez les échecs avec élégance

Les agents échouent indépendamment. L'erreur d'un agent ne bloque pas les autres :

```
Message -> [Agent A OK, Agent B erreur, Agent C OK]
Résultat : Agent A et C répondent, Agent B journalise l'erreur
```

## Compatibilité

### Fournisseurs

Les groupes de diffusion fonctionnent actuellement avec :

- WhatsApp (implémenté)
- Telegram (prévu)
- Discord (prévu)
- Slack (prévu)

### Routage

Les groupes de diffusion fonctionnent en parallèle du routage existant :

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A` : Seul alfred répond (routage normal)
- `GROUP_B` : agent1 ET agent2 répondent (diffusion)

**Priorité :** `broadcast` a la priorité sur `bindings`.

## Dépannage

### Les agents ne répondent pas

**Vérifiez :**

1. Les identifiants des agents existent dans `agents.list`
2. Le format de l'identifiant du pair est correct (par exemple, `120363403215116621@g.us`)
3. Les agents ne sont pas dans les listes de refus

**Déboguer :**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Un seul agent répond

**Cause :** L'identifiant du pair pourrait être dans `bindings` mais pas dans `broadcast`.

**Correction :** Ajoutez à la config de diffusion ou supprimez des bindings.

### Problèmes de performances

**Si lent avec beaucoup d'agents :**

- Réduisez le nombre d'agents par groupe
- Utilisez des modèles plus légers (sonnet au lieu d'opus)
- Vérifiez le temps de démarrage du bac à sable

## Exemples

### Exemple 1 : Équipe de revue de code

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**L'utilisateur envoie :** Un extrait de code
**Réponses :**

- code-formatter : "Indentation corrigée et annotations de type ajoutées"
- security-scanner : "Vulnérabilité d'injection SQL à la ligne 12"
- test-coverage : "La couverture est à 45%, tests manquants pour les cas d'erreur"
- docs-checker : "Docstring manquante pour la fonction `process_data`"

### Exemple 2 : Support multi-langue

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Référence API

### Schéma de configuration

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Champs

- `strategy` (optionnel) : Comment traiter les agents
  - `"parallel"` (par défaut) : Tous les agents traitent simultanément
  - `"sequential"` : Les agents traitent dans l'ordre du tableau
- `[peerId]` : JID de groupe WhatsApp, numéro E.164, ou autre identifiant de pair
  - Valeur : Tableau d'identifiants d'agents qui doivent traiter les messages

## Limitations

1. **Max agents :** Pas de limité stricte, mais 10+ agents peuvent être lents
2. **Contexte partagé :** Les agents ne voient pas les réponses des autres (par conception)
3. **Ordre des messages :** Les réponses parallèles peuvent arriver dans n'importe quel ordre
4. **Limités de débit :** Tous les agents comptent dans les limités de débit WhatsApp

## Améliorations futures

Fonctionnalités prévues :

- [ ] Mode contexte partagé (les agents voient les réponses des autres)
- [ ] Coordination d'agents (les agents peuvent se signaler mutuellement)
- [ ] Sélection dynamique d'agents (choisir les agents selon le contenu du message)
- [ ] Priorités d'agents (certains agents répondent avant les autres)

## Voir aussi

- [Configuration multi-agent](/tools/multi-agent-sandbox-tools)
- [Configuration du routage](/channels/channel-routing)
- [Gestion des sessions](/concepts/sessions)
