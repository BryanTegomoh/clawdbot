---
read_when:
  - Choix d'un chemin de configuration initiale
  - Installation dans un nouvel environnement
summary: Vue d'ensemble des options et flux de configuration initiale d'OpenClaw
title: Vue d'ensemble de la configuration initiale
sidebarTitle: Vue d'ensemble de la configuration initiale
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/onboarding-overview.md
  workflow: manual
---

# Vue d'ensemble de la configuration initiale

OpenClaw propose plusieurs chemins de configuration initiale selon l'emplacement du Gateway
et la manière dont vous préférez configurer les fournisseurs.

## Choisissez votre chemin de configuration initiale

- **Assistant CLI** pour macOS, Linux et Windows (via WSL2).
- **Application macOS** pour un premier lancement guidé sur Apple Silicon ou Intel Macs.

## Assistant de configuration initiale CLI

Lancez l'assistant dans un terminal :

```bash
openclaw onboard
```

Utilisez l'assistant CLI lorsque vous souhaitez un contrôle total sur le Gateway, l'espace de travail,
les canaux et les skills. Documentation :

- [Assistant de configuration initiale (CLI)](/start/wizard)
- [Commande `openclaw onboard`](/cli/onboard)

## Configuration initiale de l'application macOS

Utilisez l'application OpenClaw pour une installation entièrement guidée sur macOS. Documentation :

- [Configuration initiale (application macOS)](/start/onboarding)

## Fournisseur personnalisé

Si vous avez besoin d'un endpoint qui n'est pas listé, y compris les fournisseurs hébergés qui
exposent des API standard OpenAI ou Anthropic, choisissez **Fournisseur personnalisé** dans
l'assistant CLI. Il vous sera demandé de :

- Choisir compatible OpenAI, compatible Anthropic ou **Inconnu** (détection automatique).
- Saisir une URL de base et une clé API (si requis par le fournisseur).
- Fournir un identifiant de modèle et un alias optionnel.
- Choisir un identifiant d'endpoint pour que plusieurs endpoints personnalisés puissent coexister.

Pour les étapes détaillées, suivez la documentation de la configuration initiale CLI ci-dessus.
