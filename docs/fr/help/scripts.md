---
summary: "Scripts du dépôt : objectif, portée et notes de sécurité"
read_when:
  - Exécution de scripts depuis le dépôt
  - Ajout ou modification de scripts sous ./scripts
title: "Scripts"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: help/scripts.md
  workflow: manual
---

# Scripts

Le répertoire `scripts/` contient des scripts utilitaires pour les flux de travail locaux et les tâches d'exploitation.
Utilisez-les lorsqu'une tâche est clairement liée à un script ; sinon, préférez le CLI.

## Conventions

- Les scripts sont **optionnels** sauf s'ils sont référencés dans la documentation ou les checklists de publication.
- Préférez les interfaces CLI lorsqu'elles existent (exemple : la surveillance d'authentification utilise `openclaw models status --check`).
- Considérez que les scripts sont spécifiques à la machine ; lisez-les avant de les exécuter sur une nouvelle machine.

## Scripts de surveillance d'authentification

Les scripts de surveillance d'authentification sont documentés ici :
[/automation/auth-monitoring](/automation/auth-monitoring)

## Lors de l'ajout de scripts

- Gardez les scripts ciblés et documentés.
- Ajoutez une courte entrée dans le document concerné (ou créez-en un si manquant).
