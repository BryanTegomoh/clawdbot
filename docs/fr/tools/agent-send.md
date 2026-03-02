---
summary: "Exécutions directes via `openclaw agent` en CLI (avec livraison optionnelle)"
read_when:
  - Ajout ou modification du point d'entrée CLI de l'agent
title: "Agent Send"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/agent-send.md
  workflow: manual
---

# `openclaw agent` (exécutions directes de l'agent)

`openclaw agent` exécute un seul tour d'agent sans avoir besoin d'un message de chat entrant.
Par défaut, il passe **par le Gateway** ; ajoutez `--local` pour forcer le runtime
embarqué sur la machine actuelle.

## Comportement

- Obligatoire : `--message <texte>`
- Sélection de session :
  - `--to <dest>` dérive la clé de session (les cibles groupe/canal préservent l'isolation ; les chats directs se réduisent à `main`), **ou**
  - `--session-id <id>` réutilise une session existante par id, **ou**
  - `--agent <id>` cible un agent configuré directement (utilisé la clé de session `main` de cet agent)
- Exécute le même runtime d'agent embarqué que les réponses entrantes normales.
- Les drapeaux thinking/verbose persistent dans le magasin de sessions.
- Sortie :
  - par défaut : affiche le texte de réponse (plus les lignes `MEDIA:<url>`)
  - `--json` : affiche le payload structuré + les métadonnées
- Livraison optionnelle vers un canal avec `--deliver` + `--channel` (les formats cibles correspondent à `openclaw message --target`).
- Utilisez `--reply-channel`/`--reply-to`/`--reply-account` pour surcharger la livraison sans changer la session.

Si le Gateway est injoignable, le CLI **bascule** vers l'exécution locale embarquée.

## Exemples

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Drapeaux

- `--local` : exécuter localement (nécessite les clés API du fournisseur de modèles dans votre shell)
- `--deliver` : envoyer la réponse au canal choisi
- `--channel` : canal de livraison (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, défaut : `whatsapp`)
- `--reply-to` : surcharge de la cible de livraison
- `--reply-channel` : surcharge du canal de livraison
- `--reply-account` : surcharge de l'id du compte de livraison
- `--thinking <off|minimal|low|medium|high|xhigh>` : persister le niveau de réflexion (modèles GPT-5.2 + Codex uniquement)
- `--verbose <on|full|off>` : persister le niveau verbose
- `--timeout <secondes>` : surcharger le timeout de l'agent
- `--json` : sortie JSON structurée
