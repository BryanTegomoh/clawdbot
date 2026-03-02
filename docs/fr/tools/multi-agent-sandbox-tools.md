---
summary: "Bac à sable par agent + restrictions d'outils, préséance et exemples"
title: Bac à sable & outils multi-agents
read_when: "Vous voulez du sandboxing par agent ou des politiques d'autorisation/refus d'outils par agent dans un gateway multi-agents."
status: activé
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/multi-agent-sandbox-tools.md
  workflow: manual
---

# Configuration du bac à sable & des outils multi-agents

## Vue d'ensemble

Chaque agent dans une configuration multi-agents peut désormais avoir son propre :

- **Configuration de bac à sable** (`agents.list[].sandbox` remplace `agents.defaults.sandbox`)
- **Restrictions d'outils** (`tools.allow` / `tools.deny`, plus `agents.list[].tools`)

Cela vous permet d'exécuter plusieurs agents avec différents profils de sécurité :

- Assistant personnel avec accès complet
- Agents famille/travail avec outils restreints
- Agents publics dans des bacs à sable

`setupCommand` appartient sous `sandbox.docker` (global ou par agent) et s'exécute une fois quand le conteneur est créé.

L'authentification est par agent : chaque agent lit depuis son propre store d'auth `agentDir` à :

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

Les identifiants ne sont **pas** partagés entre agents. Ne réutilisez jamais `agentDir` entre agents.
Si vous voulez partager des identifiants, copiez `auth-profiles.json` dans le `agentDir` de l'autre agent.

Pour le comportement du sandboxing à l'exécution, voir [Sandboxing](/gateway/sandboxing).
Pour déboguer « pourquoi c'est bloqué ? », voir [Sandbox vs politique d'outils vs élevé](/gateway/sandbox-vs-tool-policy-vs-elevated) et `openclaw sandbox explain`.

---

## Exemples de configuration

### Exemple 1 : Agent personnel + agent famille restreint

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**Résultat :**

- Agent `main` : S'exécute sur l'hôte, accès complet aux outils
- Agent `family` : S'exécute dans Docker (un conteneur par agent), outil `read` uniquement

---

### Exemple 2 : Agent travail avec bac à sable partagé

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

---

### Exemple 2b : Profil coding global + agent messaging uniquement

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**Résultat :**

- Les agents par défaut obtiennent les outils de coding
- L'agent `support` est messaging uniquement (+ outil Slack)

---

### Exemple 3 : Différents modes de bac à sable par agent

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // Défaut global
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // Override : main jamais sandboxé
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // Override : public toujours sandboxé
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

---

## Préséance de configuration

Quand les configurations globales (`agents.defaults.*`) et spécifiques à l'agent (`agents.list[].*`) coexistent :

### Configuration du bac à sable

Les paramètres spécifiques à l'agent remplacent les globaux :

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Notes :**

- `agents.list[].sandbox.{docker,browser,prune}.*` remplace `agents.defaults.sandbox.{docker,browser,prune}.*` pour cet agent (ignoré quand le scope du bac à sable se résout à `"shared"`).

### Restrictions d'outils

L'ordre de filtrage est :

1. **Profil d'outils** (`tools.profile` ou `agents.list[].tools.profile`)
2. **Profil d'outils par fournisseur** (`tools.byProvider[provider].profile` ou `agents.list[].tools.byProvider[provider].profile`)
3. **Politique d'outils globale** (`tools.allow` / `tools.deny`)
4. **Politique d'outils par fournisseur** (`tools.byProvider[provider].allow/deny`)
5. **Politique d'outils spécifique à l'agent** (`agents.list[].tools.allow/deny`)
6. **Politique par fournisseur de l'agent** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Politique d'outils du bac à sable** (`tools.sandbox.tools` ou `agents.list[].tools.sandbox.tools`)
8. **Politique d'outils des sous-agents** (`tools.subagents.tools`, si applicable)

Chaque niveau peut restreindre davantage les outils, mais ne peut pas redonner accès à des outils refusés aux niveaux précédents.
Si `agents.list[].tools.sandbox.tools` est défini, il remplace `tools.sandbox.tools` pour cet agent.
Si `agents.list[].tools.profile` est défini, il remplace `tools.profile` pour cet agent.
Les clés d'outils par fournisseur acceptent soit `provider` (par ex. `google-antigravity`) soit `provider/model` (par ex. `openai/gpt-5.2`).

### Groupes d'outils (raccourcis)

Les politiques d'outils (globales, agent, bac à sable) supportent les entrées `group:*` qui se développent en plusieurs outils concrets :

- `group:runtime` : `exec`, `bash`, `process`
- `group:fs` : `read`, `write`, `edit`, `apply_patch`
- `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory` : `memory_search`, `memory_get`
- `group:ui` : `browser`, `canvas`
- `group:automation` : `cron`, `gateway`
- `group:messaging` : `message`
- `group:nodes` : `nodes`
- `group:openclaw` : tous les outils OpenClaw intégrés (exclut les plugins fournisseur)

### Mode élevé

`tools.elevated` est la ligne de base globale (liste d'autorisation basée sur l'expéditeur). `agents.list[].tools.elevated` peut restreindre davantage le mode élevé pour des agents spécifiques (les deux doivent autoriser).

Patterns de mitigation :

- Refuser `exec` pour les agents non fiables (`agents.list[].tools.deny: ["exec"]`)
- Éviter d'ajouter des expéditeurs qui routent vers des agents restreints dans les listes d'autorisation
- Désactiver le mode élevé globalement (`tools.elevated.enabled: false`) si vous ne voulez que de l'exécution sandboxée
- Désactiver le mode élevé par agent (`agents.list[].tools.elevated.enabled: false`) pour les profils sensibles

---

## Migration depuis un agent unique

**Avant (agent unique) :**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**Après (multi-agents avec différents profils) :**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

Les configurations legacy `agent.*` sont migrées par `openclaw doctor` ; préférez `agents.defaults` + `agents.list` à l'avenir.

---

## Exemples de restrictions d'outils

### Agent en lecture seule

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### Agent d'exécution sûre (pas de modifications de fichiers)

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### Agent de communication uniquement

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

---

## Piège courant : « non-main »

`agents.defaults.sandbox.mode: "non-main"` est basé sur `session.mainKey` (défaut `"main"`),
pas sur l'id de l'agent. Les sessions de groupe/canal obtiennent toujours leurs propres clés, donc elles
sont traitées comme non-main et seront sandboxées. Si vous voulez qu'un agent ne soit jamais
sandboxé, définissez `agents.list[].sandbox.mode: "off"`.

---

## Tests

Après avoir configuré le bac à sable et les outils multi-agents :

1. **Vérifier la résolution des agents :**

   ```exec
   openclaw agents list --bindings
   ```

2. **Vérifier les conteneurs de bac à sable :**

   ```exec
   docker ps --filter "name=openclaw-sbx-"
   ```

3. **Tester les restrictions d'outils :**
   - Envoyez un message nécessitant des outils restreints
   - Vérifiez que l'agent ne peut pas utiliser les outils refusés

4. **Surveiller les logs :**

   ```exec
   tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

---

## Dépannage

### Agent non sandboxé malgré `mode: "all"`

- Vérifiez s'il y a un `agents.defaults.sandbox.mode` global qui le remplace
- La configuration spécifique à l'agent a la préséance, donc définissez `agents.list[].sandbox.mode: "all"`

### Outils toujours disponibles malgré la liste de refus

- Vérifiez l'ordre de filtrage des outils : global → agent → bac à sable → sous-agent
- Chaque niveau ne peut que restreindre davantage, pas redonner accès
- Vérifiez dans les logs : `[tools] filtering tools for agent:${agentId}`

### Conteneur non isolé par agent

- Définissez `scope: "agent"` dans la configuration de bac à sable spécifique à l'agent
- Le défaut est `"session"` qui crée un conteneur par session

---

## Voir aussi

- [Routage multi-agents](/concepts/multi-agent)
- [Configuration du bac à sable](/gateway/configuration#agentsdefaults-sandbox)
- [Gestion des sessions](/concepts/session)
