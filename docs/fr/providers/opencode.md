---
summary: "Utiliser OpenCode Zen (modèles sélectionnés) avec OpenClaw"
read_when:
  - Vous souhaitez utiliser OpenCode Zen pour l'accès aux modèles
  - Vous cherchez une liste sélectionnée de modèles adaptés au codage
title: "OpenCode Zen"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/opencode.md
  workflow: manual
---

# OpenCode Zen

OpenCode Zen est une **liste sélectionnée de modèles** recommandés par l'équipe OpenCode pour les agents de codage. C'est un chemin d'accès optionnel et héberge qui utilise une clé API et le fournisseur `opencode`. Zen est actuellement en bêta.

## Configuration CLI

```bash
openclaw onboard --auth-choice opencode-zen
# ou en mode non interactif
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## Extrait de configuration

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## Notes

- `OPENCODE_ZEN_API_KEY` est également pris en charge.
- Vous vous connectez à Zen, ajoutez vos informations de facturation et copiez votre clé API.
- OpenCode Zen facture par requête ; consultez le tableau de bord OpenCode pour les détails.
