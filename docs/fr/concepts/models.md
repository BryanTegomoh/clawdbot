---
summary: "CLI Models : list, set, aliases, fallbacks, scan, status"
read_when:
  - Ajout ou modification de la CLI models (models list/set/scan/aliases/fallbacks)
  - Changement du comportement de repli de modèle ou de l'UX de sélection
  - Mise à jour des sondes de scan de modèle (outils/images)
title: "CLI Models"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/models.md
  workflow: manual
---

# CLI Models

Voir [/concepts/model-failover](/concepts/model-failover) pour la rotation des profils d'authentification, les temps de repos et comment cela interagit avec les replis.
Vue d'ensemble rapide des fournisseurs + exemples : [/concepts/model-providers](/concepts/model-providers).

## Fonctionnement de la sélection de modèle

OpenClaw sélectionne les modèles dans cet ordre :

1. **Modèle principal** (`agents.defaults.model.primary` ou `agents.defaults.model`).
2. **Replis** dans `agents.defaults.model.fallbacks` (dans l'ordre).
3. **Repli d'authentification du fournisseur** se produit au sein d'un fournisseur avant de passer au modèle suivant.

Associé :

- `agents.defaults.models` est la liste blanche/catalogue des modèles qu'OpenClaw peut utiliser (plus les alias).
- `agents.defaults.imageModel` est utilisé **uniquement quand** le modèle principal ne peut pas accepter d'images.
- Les paramètres par agent peuvent surcharger `agents.defaults.model` via `agents.list[].model` plus les bindings (voir [/concepts/multi-agent](/concepts/multi-agent)).

## Choix rapides de modèles (anecdotiques)

- **GLM** : un peu meilleur pour le code/les appels d'outils.
- **MiniMax** : meilleur pour l'écriture et l'ambiance.

## Assistant de configuration (recommandé)

Si vous ne voulez pas modifier la configuration manuellement, lancez l'assistant d'intégration :

```bash
openclaw onboard
```

Il peut configurer le modèle + l'authentification pour les fournisseurs courants, y compris l'**abonnement OpenAI Code (Codex)** (OAuth) et **Anthropic** (clé API recommandée ; `claude setup-token` également supporté).

## Clés de configuration (vue d'ensemble)

- `agents.defaults.model.primary` et `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel.primary` et `agents.defaults.imageModel.fallbacks`
- `agents.defaults.models` (liste blanche + alias + paramètres du fournisseur)
- `models.providers` (fournisseurs personnalisés écrits dans `models.json`)

Les références de modèle sont normalisées en minuscules. Les alias de fournisseur comme `z.ai/*` se normalisent en `zai/*`.

Les exemples de configuration de fournisseur (y compris OpenCode Zen) se trouvent dans [/gateway/configuration](/gateway/configuration#opencode-zen-multi-model-proxy).

## « Model is not allowed » (et pourquoi les réponses s'arrêtent)

Si `agents.defaults.models` est défini, il devient la **liste blanche** pour `/model` et les surcharges de session. Quand un utilisateur sélectionne un modèle qui n'est pas dans cette liste blanche, OpenClaw retourne :

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Cela se produit **avant** qu'une réponse normale soit générée, donc le message peut donner l'impression qu'il « n'a pas répondu ». La solution est de :

- Ajouter le modèle à `agents.defaults.models`, ou
- Vider la liste blanche (supprimer `agents.defaults.models`), ou
- Choisir un modèle depuis `/model list`.

Exemple de configuration de liste blanche :

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## Changer de modèle en chat (`/model`)

Vous pouvez changer de modèle pour la session en cours sans redémarrer :

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Notes :

- `/model` (et `/model list`) est un sélecteur compact et numéroté (famille de modèle + fournisseurs disponibles).
- Sur Discord, `/model` et `/models` ouvrent un sélecteur interactif avec des menus déroulants de fournisseur et de modèle plus une étape de soumission.
- `/model <#>` sélectionne depuis ce sélecteur.
- `/model status` est la vue détaillée (candidats d'authentification et, quand configuré, `baseUrl` + mode `api` du point d'accès du fournisseur).
- Les références de modèle sont analysées en découpant au **premier** `/`. Utilisez `provider/model` quand vous tapez `/model <ref>`.
- Si l'identifiant du modèle lui-même contient `/` (style OpenRouter), vous devez inclure le préfixe du fournisseur (exemple : `/model openrouter/moonshotai/kimi-k2`).
- Si vous omettez le fournisseur, OpenClaw traite l'entrée comme un alias ou un modèle pour le **fournisseur par défaut** (fonctionne uniquement quand il n'y a pas de `/` dans l'identifiant du modèle).

Comportement/configuration complet des commandes : [Commandes slash](/tools/slash-commands).

## Commandes CLI

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (sans sous-commande) est un raccourci pour `models status`.

### `models list`

Affiche les modèles configurés par défaut. Options utiles :

- `--all` : catalogue complet
- `--local` : fournisseurs locaux uniquement
- `--provider <name>` : filtrer par fournisseur
- `--plain` : un modèle par ligne
- `--json` : sortie lisible par machine

### `models status`

Affiche le modèle principal résolu, les replis, le modèle d'image et un aperçu de l'authentification des fournisseurs configurés. Il affiche également le statut d'expiration OAuth pour les profils trouvés dans le magasin d'authentification (avertit dans les 24h par défaut). `--plain` affiche uniquement le modèle principal résolu.
Le statut OAuth est toujours affiché (et inclus dans la sortie `--json`). Si un fournisseur configuré n'a pas d'identifiants, `models status` affiche une section **Missing auth**. Le JSON inclut `auth.oauth` (fenêtre d'avertissement + profils) et `auth.providers` (authentification effective par fournisseur).
Utilisez `--check` pour l'automatisation (sortie `1` quand manquant/expiré, `2` quand en cours d'expiration).

L'authentification Anthropic préférée est le setup-token de la CLI Claude Code (exécutable n'importe où ; coller sur l'hôte du Gateway si nécessaire) :

```bash
claude setup-token
openclaw models status
```

## Scan (modèles gratuits OpenRouter)

`openclaw models scan` inspecte le **catalogue de modèles gratuits d'OpenRouter** et peut optionnellement sonder les modèles pour le support des outils et des images.

Options clés :

- `--no-probe` : ignorer les sondes en direct (métadonnées uniquement)
- `--min-params <b>` : taille minimale des paramètres (milliards)
- `--max-age-days <days>` : ignorer les modèles plus anciens
- `--provider <name>` : filtré par préfixe de fournisseur
- `--max-candidates <n>` : taille de la liste de repli
- `--set-default` : définir `agents.defaults.model.primary` sur la première sélection
- `--set-image` : définir `agents.defaults.imageModel.primary` sur la première sélection d'image

Le sondage nécessite une clé API OpenRouter (depuis les profils d'authentification ou `OPENROUTER_API_KEY`). Sans clé, utilisez `--no-probe` pour lister uniquement les candidats.

Les résultats de scan sont classés par :

1. Support des images
2. Latence des outils
3. Taille du contexte
4. Nombre de paramètres

Entrée

- Liste `/models` d'OpenRouter (filtré `:free`)
- Nécessite une clé API OpenRouter depuis les profils d'authentification ou `OPENROUTER_API_KEY` (voir [/environment](/help/environment))
- Filtrés optionnels : `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
- Contrôles de sonde : `--timeout`, `--concurrency`

Quand exécuté dans un TTY, vous pouvez sélectionner les replis interactivement. En mode non interactif, passez `--yes` pour accepter les valeurs par défaut.

## Registre de modèles (`models.json`)

Les fournisseurs personnalisés dans `models.providers` sont écrits dans `models.json` sous le répertoire de l'agent (par défaut `~/.openclaw/agents/<agentId>/models.json`). Ce fichier est fusionné par défaut sauf si `models.mode` est défini sur `replace`.

Precedence du mode de fusion pour les identifiants de fournisseur correspondants :

- Les `apiKey`/`baseUrl` non vides déjà présents dans le `models.json` de l'agent ont la priorité.
- Les `apiKey`/`baseUrl` vides ou manquants de l'agent se rabattent sur `models.providers` de la configuration.
- Les autres champs du fournisseur sont rafraichis a partir de la configuration et des donnees normalisees du catalogue.
