---
summary: "Sous-agents : lancer des exécutions d'agents isolées qui annoncent les résultats au chat demandeur"
read_when:
  - Vous souhaitez du travail en arrière-plan/parallèle via l'agent
  - Vous modifiez sessions_spawn ou la politique d'outils des sous-agents
  - Vous implémentez ou dépannez des sessions de sous-agents liées aux threads
title: "Sous-agents"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/subagents.md
  workflow: manual
---

# Sous-agents

Les sous-agents sont des exécutions d'agents en arrière-plan lancées depuis une exécution d'agent existante. Ils s'exécutent dans leur propre session (`agent:<agentId>:subagent:<uuid>`) et, une fois terminés, **annoncent** leur résultat au canal de chat du demandeur.

## Commande slash

Utilisez `/subagents` pour inspecter ou contrôler les exécutions de sous-agents pour la **session actuelle** :

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`
- `/subagents steer <id|#> <message>`
- `/subagents spawn <agentId> <task> [--model <model>] [--thinking <level>]`

Contrôles de liaison de threads :

Ces commandes fonctionnent sur les canaux qui supportent les liaisons de threads persistantes. Voir **Canaux supportant les threads** ci-dessous.

- `/focus <subagent-label|session-key|session-id|session-label>`
- `/unfocus`
- `/agents`
- `/session ttl <duration|off>`

`/subagents info` affiche les métadonnées de l'exécution (statut, horodatages, id de session, chemin du transcript, nettoyage).

### Comportement du spawn

`/subagents spawn` démarre un sous-agent en arrière-plan comme une commande utilisateur, pas un relais interne, et envoie une mise à jour finale de complétion au chat du demandeur quand l'exécution se termine.

- La commande spawn est non bloquante ; elle retourne un id d'exécution immédiatement.
- À la complétion, le sous-agent annonce un message de résumé/résultat au canal de chat du demandeur.
- Pour les spawns manuels, la livraison est résiliente :
  - OpenClaw essaie d'abord la livraison directe `agent` avec une clé d'idempotence stable.
  - Si la livraison directe échoue, il se rabat sur le routage par file.
  - Si le routage par file n'est toujours pas disponible, l'annonce est réessayée avec un backoff exponentiel court avant abandon final.
- Le message de complétion est un message système et inclut :
  - `Result` (texte de réponse `assistant`, ou dernier `toolResult` si la réponse de l'assistant est vide)
  - `Status` (`completed successfully` / `failed` / `timed out`)
  - statistiques compactes d'exécution/tokens
- `--model` et `--thinking` substituent les défauts pour cette exécution spécifique.
- Utilisez `info`/`log` pour inspecter les détails et la sortie après complétion.
- `/subagents spawn` est en mode one-shot (`mode: "run"`). Pour des sessions persistantes liées aux threads, utilisez `sessions_spawn` avec `thread: true` et `mode: "session"`.
- Pour les sessions de harnais ACP (Codex, Claude Code, Gemini CLI), utilisez `sessions_spawn` avec `runtime: "acp"` et consultez [Agents ACP](/tools/acp-agents).

Objectifs principaux :

- Paralléliser le travail « recherche / tâche longue / outil lent » sans bloquer l'exécution principale.
- Garder les sous-agents isolés par défaut (séparation de session + sandboxing optionnel).
- Garder la surface d'outils difficile à détourner : les sous-agents ne reçoivent **pas** les outils de session par défaut.
- Supporter une profondeur d'imbrication configurable pour les patterns d'orchestration.

Note de coût : chaque sous-agent a son **propre** contexte et sa consommation de tokens. Pour les tâches lourdes ou répétitives, définissez un modèle moins cher pour les sous-agents et gardez votre agent principal sur un modèle de meilleure qualité. Vous pouvez configurer ceci via `agents.defaults.subagents.model` ou des substitutions par agent.

## Outil

Utilisez `sessions_spawn` :

- Démarre une exécution de sous-agent (`deliver: false`, lane globale : `subagent`)
- Puis exécute une étape d'annonce et publie la réponse d'annonce au canal de chat du demandeur
- Modèle par défaut : hérite de l'appelant sauf si vous définissez `agents.defaults.subagents.model` (ou par agent `agents.list[].subagents.model`) ; un `sessions_spawn.model` explicite l'emporte toujours.
- Thinking par défaut : hérite de l'appelant sauf si vous définissez `agents.defaults.subagents.thinking` (ou par agent `agents.list[].subagents.thinking`) ; un `sessions_spawn.thinking` explicite l'emporte toujours.
- Timeout d'exécution par défaut : si `sessions_spawn.runTimeoutSeconds` est omis, OpenClaw utilise `agents.defaults.subagents.runTimeoutSeconds` quand défini ; sinon il revient à `0` (pas de timeout).

Paramètres de l'outil :

- `task` (obligatoire)
- `label?` (optionnel)
- `agentId?` (optionnel ; spawner sous un autre id d'agent si autorisé)
- `model?` (optionnel ; substitue le modèle du sous-agent ; les valeurs invalides sont ignorées et le sous-agent s'exécute sur le modèle par défaut avec un avertissement dans le résultat de l'outil)
- `thinking?` (optionnel ; substitue le niveau de thinking pour l'exécution du sous-agent)
- `runTimeoutSeconds?` (défaut : `agents.defaults.subagents.runTimeoutSeconds` quand défini, sinon `0` ; quand défini, l'exécution du sous-agent est interrompue après N secondes)
- `thread?` (défaut `false` ; quand `true`, demande une liaison de thread de canal pour cette session de sous-agent)
- `mode?` (`run|session`)
  - défaut : `run`
  - si `thread: true` et `mode` omis, le défaut devient `session`
  - `mode: "session"` nécessite `thread: true`
- `cleanup?` (`delete|keep`, défaut `keep`)

## Sessions liées aux threads

Quand les liaisons de threads sont activées pour un canal, un sous-agent peut rester lié à un thread pour que les messages utilisateur suivants dans ce thread continuent d'être routés vers la même session de sous-agent.

### Canaux supportant les threads

- Discord (actuellement le seul canal supporté) : supporte les sessions de sous-agents persistantes liées aux threads (`sessions_spawn` avec `thread: true`), les contrôles de threads manuels (`/focus`, `/unfocus`, `/agents`, `/session ttl`), et les clés d'adaptateur `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.ttlHours`, et `channels.discord.threadBindings.spawnSubagentSessions`.

Flux rapide :

1. Spawner avec `sessions_spawn` en utilisant `thread: true` (et optionnellement `mode: "session"`).
2. OpenClaw crée ou lié un thread à cette cible de session dans le canal actif.
3. Les réponses et messages suivants dans ce thread sont routés vers la session liée.
4. Utilisez `/session ttl` pour inspecter/mettre à jour le TTL d'auto-détachement.
5. Utilisez `/unfocus` pour détacher manuellement.

Contrôles manuels :

- `/focus <target>` lié le thread actuel (ou en crée un) à une cible de sous-agent/session.
- `/unfocus` supprimé la liaison pour le thread lié actuel.
- `/agents` liste les exécutions activés et l'état de liaison (`thread:<id>` ou `unbound`).
- `/session ttl` ne fonctionne que pour les threads liés focalisés.

Options de configuration :

- Défaut global : `session.threadBindings.enabled`, `session.threadBindings.ttlHours`
- Les clés de substitution de canal et de liaison automatique au spawn sont spécifiques à l'adaptateur. Voir **Canaux supportant les threads** ci-dessus.

Voir [Référence de configuration](/gateway/configuration-reference) et [Commandes slash](/tools/slash-commands) pour les détails actuels des adaptateurs.

Liste d'autorisation :

- `agents.list[].subagents.allowAgents` : liste des ids d'agents qui peuvent être ciblés via `agentId` (`["*"]` pour autoriser tous). Défaut : uniquement l'agent demandeur.

Découverte :

- Utilisez `agents_list` pour voir quels ids d'agents sont actuellement autorisés pour `sessions_spawn`.

Archivage automatique :

- Les sessions de sous-agents sont automatiquement archivées après `agents.defaults.subagents.archiveAfterMinutes` (défaut : 60).
- L'archivage utilisé `sessions.delete` et renomme le transcript en `*.deleted.<timestamp>` (même dossier).
- `cleanup: "delete"` archive immédiatement après l'annonce (conserve toujours le transcript via renommage).
- L'archivage automatique est best-effort ; les minuteries en attente sont perdues si le gateway redémarre.
- `runTimeoutSeconds` n'archive **pas** automatiquement ; il arrête seulement l'exécution. La session reste jusqu'à l'archivage automatique.
- L'archivage automatique s'applique également aux sessions de profondeur 1 et profondeur 2.

## Sous-agents imbriqués

Par défaut, les sous-agents ne peuvent pas spawner leurs propres sous-agents (`maxSpawnDepth: 1`). Vous pouvez activer un niveau d'imbrication en définissant `maxSpawnDepth: 2`, ce qui permet le **pattern d'orchestration** : principal → sous-agent orchestrateur → sous-sous-agents travailleurs.

### Comment activer

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // autoriser les sous-agents à spawner des enfants (défaut : 1)
        maxChildrenPerAgent: 5, // max d'enfants actifs par session d'agent (défaut : 5)
        maxConcurrent: 8, // cap de la lane de concurrence globale (défaut : 8)
        runTimeoutSeconds: 900, // timeout par défaut pour sessions_spawn quand omis (0 = pas de timeout)
      },
    },
  },
}
```

### Niveaux de profondeur

| Profondeur | Forme de la clé de session                    | Rôle                                                    | Peut spawner ?                    |
| ---------- | --------------------------------------------- | ------------------------------------------------------- | --------------------------------- |
| 0          | `agent:<id>:main`                             | Agent principal                                         | Toujours                          |
| 1          | `agent:<id>:subagent:<uuid>`                  | Sous-agent (orchestrateur quand profondeur 2 autorisée) | Seulement si `maxSpawnDepth >= 2` |
| 2          | `agent:<id>:subagent:<uuid>:subagent:<uuid>`  | Sous-sous-agent (travailleur feuille)                   | Jamais                            |

### Chaîne d'annonce

Les résultats remontent la chaîne :

1. Le travailleur de profondeur 2 termine → annonce à son parent (orchestrateur de profondeur 1)
2. L'orchestrateur de profondeur 1 reçoit l'annonce, synthétise les résultats, termine → annonce au principal
3. L'agent principal reçoit l'annonce et livre à l'utilisateur

Chaque niveau ne voit que les annonces de ses enfants directs.

### Politique d'outils par profondeur

- **Profondeur 1 (orchestrateur, quand `maxSpawnDepth >= 2`)** : Reçoit `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` pour pouvoir gérer ses enfants. Les autres outils de session/système restent refusés.
- **Profondeur 1 (feuille, quand `maxSpawnDepth == 1`)** : Pas d'outils de session (comportement par défaut actuel).
- **Profondeur 2 (travailleur feuille)** : Pas d'outils de session ; `sessions_spawn` est toujours refusé en profondeur 2. Ne peut pas spawner d'autres enfants.

### Limité de spawn par agent

Chaque session d'agent (à n'importe quelle profondeur) peut avoir au maximum `maxChildrenPerAgent` (défaut : 5) enfants actifs à la fois. Cela empêche l'emballement de fan-out depuis un seul orchestrateur.

### Arrêt en cascade

L'arrêt d'un orchestrateur de profondeur 1 arrête automatiquement tous ses enfants de profondeur 2 :

- `/stop` dans le chat principal arrête tous les agents de profondeur 1 et cascade à leurs enfants de profondeur 2.
- `/subagents kill <id>` arrête un sous-agent spécifique et cascade à ses enfants.
- `/subagents kill all` arrête tous les sous-agents du demandeur et cascade.

## Authentification

L'authentification des sous-agents est résolue par **id d'agent**, pas par type de session :

- La clé de session du sous-agent est `agent:<agentId>:subagent:<uuid>`.
- Le magasin d'authentification est chargé depuis le `agentDir` de cet agent.
- Les profils d'authentification de l'agent principal sont fusionnés comme **repli** ; les profils de l'agent substituent les profils principaux en cas de conflits.

Note : la fusion est additive, donc les profils principaux sont toujours disponibles comme repli. L'authentification entièrement isolée par agent n'est pas encore supportée.

## Annonce

Les sous-agents rapportent via une étape d'annonce :

- L'étape d'annonce s'exécute à l'intérieur de la session du sous-agent (pas la session du demandeur).
- Si le sous-agent répond exactement `ANNOUNCE_SKIP`, rien n'est publié.
- Sinon la réponse d'annonce est publiée au canal de chat du demandeur via un appel `agent` de suivi (`deliver=true`).
- Les réponses d'annonce préservent le routage de thread/sujet quand disponible sur les adaptateurs de canal.
- Les messages d'annonce sont normalisés en un template stable :
  - `Status:` dérivé du résultat de l'exécution (`success`, `error`, `timeout`, ou `unknown`).
  - `Result:` le contenu du résumé de l'étape d'annonce (ou `(not available)` si manquant).
  - `Notes:` détails d'erreur et autre contexte utile.
- `Status` n'est pas inféré de la sortie du modèle ; il provient des signaux de résultat du runtime.

Les charges utiles d'annonce incluent une ligne de statistiques à la fin (même quand encapsulées) :

- Durée d'exécution (par ex. `runtime 5m12s`)
- Utilisation de tokens (entrée/sortie/total)
- Coût estimé quand la tarification du modèle est configurée (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId`, et chemin du transcript (pour que l'agent principal puisse récupérer l'historique via `sessions_history` ou inspecter le fichier sur disque)

## Politique d'outils (outils des sous-agents)

Par défaut, les sous-agents obtiennent **tous les outils sauf les outils de session** et les outils système :

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Quand `maxSpawnDepth >= 2`, les sous-agents orchestrateurs de profondeur 1 reçoivent en plus `sessions_spawn`, `subagents`, `sessions_list`, et `sessions_history` pour pouvoir gérer leurs enfants.

Substitution via la configuration :

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny l'emporte
        deny: ["gateway", "cron"],
        // si allow est défini, il devient allow-only (deny l'emporte toujours)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrence

Les sous-agents utilisent une lane de file en processus dédiée :

- Nom de la lane : `subagent`
- Concurrence : `agents.defaults.subagents.maxConcurrent` (défaut `8`)

## Arrêt

- L'envoi de `/stop` dans le chat du demandeur interrompt la session du demandeur et arrête toutes les exécutions de sous-agents activés lancées depuis celle-ci, avec cascade aux enfants imbriqués.
- `/subagents kill <id>` arrête un sous-agent spécifique et cascade à ses enfants.

## Limitations

- L'annonce des sous-agents est **best-effort**. Si le gateway redémarre, le travail « annonce de retour » en attente est perdu.
- Les sous-agents partagent toujours les mêmes ressources de processus du gateway ; traitez `maxConcurrent` comme une soupape de sécurité.
- `sessions_spawn` est toujours non bloquant : il retourne `{ status: "accepted", runId, childSessionKey }` immédiatement.
- Le contexte des sous-agents injecte uniquement `AGENTS.md` + `TOOLS.md` (pas `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, ni `BOOTSTRAP.md`).
- La profondeur d'imbrication maximale est 5 (plage `maxSpawnDepth` : 1-5). La profondeur 2 est recommandée pour la plupart des cas d'utilisation.
- `maxChildrenPerAgent` limité les enfants actifs par session (défaut : 5, plage : 1-20).
