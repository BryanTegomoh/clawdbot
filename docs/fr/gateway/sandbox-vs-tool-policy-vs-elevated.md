---
title: "Bac à sable vs politique d'outils vs mode élevé"
summary: "Pourquoi un outil est bloqué : environnement d'exécution du bac à sable, politique d'autorisation/refus des outils, et contrôles d'exécution en mode élevé"
read_when: "Vous rencontrez un « emprisonnement bac à sable » ou voyez un refus d'outil/mode élevé et voulez la clé de configuration exacte à modifier."
status: activé
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/sandbox-vs-tool-policy-vs-elevated.md
  workflow: manual
---

# Bac à sable vs politique d'outils vs mode élevé

OpenClaw dispose de trois contrôles liés (mais différents) :

1. **Bac à sable** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) décide **où les outils s'exécutent** (Docker vs hôte).
2. **Politique d'outils** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) décide **quels outils sont disponibles/autorisés**.
3. **Mode élevé** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) est une **voie de sortie uniquement pour exec** permettant d'exécuter sur l'hôte lorsque vous êtes dans un bac à sable.

## Débogage rapide

Utilisez l'inspecteur pour voir ce qu'OpenClaw fait _réellement_ :

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

Il affiche :

- le mode/scope/accès espace de travail effectif du bac à sable
- si la session est actuellement dans un bac à sable (principale vs non-principale)
- la politique d'autorisation/refus effective des outils du bac à sable (et si elle provient de l'agent/global/défaut)
- les contrôles du mode élevé et les chemins de clés de correction

## Bac à sable : où les outils s'exécutent

Le bac à sable est contrôlé par `agents.defaults.sandbox.mode` :

- `"off"` : tout s'exécute sur l'hôte.
- `"non-main"` : seules les sessions non-principales sont dans le bac à sable (surprise courante pour les groupes/canaux).
- `"all"` : tout est dans le bac à sable.

Voir [Bac à sable](/gateway/sandboxing) pour la matrice complète (scope, montages d'espaces de travail, images).

### Montages de liaison (vérification rapide de sécurité)

- `docker.binds` _transperce_ le système de fichiers du bac à sable : tout ce que vous montez est visible à l'intérieur du conteneur avec le mode que vous définissez (`:ro` ou `:rw`).
- Le mode par défaut est lecture-écriture si vous omettez le mode ; préférez `:ro` pour les sources/secrets.
- `scope: "shared"` ignoré les montages par agent (seuls les montages globaux s'appliquent).
- Monter `/var/run/docker.sock` donne effectivement le contrôle de l'hôte au bac à sable ; ne faites cela qu'intentionnellement.
- L'accès à l'espace de travail (`workspaceAccess: "ro"`/`"rw"`) est indépendant des modes de montage.

## Politique d'outils : quels outils existent/sont appelables

Deux couches comptent :

- **Profil d'outils** : `tools.profile` et `agents.list[].tools.profile` (liste d'autorisation de base)
- **Profil d'outils par fournisseur** : `tools.byProvider[provider].profile` et `agents.list[].tools.byProvider[provider].profile`
- **Politique d'outils globale/par agent** : `tools.allow`/`tools.deny` et `agents.list[].tools.allow`/`agents.list[].tools.deny`
- **Politique d'outils par fournisseur** : `tools.byProvider[provider].allow/deny` et `agents.list[].tools.byProvider[provider].allow/deny`
- **Politique d'outils du bac à sable** (s'applique uniquement dans le bac à sable) : `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` et `agents.list[].tools.sandbox.tools.*`

Règles pratiques :

- `deny` l'emporte toujours.
- Si `allow` est non vide, tout le reste est traité comme bloqué.
- La politique d'outils est l'arrêt définitif : `/exec` ne peut pas outrepasser un outil `exec` refusé.
- `/exec` ne modifie que les paramètres par défaut de session pour les expéditeurs autorisés ; il n'accorde pas l'accès aux outils.
  Les clés d'outils par fournisseur acceptent soit `provider` (ex. `google-antigravity`) soit `provider/model` (ex. `openai/gpt-5.2`).

### Groupes d'outils (raccourcis)

Les politiques d'outils (globale, agent, bac à sable) prennent en charge les entrées `group:*` qui s'étendent à plusieurs outils :

```json5
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

Groupes disponibles :

- `group:runtime` : `exec`, `bash`, `process`
- `group:fs` : `read`, `write`, `edit`, `apply_patch`
- `group:sessions` : `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory` : `memory_search`, `memory_get`
- `group:ui` : `browser`, `canvas`
- `group:automation` : `cron`, `gateway`
- `group:messaging` : `message`
- `group:nodes` : `nodes`
- `group:openclaw` : tous les outils intégrés d'OpenClaw (exclut les plugins de fournisseurs)

## Mode élevé : « exécuter sur l'hôte » uniquement pour exec

Le mode élevé n'accorde **pas** d'outils supplémentaires ; il n'affecte que `exec`.

- Si vous êtes dans un bac à sable, `/elevated on` (ou `exec` avec `elevated: true`) s'exécute sur l'hôte (des approbations peuvent toujours s'appliquer).
- Utilisez `/elevated full` pour ignorer les approbations d'exécution pour la session.
- Si vous exécutez déjà en direct, le mode élevé est effectivement un no-op (toujours contrôlé).
- Le mode élevé n'est **pas** limité aux compétences et ne **surpasse pas** la politique d'autorisation/refus des outils.
- `/exec` est distinct du mode élevé. Il ajuste uniquement les paramètres par défaut d'exécution par session pour les expéditeurs autorisés.

Contrôles :

- Activation : `tools.elevated.enabled` (et optionnellement `agents.list[].tools.elevated.enabled`)
- Listes d'autorisation par expéditeur : `tools.elevated.allowFrom.<provider>` (et optionnellement `agents.list[].tools.elevated.allowFrom.<provider>`)

Voir [Mode élevé](/tools/elevated).

## Corrections courantes de « prison du bac à sable »

### « L'outil X est bloqué par la politique d'outils du bac à sable »

Clés de correction (choisissez-en une) :

- Désactiver le bac à sable : `agents.defaults.sandbox.mode=off` (ou par agent `agents.list[].sandbox.mode=off`)
- Autoriser l'outil à l'intérieur du bac à sable :
  - retirez-le de `tools.sandbox.tools.deny` (ou par agent `agents.list[].tools.sandbox.tools.deny`)
  - ou ajoutez-le à `tools.sandbox.tools.allow` (ou dans l'autorisation par agent)

### « Je pensais que c'était la session principale, pourquoi est-elle dans le bac à sable ? »

En mode `"non-main"`, les clés de groupe/canal ne sont _pas_ principales. Utilisez la clé de session principale (affichée par `sandbox explain`) ou passez en mode `"off"`.
