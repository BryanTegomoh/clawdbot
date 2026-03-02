---
summary: "Plugins communautaires : exigences de qualité, hébergément requis et processus de soumission par PR"
read_when:
  - Vous souhaitez publier un plugin tiers pour OpenClaw
  - Vous souhaitez proposer un plugin pour la liste de la documentation
title: "Plugins communautaires"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/plugins/community.md
  workflow: manual
---

# Plugins communautaires

Cette page repertorie les **plugins maintenus par la communauté** de haute qualité pour OpenClaw.

Nous acceptons les PR qui ajoutent des plugins communautaires ici lorsqu'ils respectent les exigences de qualité.

## Requis pour la publication

- Le package du plugin est publie sur npmjs (installable via `openclaw plugins install <npm-spec>`).
- Le code source est hébergé sur GitHub (dépôt public).
- Le dépôt inclut une documentation d'installation/utilisation et un système de suivi des problèmes.
- Le plugin présenté un signal de maintenance clair (mainteneur actif, mises a jour recentes ou gestion réactive des problèmes).

## Comment soumettre

Ouvrez une PR qui ajoute votre plugin a cette page avec :

- Nom du plugin
- Nom du package npm
- URL du dépôt GitHub
- Description en une ligne
- Commande d'installation

## Critères d'évaluation

Nous privilegions les plugins utiles, documentes et surs a utiliser.
Les wrappers sans effort, les projets sans propriétaire clair ou les packages non maintenus peuvent être refuses.

## Format de candidature

Utilisez ce format lors de l'ajout d'entrées :

- **Nom du plugin** — description courte
  npm: `@scope/package`
  repo: `https://github.com/org/repo`
  install: `openclaw plugins install @scope/package`

## Plugins repertories

- **WeChat** — Connectez OpenClaw a des comptes personnels WeChat via WeChatPadPro (protocole iPad). Prend en charge l'echange de texte, d'images et de fichiers avec des conversations declenchees par mots-clés.
  npm: `@icesword760/openclaw-wechat`
  repo: `https://github.com/icesword0760/openclaw-wechat`
  install: `openclaw plugins install @icesword760/openclaw-wechat`
