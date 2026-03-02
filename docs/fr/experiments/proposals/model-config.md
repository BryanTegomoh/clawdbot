---
summary: "Exploration : configuration de modèle, profils d'authentification et comportement de repli"
read_when:
  - Exploration des idées futures de sélection de modèle et de profil d'authentification
title: "Exploration de la configuration de modèle"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/experiments/proposals/model-config.md
  workflow: manual
---

# Configuration de modèle (exploration)

Ce document capture des **idées** pour la future configuration de modèle. Ce n'est pas une
spécification de livraison. Pour le comportement actuel, voir :

- [Modèles](/concepts/models)
- [Basculement de modèle](/concepts/model-failover)
- [OAuth + profils](/concepts/oauth)

## Motivation

Les opérateurs veulent :

- Plusieurs profils d'authentification par fournisseur (personnel vs professionnel).
- Une sélection `/model` simple avec des replis prévisibles.
- Une séparation claire entre les modèles texte et les modèles capables de traiter des images.

## Direction possible (haut niveau)

- Conserver la sélection de modèle simple : `provider/model` avec des alias optionnels.
- Permettre aux fournisseurs d'avoir plusieurs profils d'authentification, avec un ordre explicite.
- Utiliser une liste de repli globale pour que toutes les sessions basculent de manière cohérente.
- Ne surcharger le routage d'image que lorsque c'est explicitement configuré.

## Questions ouvertes

- La rotation de profil devrait-elle être par fournisseur ou par modèle ?
- Comment l'interface utilisateur devrait-elle présenter la sélection de profil pour une session ?
- Quel est le chemin de migration le plus sûr depuis les clés de configuration legacy ?
