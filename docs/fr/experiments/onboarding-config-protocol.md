---
summary: "Notes de protocole RPC pour l'assistant d'intégration et le schema de configuration"
read_when: "Modification des étapes de l'assistant d'intégration ou des endpoints de schema de configuration"
title: "Protocole d'intégration et de configuration"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: experiments/onboarding-config-protocol.md
  workflow: manual
---

# Protocole d'intégration + configuration

Objectif : surfaces d'intégration et de configuration partagees entre le CLI, l'app macOS et l'interface Web.

## Composants

- Moteur d'assistant (session partagee + invites + état d'intégration).
- L'intégration CLI utilisé le même flux d'assistant que les clients UI.
- Le Gateway RPC expose les endpoints d'assistant + schema de configuration.
- L'intégration macOS utilise le modèle d'étapes de l'assistant.
- L'interface Web affiche les formulaires de configuration a partir du JSON Schema + indications UI.

## Gateway RPC

- `wizard.start` paramètres : `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` paramètres : `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` paramètres : `{ sessionId }`
- `wizard.status` paramètres : `{ sessionId }`
- `config.schema` paramètres : `{}`

Réponses (forme)

- Assistant : `{ sessionId, done, step?, status?, error? }`
- Schema de configuration : `{ schema, uiHints, version, generatedAt }`

## Indications UI

- `uiHints` indexees par chemin ; métadonnées optionnelles (label/help/group/order/advanced/sensitive/placeholder).
- Les champs sensibles s'affichent comme des champs mot de passe ; pas de couche de masquage.
- Les nœuds de schema non supportes se replient sur l'editeur JSON brut.

## Notes

- Ce document est le seul endroit pour suivre les refactorisations de protocole pour l'intégration/configuration.
