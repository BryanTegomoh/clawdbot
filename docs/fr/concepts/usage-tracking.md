---
summary: "Surfaces de suivi de l'utilisation et exigences d'authentification"
read_when:
  - Vous connectez les surfaces d'utilisation/quota des fournisseurs
  - Vous devez expliquer le comportement du suivi d'utilisation ou les exigences d'authentification
title: "Suivi de l'utilisation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/usage-tracking.md
  workflow: manual
---

# Suivi de l'utilisation

## De quoi il s'agit

- Récupère l'utilisation/quota du fournisseur directement depuis ses points de terminaison d'utilisation.
- Pas de coûts estimés ; uniquement les fenêtres rapportees par le fournisseur.

## Ou cela s'affiche

- `/status` dans les chats : carte de statut riche en emojis avec les tokens de session + coût estime (clé API uniquement). L'utilisation du fournisseur s'affiche pour le **fournisseur du modèle actuel** lorsque disponible.
- `/usage off|tokens|full` dans les chats : pied de page d'utilisation par réponse (OAuth affiche uniquement les tokens).
- `/usage cost` dans les chats : résumé des coûts locaux agrege a partir des journaux de session OpenClaw.
- CLI : `openclaw status --usage` affiche un detail complet par fournisseur.
- CLI : `openclaw channels list` affiche le même aperçu d'utilisation a côté de la configuration du fournisseur (utilisez `--no-usage` pour l'ignorer).
- Barre de menu macOS : section « Utilisation » sous Contexte (uniquement si disponible).

## Fournisseurs + identifiants

- **Anthropic (Claude)** : tokens OAuth dans les profils d'authentification.
- **GitHub Copilot** : tokens OAuth dans les profils d'authentification.
- **Gemini CLI** : tokens OAuth dans les profils d'authentification.
- **Antigravity** : tokens OAuth dans les profils d'authentification.
- **OpenAI Codex** : tokens OAuth dans les profils d'authentification (accountId utilisé lorsque présent).
- **MiniMax** : clé API (clé de plan coding ; `MINIMAX_CODE_PLAN_KEY` ou `MINIMAX_API_KEY`) ; utilisé la fenêtre de plan coding de 5 heures.
- **z.ai** : clé API via env/config/magasin d'authentification.

L'utilisation est masquee si aucun identifiant OAuth/API correspondant n'existe.
