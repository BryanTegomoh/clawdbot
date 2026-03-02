---
summary: "Utiliser les modèles Venice AI axés confidentialité dans OpenClaw"
read_when:
  - Vous souhaitez une inférence axée sur la confidentialité dans OpenClaw
  - Vous avez besoin de conseils de configuration Venice AI
title: "Venice AI"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/venice.md
  workflow: manual
---

# Venice AI (en vedette)

**Venice** est notre configuration Venice en vedette pour une inférence axée sur la confidentialité avec un accès anonymisé optionnel aux modèles propriétaires.

Venice AI fournit une inférence IA axée sur la confidentialité avec prise en charge de modèles non censurés et accès aux principaux modèles propriétaires via leur proxy anonymisé. Toute l'inférence est privée par défaut : pas d'entraînement sur vos données, pas de journalisation.

## Pourquoi Venice dans OpenClaw

- **Inférence privée** pour les modèles open source (pas de journalisation).
- **Modèles non censurés** quand vous en avez besoin.
- **Accès anonymisé** aux modèles propriétaires (Opus/GPT/Gemini) quand la qualité compte.
- Endpoints `/v1` compatibles OpenAI.

## Modes de confidentialité

Venice propose deux niveaux de confidentialité : comprendre cette distinction est essentiel pour choisir votre modèle.

| Mode           | Description                                                                                                               | Modèles                                        |
| -------------- | ------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Privé**      | Entièrement privé. Les prompts/réponses ne sont **jamais stockés ni journalisés**. Éphémère.                              | Llama, Qwen, DeepSeek, Venice Uncensored, etc. |
| **Anonymisé**  | Redirigé via un proxy Venice avec les métadonnées supprimées. Le fournisseur sous-jacent (OpenAI, Anthropic) voit des requêtes anonymisées. | Claude, GPT, Gemini, Grok, Kimi, MiniMax       |

## Fonctionnalités

- **Axé confidentialité** : Choisissez entre les modes "privé" (entièrement privé) et "anonymisé" (redirigé via un proxy)
- **Modèles non censurés** : Accès à des modèles sans restrictions de contenu
- **Accès aux grands modèles** : Utilisez Claude, GPT-5.2, Gemini, Grok via le proxy anonymisé de Venice
- **API compatible OpenAI** : Endpoints `/v1` standard pour une intégration facile
- **Streaming** : Pris en charge sur tous les modèles
- **Appel de fonctions** : Pris en charge sur certains modèles (vérifiez les capacités du modèle)
- **Vision** : Pris en charge sur les modèles avec capacité de vision
- **Pas de limités de débit strictes** : Un throttling d'usage raisonnable peut s'appliquer en cas d'utilisation extrême

## Configuration

### 1. Obtenir une clé API

1. Inscrivez-vous sur [venice.ai](https://venice.ai)
2. Allez dans **Settings > API Keys > Create new key**
3. Copiez votre clé API (format : `vapi_xxxxxxxxxxxx`)

### 2. Configurer OpenClaw

**Option A : Variable d'environnement**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**Option B : Configuration interactive (recommandée)**

```bash
openclaw onboard --auth-choice venice-api-key
```

Cela va :

1. Demander votre clé API (ou utiliser le `VENICE_API_KEY` existant)
2. Afficher tous les modèles Venice disponibles
3. Vous laisser choisir votre modèle par défaut
4. Configurer le fournisseur automatiquement

**Option C : Non interactif**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3. Vérifier la configuration

```bash
openclaw agent --model venice/llama-3.3-70b --message "Hello, are you working?"
```

## Sélection de modèle

Après la configuration, OpenClaw affiche tous les modèles Venice disponibles. Choisissez selon vos besoins :

- **Par défaut (notre choix)** : `venice/llama-3.3-70b` pour une performance privée et équilibrée.
- **Meilleure qualité globale** : `venice/claude-opus-45` pour les tâches complexes (Opus reste le plus performant).
- **Confidentialité** : Choisissez les modèles "privés" pour une inférence entièrement privée.
- **Capacité** : Choisissez les modèles "anonymisés" pour accéder à Claude, GPT, Gemini via le proxy de Venice.

Changez votre modèle par défaut à tout moment :

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

Listez tous les modèles disponibles :

```bash
openclaw models list | grep venice
```

## Configurer via `openclaw configuré`

1. Exécutez `openclaw configuré`
2. Sélectionnez **Model/auth**
3. Choisissez **Venice AI**

## Quel modèle utiliser ?

| Cas d'usage                       | Modèle recommandé                 | Pourquoi                                       |
| --------------------------------- | --------------------------------- | ----------------------------------------------- |
| **Chat général**                  | `llama-3.3-70b`                   | Polyvalent, entièrement privé                   |
| **Meilleure qualité globale**     | `claude-opus-45`                  | Opus reste le meilleur pour les tâches complexes |
| **Confidentialité + qualité Claude** | `claude-opus-45`               | Meilleur raisonnement via proxy anonymisé        |
| **Codage**                        | `qwen3-coder-480b-a35b-instruct` | Optimisé code, contexte 262k                    |
| **Tâches de vision**              | `qwen3-vl-235b-a22b`             | Meilleur modèle de vision privé                 |
| **Non censuré**                   | `venice-uncensored`               | Pas de restrictions de contenu                  |
| **Rapide + économique**           | `qwen3-4b`                        | Léger, toujours performant                      |
| **Raisonnement complexe**         | `deepseek-v3.2`                   | Raisonnement solide, privé                      |

## Modèles disponibles (25 au total)

### Modèles privés (15) : entièrement privés, sans journalisation

| ID du modèle                     | Nom                     | Contexte (tokens) | Caractéristiques        |
| -------------------------------- | ----------------------- | ------------------ | ----------------------- |
| `llama-3.3-70b`                  | Llama 3.3 70B           | 131k               | Général                 |
| `llama-3.2-3b`                   | Llama 3.2 3B            | 131k               | Rapide, léger           |
| `hermes-3-llama-3.1-405b`        | Hermes 3 Llama 3.1 405B | 131k               | Tâches complexes        |
| `qwen3-235b-a22b-thinking-2507`  | Qwen3 235B Thinking     | 131k               | Raisonnement            |
| `qwen3-235b-a22b-instruct-2507`  | Qwen3 235B Instruct     | 131k               | Général                 |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B        | 262k               | Code                    |
| `qwen3-next-80b`                 | Qwen3 Next 80B          | 262k               | Général                 |
| `qwen3-vl-235b-a22b`             | Qwen3 VL 235B           | 262k               | Vision                  |
| `qwen3-4b`                       | Venice Small (Qwen3 4B) | 32k                | Rapide, raisonnement    |
| `deepseek-v3.2`                  | DeepSeek V3.2           | 163k               | Raisonnement            |
| `venice-uncensored`              | Venice Uncensored       | 32k                | Non censuré             |
| `mistral-31-24b`                 | Venice Medium (Mistral) | 131k               | Vision                  |
| `google-gemma-3-27b-it`          | Gemma 3 27B Instruct    | 202k               | Vision                  |
| `openai-gpt-oss-120b`            | OpenAI GPT OSS 120B     | 131k               | Général                 |
| `zai-org-glm-4.7`                | GLM 4.7                 | 202k               | Raisonnement, multilingue |

### Modèles anonymisés (10) : via le proxy Venice

| ID du modèle             | Original          | Contexte (tokens) | Caractéristiques  |
| ------------------------ | ----------------- | ------------------ | ----------------- |
| `claude-opus-45`         | Claude Opus 4.5   | 202k               | Raisonnement, vision |
| `claude-sonnet-45`       | Claude Sonnet 4.5 | 202k               | Raisonnement, vision |
| `openai-gpt-52`          | GPT-5.2           | 262k               | Raisonnement      |
| `openai-gpt-52-codex`    | GPT-5.2 Codex     | 262k               | Raisonnement, vision |
| `gemini-3-pro-preview`   | Gemini 3 Pro      | 202k               | Raisonnement, vision |
| `gemini-3-flash-preview` | Gemini 3 Flash    | 262k               | Raisonnement, vision |
| `grok-41-fast`           | Grok 4.1 Fast     | 262k               | Raisonnement, vision |
| `grok-code-fast-1`       | Grok Code Fast 1  | 262k               | Raisonnement, code |
| `kimi-k2-thinking`       | Kimi K2 Thinking  | 262k               | Raisonnement      |
| `minimax-m21`            | MiniMax M2.1      | 202k               | Raisonnement      |

## Découverte de modèles

OpenClaw découvre automatiquement les modèles depuis l'API Venice lorsque `VENICE_API_KEY` est défini. Si l'API est inaccessible, il revient à un catalogue statique.

L'endpoint `/models` est public (pas d'authentification nécessaire pour la liste), mais l'inférence nécessite une clé API valide.

## Prise en charge du streaming et des outils

| Fonctionnalité       | Prise en charge                                         |
| -------------------- | ------------------------------------------------------- |
| **Streaming**        | Tous les modèles                                        |
| **Appel de fonctions** | La plupart des modèles (vérifiez `supportsFunctionCalling` dans l'API) |
| **Vision/Images**    | Modèles marqués avec la fonctionnalité "Vision"         |
| **Mode JSON**        | Pris en charge via `response_format`                    |

## Tarification

Venice utilise un système de crédits. Consultez [venice.ai/pricing](https://venice.ai/pricing) pour les tarifs actuels :

- **Modèles privés** : Coût généralement inférieur
- **Modèles anonymisés** : Similaire au prix direct de l'API + un petit supplément Venice

## Comparaison : Venice vs API directe

| Aspect             | Venice (Anonymisé)                 | API directe          |
| ------------------ | ---------------------------------- | -------------------- |
| **Confidentialité** | Métadonnées supprimées, anonymisé | Votre compte lié     |
| **Latence**        | +10-50ms (proxy)                   | Direct               |
| **Fonctionnalités** | La plupart des fonctionnalités prises en charge | Toutes les fonctionnalités |
| **Facturation**    | Crédits Venice                     | Facturation fournisseur |

## Exemples d'utilisation

```bash
# Utiliser le modèle privé par défaut
openclaw agent --model venice/llama-3.3-70b --message "Quick health check"

# Utiliser Claude via Venice (anonymisé)
openclaw agent --model venice/claude-opus-45 --message "Summarize this task"

# Utiliser le modèle non censuré
openclaw agent --model venice/venice-uncensored --message "Draft options"

# Utiliser le modèle de vision avec image
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# Utiliser le modèle de codage
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## Dépannage

### Clé API non reconnue

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

Assurez-vous que la clé commence par `vapi_`.

### Modèle non disponible

Le catalogue de modèles Venice se met à jour dynamiquement. Exécutez `openclaw models list` pour voir les modèles actuellement disponibles. Certains modèles peuvent être temporairement hors ligne.

### Problèmes de connexion

L'API Venice est à `https://api.venice.ai/api/v1`. Assurez-vous que votre réseau autorisé les connexions HTTPS.

## Exemple de fichier de configuration

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Liens

- [Venice AI](https://venice.ai)
- [Documentation API](https://docs.venice.ai)
- [Tarification](https://venice.ai/pricing)
- [Statut](https://status.venice.ai)
