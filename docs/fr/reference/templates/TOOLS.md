---
title: "Modèle TOOLS.md"
summary: "Modèle d'espace de travail pour TOOLS.md"
read_when:
  - Initialisation manuelle d'un espace de travail
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/templates/TOOLS.md
  workflow: manual
---

# TOOLS.md - Notes locales

Les skills définissent _comment_ les outils fonctionnent. Ce fichier est pour _vos_ spécificités — ce qui est unique à votre configuration.

## Quoi mettre ici

Des choses comme :

- Noms et emplacements des caméras
- Hôtes et alias SSH
- Voix préférées pour le TTS
- Noms des enceintes/pièces
- Surnoms des appareils
- Tout ce qui est spécifique à l'environnement

## Exemples

```markdown
### Caméras

- living-room → Pièce principale, grand angle 180°
- front-door → Entrée, déclenchement par mouvement

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Voix préférée : "Nova" (chaleureuse, légèrement britannique)
- Enceinte par défaut : HomePod de la cuisine
```

## Pourquoi séparer ?

Les skills sont partagés. Votre configuration vous est propre. Les séparer signifie que vous pouvez mettre à jour les skills sans perdre vos notes, et partager les skills sans divulguer votre infrastructure.

---

Ajoutez tout ce qui vous aide à faire votre travail. C'est votre aide-mémoire.
