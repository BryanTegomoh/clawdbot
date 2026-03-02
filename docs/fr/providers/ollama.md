---
summary: "Exécuter OpenClaw avec Ollama (runtime LLM local)"
read_when:
  - Vous souhaitez exécuter OpenClaw avec des modèles locaux via Ollama
  - Vous avez besoin de conseils d'installation et de configuration Ollama
title: "Ollama"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/ollama.md
  workflow: manual
---

# Ollama

Ollama est un runtime LLM local qui facilite l'exécution de modèles open source sur votre machine. OpenClaw s'intègre à l'API native d'Ollama (`/api/chat`), prenant en charge le streaming et l'appel d'outils, et peut **découvrir automatiquement les modèles compatibles avec les outils** lorsque vous activez `OLLAMA_API_KEY` (ou un profil d'authentification) et ne définissez pas d'entrée explicite `models.providers.ollama`.

## Démarrage rapide

1. Installez Ollama : [https://ollama.ai](https://ollama.ai)

2. Téléchargez un modèle :

```bash
ollama pull gpt-oss:20b
# ou
ollama pull llama3.3
# ou
ollama pull qwen2.5-coder:32b
# ou
ollama pull deepseek-r1:32b
```

3. Activez Ollama pour OpenClaw (n'importe quelle valeur fonctionne ; Ollama ne nécessite pas de vraie clé) :

```bash
# Définir la variable d'environnement
export OLLAMA_API_KEY="ollama-local"

# Ou configurer dans votre fichier de configuration
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Utilisez les modèles Ollama :

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Découverte de modèles (fournisseur implicite)

Lorsque vous définissez `OLLAMA_API_KEY` (ou un profil d'authentification) et **ne définissez pas** `models.providers.ollama`, OpenClaw découvre les modèles depuis l'instance Ollama locale à `http://127.0.0.1:11434` :

- Interroge `/api/tags` et `/api/show`
- Conserve uniquement les modèles qui signalent la capacité `tools`
- Marque `reasoning` lorsque le modèle signale `thinking`
- Lit `contextWindow` depuis `model_info["<arch>.context_length"]` lorsque disponible
- Définit `maxTokens` à 10 fois la fenêtre de contexte
- Définit tous les coûts à `0`

Cela évite les entrées de modèles manuelles tout en gardant le catalogue aligné avec les capacités d'Ollama.

Pour voir quels modèles sont disponibles :

```bash
ollama list
openclaw models list
```

Pour ajouter un nouveau modèle, téléchargez-le simplement avec Ollama :

```bash
ollama pull mistral
```

Le nouveau modèle sera automatiquement découvert et disponible à l'utilisation.

Si vous définissez `models.providers.ollama` explicitement, la découverte automatique est désactivée et vous devez définir les modèles manuellement (voir ci-dessous).

## Configuration

### Configuration basique (découverte implicite)

La manière la plus simple d'activer Ollama est via une variable d'environnement :

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Configuration explicite (modèles manuels)

Utilisez la configuration explicite lorsque :

- Ollama s'exécute sur un autre hôte/port.
- Vous souhaitez forcer des fenêtres de contexte ou des listes de modèles spécifiques.
- Vous souhaitez inclure des modèles qui ne signalent pas la prise en charge des outils.

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Si `OLLAMA_API_KEY` est défini, vous pouvez omettre `apiKey` dans l'entrée du fournisseur et OpenClaw le remplira pour les vérifications de disponibilité.

### URL de base personnalisée (configuration explicite)

Si Ollama s'exécute sur un hôte ou port différent (la configuration explicite désactive la découverte automatique, donc définissez les modèles manuellement) :

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434",
      },
    },
  },
}
```

### Sélection de modèle

Une fois configurés, tous vos modèles Ollama sont disponibles :

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Avancé

### Modèles de raisonnement

OpenClaw marque les modèles comme capables de raisonnement lorsqu'Ollama signale `thinking` dans `/api/show` :

```bash
ollama pull deepseek-r1:32b
```

### Coûts des modèles

Ollama est gratuit et s'exécute localement, donc tous les coûts de modèles sont fixés à 0 $.

### Configuration du streaming

L'intégration Ollama d'OpenClaw utilise l'**API native Ollama** (`/api/chat`) par défaut, qui prend entièrement en charge le streaming et l'appel d'outils simultanément. Aucune configuration spéciale n'est nécessaire.

#### Mode compatible OpenAI (ancien)

Si vous devez utiliser l'endpoint compatible OpenAI (par exemple derrière un proxy qui ne supporte que le format OpenAI), définissez `api: "openai-completions"` explicitement :

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Note : L'endpoint compatible OpenAI peut ne pas prendre en charge le streaming et l'appel d'outils simultanément. Vous devrez peut-être désactiver le streaming avec `params: { streaming: false }` dans la configuration du modèle.

### Fenêtres de contexte

Pour les modèles découverts automatiquement, OpenClaw utilise la fenêtre de contexte signalée par Ollama lorsqu'elle est disponible, sinon la valeur par défaut est `8192`. Vous pouvez surcharger `contextWindow` et `maxTokens` dans la configuration explicite du fournisseur.

## Dépannage

### Ollama non détecté

Assurez-vous qu'Ollama est en cours d'exécution et que vous avez défini `OLLAMA_API_KEY` (ou un profil d'authentification), et que vous n'avez **pas** défini d'entrée explicite `models.providers.ollama` :

```bash
ollama serve
```

Et que l'API est accessible :

```bash
curl http://localhost:11434/api/tags
```

### Aucun modèle disponible

OpenClaw ne découvre automatiquement que les modèles qui signalent la prise en charge des outils. Si votre modèle n'est pas listé, soit :

- Téléchargez un modèle compatible avec les outils, soit
- Définissez le modèle explicitement dans `models.providers.ollama`.

Pour ajouter des modèles :

```bash
ollama list  # Voir ce qui est installé
ollama pull gpt-oss:20b  # Télécharger un modèle compatible avec les outils
ollama pull llama3.3     # Ou un autre modèle
```

### Connexion refusée

Vérifiez qu'Ollama s'exécute sur le bon port :

```bash
# Vérifier si Ollama est en cours d'exécution
ps aux | grep ollama

# Ou redémarrer Ollama
ollama serve
```

## Voir aussi

- [Fournisseurs de modèles](/concepts/model-providers) - Aperçu de tous les fournisseurs
- [Sélection de modèle](/concepts/models) - Comment choisir les modèles
- [Configuration](/gateway/configuration) - Référence complète de configuration
