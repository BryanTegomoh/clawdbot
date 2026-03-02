---
summary: "Référence CLI pour `openclaw onboard` (assistant d'intégration interactif)"
read_when:
  - Vous souhaitez une configuration guidée pour le Gateway, l'espace de travail, l'authentification, les canaux et les compétences
title: "onboard"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/onboard.md
  workflow: manual
---

# `openclaw onboard`

Assistant d'intégration interactif (configuration Gateway local ou distant).

## Guides liés

- Hub d'intégration CLI : [Onboarding Wizard (CLI)](/start/wizard)
- Aperçu de l'intégration : [Onboarding Overview](/start/onboarding-overview)
- Référence d'intégration CLI : [CLI Onboarding Référence](/start/wizard-cli-reference)
- Automatisation CLI : [CLI Automation](/start/wizard-cli-automation)
- Intégration macOS : [Onboarding (macOS App)](/start/onboarding)

## Exemples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Fournisseur personnalisé non interactif :

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` est optionnel en mode non interactif. Si omis, l'intégration vérifie `CUSTOM_API_KEY`.

Stocker les clés de fournisseur comme refs au lieu de clair :

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

Avec `--secret-input-mode ref`, l'intégration écrit des refs env au lieu de valeurs de clé en clair.
Pour les fournisseurs basés sur un profil d'authentification, cela écrit des entrées `keyRef` ; pour les fournisseurs personnalisés, cela écrit `models.providers.<id>.apiKey` comme ref env (par exemple `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`).

Contrat du mode `ref` non interactif :

- Définissez la variable d'environnement du fournisseur dans l'environnement du processus d'intégration (par exemple `OPENAI_API_KEY`).
- Ne passez pas de flags de clé en ligne (par exemple `--openai-api-key`) sauf si cette variable d'environnement est aussi définie.
- Si un flag de clé en ligne est passé sans la variable d'environnement requise, l'intégration échoue immédiatement avec des instructions.

Comportement de l'intégration interactive avec le mode référence :

- Choisissez **Use secret référence** lorsque vous y êtes invite.
- Puis choisissez soit :
  - Variable d'environnement
  - Fournisseur de secrets configuré (`file` ou `exec`)
- L'intégration effectué une validation preflight rapide avant de sauvegarder la ref.
  - Si la validation échoue, l'intégration affiche l'erreur et vous permet de réessayer.

Choix d'endpoint Z.AI non interactifs :

Note : `--auth-choice zai-api-key` détecte automatiquement le meilleur endpoint Z.AI pour votre clé (préfère l'API générale avec `zai/glm-5`).
Si vous souhaitez spécifiquement les endpoints GLM Coding Plan, choisissez `zai-coding-global` ou `zai-coding-cn`.

```bash
# Sélection d'endpoint sans invite
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Autres choix d'endpoint Z.AI :
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Exemple Mistral non interactif :

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

Notes sur les flux :

- `quickstart` : invites minimales, génère automatiquement un jeton Gateway.
- `manual` : invites complêtes pour port/liaison/authentification (alias de `advanced`).
- Comportement de portée DM de l'intégration locale : [CLI Onboarding Référence](/start/wizard-cli-reference#outputs-and-internals).
- Premier chat le plus rapide : `openclaw dashboard` (interface de contrôle, sans configuration de canal).
- Fournisseur personnalisé : connectez tout endpoint compatible OpenAI ou Anthropic, y compris les fournisseurs hébergés non listes. Utilisez Unknown pour la détection automatique.

## Commandes de suivi courantes

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` n'implique pas le mode non interactif. Utilisez `--non-interactive` pour les scripts.
</Note>
