---
read_when:
  - Comprendre ce qui se passe lors du premier lancement de l'agent
  - Savoir où se trouvent les fichiers d'initialisation
  - Déboguer la configuration de l'identité lors de la configuration initiale
summary: Rituel d'initialisation de l'agent qui prépare l'espace de travail et les fichiers d'identité
title: Initialisation de l'agent
sidebarTitle: Initialisation
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/bootstrapping.md
  workflow: manual
---

# Initialisation de l'agent

L'initialisation est le rituel de **premier lancement** qui prépare un espace de travail d'agent et
collecte les informations d'identité. Elle se produit après la configuration initiale, lorsque l'agent démarre
pour la première fois.

## Ce que fait l'initialisation

Lors du premier lancement de l'agent, OpenClaw initialisé l'espace de travail (par défaut
`~/.openclaw/workspace`) :

- Crée `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
- Lance un court rituel de questions-réponses (une question à la fois).
- Écrit l'identité et les préférences dans `IDENTITY.md`, `USER.md`, `SOUL.md`.
- Supprimé `BOOTSTRAP.md` une fois terminé pour qu'il ne s'exécute qu'une seule fois.

## Où elle s'exécute

L'initialisation s'exécute toujours sur l'**hôte du gateway**. Si l'application macOS se connecte à
un Gateway distant, l'espace de travail et les fichiers d'initialisation se trouvent sur cette machine
distante.

<Note>
Lorsque le Gateway s'exécute sur une autre machine, éditez les fichiers de l'espace de travail sur l'hôte
du gateway (par exemple, `user@gateway-host:~/.openclaw/workspace`).
</Note>

## Documentation associée

- Configuration initiale de l'application macOS : [Configuration initiale](/start/onboarding)
- Structure de l'espace de travail : [Espace de travail de l'agent](/concepts/agent-workspace)
