---
summary: "Utiliser l'API unifiée de Qianfan pour accéder à de nombreux modèles dans OpenClaw"
read_when:
  - Vous souhaitez une seule clé API pour de nombreux LLMs
  - Vous avez besoin de conseils de configuration Baidu Qianfan
title: "Qianfan"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/qianfan.md
  workflow: manual
---

# Guide du fournisseur Qianfan

Qianfan est la plateforme MaaS de Baidu. Elle fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles derrière un seul endpoint et une seule clé API. Elle est compatible OpenAI, donc la plupart des SDK OpenAI fonctionnent en changeant l'URL de base.

## Prérequis

1. Un compte Baidu Cloud avec accès à l'API Qianfan
2. Une clé API depuis la console Qianfan
3. OpenClaw installé sur votre système

## Obtenir votre clé API

1. Rendez-vous sur la [Console Qianfan](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2. Créez une nouvelle application ou sélectionnez-en une existante
3. Générez une clé API (format : `bce-v3/ALTAK-...`)
4. Copiez la clé API pour l'utiliser avec OpenClaw

## Configuration CLI

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## Documentation associée

- [Configuration OpenClaw](/gateway/configuration)
- [Fournisseurs de modèles](/concepts/model-providers)
- [Configuration d'agent](/concepts/agent)
- [Documentation de l'API Qianfan](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)
