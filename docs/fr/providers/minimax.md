---
summary: "Utiliser MiniMax M2.1 dans OpenClaw"
read_when:
  - Vous souhaitez utiliser les modèles MiniMax dans OpenClaw
  - Vous avez besoin de conseils de configuration MiniMax
title: "MiniMax"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/minimax.md
  workflow: manual
---

# MiniMax

MiniMax est une entreprise d'IA qui développe la famille de modèles **M2/M2.1**. La version actuelle orientée codage est **MiniMax M2.1** (23 décembre 2025), conçue pour les tâches complexes du monde réel.

Source : [Note de version MiniMax M2.1](https://www.minimax.io/news/minimax-m21)

## Aperçu du modèle (M2.1)

MiniMax met en avant ces améliorations dans M2.1 :

- **Codage multilingue** renforcé (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
- Meilleur **développement web/app** et qualité de sortie esthétique (y compris le mobile natif).
- Amélioration de la gestion des **instructions composites** pour les flux de travail bureautiques, s'appuyant sur la réflexion entrelacée et l'exécution intégrée de contraintes.
- **Réponses plus concises** avec une consommation de tokens réduite et des boucles d'itération plus rapides.
- Meilleure compatibilité avec les **frameworks d'outils/agents** et gestion du contexte (Claude Code, Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
- Sorties de **dialogue et rédaction technique** de meilleure qualité.

## MiniMax M2.1 vs MiniMax M2.1 Lightning

- **Vitesse :** Lightning est la variante "rapide" dans la documentation tarifaire de MiniMax.
- **Coût :** La tarification montre le même coût en entrée, mais Lightning a un coût de sortie plus élevé.
- **Routage du plan codage :** Le backend Lightning n'est pas directement disponible sur le plan codage MiniMax. MiniMax achemine automatiquement la plupart des requêtes vers Lightning, mais revient au backend M2.1 standard lors des pics de trafic.

## Choisir une configuration

### MiniMax OAuth (Plan Codage) - recommandé

**Idéal pour :** configuration rapide avec le Plan Codage MiniMax via OAuth, sans clé API requise.

Activez le plugin OAuth intégré et authentifiez-vous :

```bash
openclaw plugins enable minimax-portal-auth  # ignorez si déjà chargé
openclaw gateway restart  # redémarrez si le gateway est déjà en cours d'exécution
openclaw onboard --auth-choice minimax-portal
```

Vous serez invité à sélectionner un endpoint :

- **Global** - Utilisateurs internationaux (`api.minimax.io`)
- **CN** - Utilisateurs en Chine (`api.minimaxi.com`)

Voir le [README du plugin MiniMax OAuth](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth) pour les détails.

### MiniMax M2.1 (Clé API)

**Idéal pour :** MiniMax hébergé avec une API compatible Anthropic.

Configurez via CLI :

- Exécutez `openclaw configuré`
- Sélectionnez **Model/auth**
- Choisissez **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.1 en repli (Opus principal)

**Idéal pour :** garder Opus 4.6 comme modèle principal, avec basculement vers MiniMax M2.1.

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
    },
  },
}
```

### Optionnel : Local via LM Studio (manuel)

**Idéal pour :** l'inférence locale avec LM Studio.
Nous avons observé de bons résultats avec MiniMax M2.1 sur du matériel puissant (par exemple un ordinateur de bureau/serveur) en utilisant le serveur local de LM Studio.

Configurez manuellement via `openclaw.json` :

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Configurer via `openclaw configuré`

Utilisez l'assistant de configuration interactif pour configurer MiniMax sans modifier le JSON :

1. Exécutez `openclaw configuré`.
2. Sélectionnez **Model/auth**.
3. Choisissez **MiniMax M2.1**.
4. Choisissez votre modèle par défaut lorsque vous y êtes invité.

## Options de configuration

- `models.providers.minimax.baseUrl` : préférez `https://api.minimax.io/anthropic` (compatible Anthropic) ; `https://api.minimax.io/v1` est optionnel pour les payloads compatibles OpenAI.
- `models.providers.minimax.api` : préférez `anthropic-messages` ; `openai-completions` est optionnel pour les payloads compatibles OpenAI.
- `models.providers.minimax.apiKey` : clé API MiniMax (`MINIMAX_API_KEY`).
- `models.providers.minimax.models` : définissez `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
- `agents.defaults.models` : créez des alias pour les modèles que vous souhaitez dans la liste autorisée.
- `models.mode` : conservez `merge` si vous souhaitez ajouter MiniMax en plus des modèles intégrés.

## Notes

- Les références de modèles sont `minimax/<model>`.
- API d'utilisation du Plan Codage : `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (nécessite une clé de plan codage).
- Mettez à jour les valeurs de tarification dans `models.json` si vous avez besoin d'un suivi précis des coûts.
- Lien de parrainage pour le Plan Codage MiniMax (10 % de réduction) : [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
- Voir [/concepts/model-providers](/concepts/model-providers) pour les règles des fournisseurs.
- Utilisez `openclaw models list` et `openclaw models set minimax/MiniMax-M2.1` pour changer.

## Dépannage

### "Unknown model: minimax/MiniMax-M2.1"

Cela signifie généralement que le **fournisseur MiniMax n'est pas configuré** (pas d'entrée de fournisseur et pas de profil d'authentification/clé d'environnement MiniMax trouvé). Un correctif pour cette détection est dans la version **2026.1.12** (non publiée au moment de la rédaction). Solution :

- Mettez à jour vers **2026.1.12** (ou exécutez depuis la source `main`), puis redémarrez le gateway.
- Exécutez `openclaw configuré` et sélectionnez **MiniMax M2.1**, ou
- Ajoutez le bloc `models.providers.minimax` manuellement, ou
- Définissez `MINIMAX_API_KEY` (ou un profil d'authentification MiniMax) pour que le fournisseur puisse être injecté.

Assurez-vous que l'identifiant du modèle respecte la **casse** :

- `minimax/MiniMax-M2.1`
- `minimax/MiniMax-M2.1-lightning`

Puis vérifiez avec :

```bash
openclaw models list
```
