---
summary: "Connexions manuelles pour l'automatisation du navigateur + publication sur X/Twitter"
read_when:
  - Vous devez vous connecter à des sites pour l'automatisation du navigateur
  - Vous souhaitez publier des mises à jour sur X/Twitter
title: "Connexion navigateur"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/browser-login.md
  workflow: manual
---

# Connexion navigateur + publication X/Twitter

## Connexion manuelle (recommandé)

Lorsqu'un site nécessite une connexion, **connectez-vous manuellement** dans le profil de navigateur **hôte** (le navigateur openclaw).

Ne donnez **pas** vos identifiants au modèle. Les connexions automatisées déclenchent souvent des défenses anti-bot et peuvent verrouiller le compte.

Retour à la documentation principale du navigateur : [Navigateur](/tools/browser).

## Quel profil Chrome est utilisé ?

OpenClaw contrôle un **profil Chrome dédié** (nommé `openclaw`, interface teintée en orange). Celui-ci est séparé de votre profil de navigateur quotidien.

Deux moyens simples d'y accéder :

1. **Demandez à l'agent d'ouvrir le navigateur** puis connectez-vous vous-même.
2. **Ouvrez-le via le CLI** :

```bash
openclaw browser start
openclaw browser open https://x.com
```

Si vous avez plusieurs profils, passez `--browser-profile <nom>` (la valeur par défaut est `openclaw`).

## X/Twitter : flux recommandé

- **Lire/rechercher/fils** : utilisez le navigateur **hôte** (connexion manuelle).
- **Publier des mises à jour** : utilisez le navigateur **hôte** (connexion manuelle).

## Sandboxing + accès au navigateur hôte

Les sessions de navigateur sandboxées sont **plus susceptibles** de déclencher la détection de bots. Pour X/Twitter (et autres sites stricts), préférez le navigateur **hôte**.

Si l'agent est sandboxé, l'outil navigateur utilisé par défaut le bac à sable. Pour autoriser le contrôle de l'hôte :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Puis ciblez le navigateur hôte :

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Ou désactivez le sandboxing pour l'agent qui publie les mises à jour.
