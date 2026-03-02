---
title: "Créer des Skills"
summary: "Construire et tester des Skills personnalisés dans votre workspace avec SKILL.md"
read_when:
  - Vous créez un nouveau skill personnalisé dans votre workspace
  - Vous avez besoin d'un workflow de démarrage rapide pour les Skills basés sur SKILL.md
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/creating-skills.md
  workflow: manual
---

# Créer des Skills personnalisés 🛠

OpenClaw est conçu pour être facilement extensible. Les « Skills » sont le principal moyen d'ajouter de nouvelles capacités à votre assistant.

## Qu'est-ce qu'un Skill ?

Un Skill est un répertoire contenant un fichier `SKILL.md` (qui fournit des instructions et des définitions d'outils au LLM) et optionnellement des scripts ou ressources.

## Étape par étape : votre premier Skill

### 1. Créer le répertoire

Les Skills se trouvent dans votre workspace, généralement `~/.openclaw/workspace/skills/`. Créez un nouveau dossier pour votre Skill :

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. Définir le `SKILL.md`

Créez un fichier `SKILL.md` dans ce répertoire. Ce fichier utilisé un frontmatter YAML pour les métadonnées et du Markdown pour les instructions.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Ajouter des outils (optionnel)

Vous pouvez définir des outils personnalisés dans le frontmatter ou demander à l'agent d'utiliser des outils système existants (comme `bash` ou `browser`).

### 4. Rafraîchir OpenClaw

Demandez à votre agent de « rafraîchir les skills » ou redémarrez le Gateway. OpenClaw découvrira le nouveau répertoire et indexera le `SKILL.md`.

## Bonnes pratiques

- **Soyez concis** : donnez au modèle des instructions sur _ce qu'il doit faire_, pas sur comment être une IA.
- **Sécurité d'abord** : si votre Skill utilisé `bash`, assurez-vous que les prompts ne permettent pas l'injection de commandes arbitraires depuis des entrées utilisateur non fiables.
- **Testez localement** : utilisez `openclaw agent --message "use my new skill"` pour tester.

## Skills partagés

Vous pouvez également parcourir et contribuer des Skills sur [ClawHub](https://clawhub.com).
