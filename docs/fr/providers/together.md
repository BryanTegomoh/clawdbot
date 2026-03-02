---
summary: "Configuration de Together AI (authentification + sélection de modèle)"
read_when:
  - Vous souhaitez utiliser Together AI avec OpenClaw
  - Vous avez besoin de la variable d'environnement de clé API ou du choix d'authentification CLI
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/together.md
  workflow: manual
---

# Together AI

[Together AI](https://together.ai) fournit l'accès aux principaux modèles open source, notamment Llama, DeepSeek, Kimi, et plus, via une API unifiée.

- Fournisseur : `together`
- Authentification : `TOGETHER_API_KEY`
- API : Compatible OpenAI

## Démarrage rapide

1. Définissez la clé API (recommandé : stockez-la pour le Gateway) :

```bash
openclaw onboard --auth-choice together-api-key
```

2. Définissez un modèle par défaut :

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Exemple non interactif

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Cela définira `together/moonshotai/Kimi-K2.5` comme modèle par défaut.

## Note sur l'environnement

Si le Gateway s'exécute en tant que daemon (launchd/systemd), assurez-vous que `TOGETHER_API_KEY` est accessible par ce processus (par exemple, dans `~/.openclaw/.env` ou via `env.shellEnv`).

## Modèles disponibles

Together AI donne accès à de nombreux modèles open source populaires :

- **GLM 4.7 Fp8** - Modèle par défaut avec fenêtre de contexte de 200K
- **Llama 3.3 70B Instruct Turbo** - Suivi d'instructions rapide et efficace
- **Llama 4 Scout** - Modèle de vision avec compréhension d'images
- **Llama 4 Maverick** - Vision et raisonnement avancés
- **DeepSeek V3.1** - Modèle puissant de codage et de raisonnement
- **DeepSeek R1** - Modèle de raisonnement avancé
- **Kimi K2 Instruct** - Modèle haute performance avec fenêtre de contexte de 262K

Tous les modèles prennent en charge les complétions de chat standard et sont compatibles avec l'API OpenAI.
