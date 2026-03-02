---
summary: "États et animations de l'icone de la barre de menus pour OpenClaw sur macOS"
read_when:
  - Modification du comportement de l'icone de la barre de menus
title: "Icone de la barre de menus"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/icon.md
  workflow: manual
---

# États de l'icone de la barre de menus

Auteur : steipete · Mis a jour : 2025-12-06 · Portee : application macOS (`apps/macos`)

- **Inactif :** animation normale de l'icone (clignement, agitation occasionnelle).
- **En pause :** l'élément de statut utilisé `appearsDisabled` ; pas de mouvement.
- **Declencheur vocal (grandes oreilles) :** le detecteur voice wake appelle `AppState.triggerVoiceEars(ttl: nil)` lorsque le mot declencheur est entendu, maintenant `earBoostActive=true` pendant que l'enonce est capture. Les oreilles s'agrandissent (1,9x), obtiennent des trous d'oreille circulaires pour la lisibilite, puis retombent via `stopVoiceEars()` après 1s de silence. Déclenché uniquement depuis le pipeline vocal intégré a l'application.
- **En travail (agent actif) :** `AppState.isWorking=true` génère un micro-mouvement « queue/pattes agitees » : agitation des pattes plus rapide et leger decalage pendant que le travail est en cours. Actuellement activé autour des executions d'agent WebChat ; ajoutez le même bascule autour d'autres taches longues lorsque vous les connectez.

Points de connexion

- Voice wake : le runtime/testeur appelle `AppState.triggerVoiceEars(ttl: nil)` au déclenchement et `stopVoiceEars()` après 1s de silence pour correspondre a la fenêtre de capture.
- Activité de l'agent : définissez `AppStateStore.shared.setWorking(true/false)` autour des periodes de travail (déjà fait dans l'appel d'agent WebChat). Gardez les periodes courtes et reinitialisez dans des blocs `defer` pour eviter les animations bloquees.

Formes et tailles

- L'icone de base est dessinee dans `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
- L'echelle des oreilles est par défaut a `1.0` ; le boost vocal définit `earScale=1.9` et active `earHoles=true` sans changer le cadre global (image template 18x18 pt rendue dans un backing store Retina 36x36 px).
- L'agitation utilise l'agitation des pattes jusqu'a ~1.0 avec un petit tremblement horizontal ; c'est additif a toute agitation inactive existante.

Notes comportementales

- Pas de bascule CLI/broker externe pour les oreilles/travail ; gardez-le interne aux signaux propres de l'application pour eviter les battements accidentels.
- Gardez les TTL courts (<10s) pour que l'icone revienne rapidement a l'état de base si une tache se bloque.
