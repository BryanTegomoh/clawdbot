---
summary: "Utiliser Qwen OAuth (niveau gratuit) dans OpenClaw"
read_when:
  - Vous souhaitez utiliser Qwen avec OpenClaw
  - Vous voulez un accès OAuth gratuit à Qwen Coder
title: "Qwen"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/qwen.md
  workflow: manual
---

# Qwen

Qwen fournit un flux OAuth gratuit pour les modèles Qwen Coder et Qwen Vision (2 000 requêtes/jour, soumis aux limités de débit Qwen).

## Activer le plugin

```bash
openclaw plugins enable qwen-portal-auth
```

Redémarrez le Gateway après l'activation.

## S'authentifier

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Cela exécute le flux OAuth par code d'appareil Qwen et écrit une entrée de fournisseur dans votre `models.json` (plus un alias `qwen` pour un changement rapide).

## Identifiants de modèles

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

Changez de modèle avec :

```bash
openclaw models set qwen-portal/coder-model
```

## Réutiliser la connexion CLI Qwen Code

Si vous vous êtes déjà connecté avec le CLI Qwen Code, OpenClaw synchronisera les identifiants depuis `~/.qwen/oauth_creds.json` lors du chargement du magasin d'authentification. Vous avez toujours besoin d'une entrée `models.providers.qwen-portal` (utilisez la commande de connexion ci-dessus pour en créer une).

## Notes

- Les tokens se rafraîchissent automatiquement ; relancez la commande de connexion si le rafraîchissement échoue ou si l'accès est révoqué.
- URL de base par défaut : `https://portal.qwen.ai/v1` (surchargez avec `models.providers.qwen-portal.baseUrl` si Qwen fournit un endpoint différent).
- Voir [Fournisseurs de modèles](/concepts/model-providers) pour les règles applicables aux fournisseurs.
