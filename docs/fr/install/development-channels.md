---
summary: "Canaux stable, beta et dev : sémantique, changement et étiquetage"
read_when:
  - Vous souhaitez basculer entre stable/beta/dev
  - Vous étiquetez ou publiez des pré-versions
title: "Canaux de développement"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/development-channels.md
  workflow: manual
---

# Canaux de développement

Dernière mise à jour : 2026-01-21

OpenClaw propose trois canaux de mise à jour :

- **stable** : dist-tag npm `latest`.
- **beta** : dist-tag npm `beta` (builds en cours de test).
- **dev** : head mobile de `main` (git). dist-tag npm : `dev` (quand publié).

Nous envoyons les builds vers **beta**, les testons, puis **promouvons un build validé vers `latest`**
sans changer le numéro de version. Les dist-tags sont la source de vérité pour les installations npm.

## Changer de canal

Checkout Git :

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

- `stable`/`beta` vérifient le dernier tag correspondant (souvent le même tag).
- `dev` bascule vers `main` et rebase sur l'upstream.

Installation globale npm/pnpm :

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

Cela met à jour via le dist-tag npm correspondant (`latest`, `beta`, `dev`).

Lorsque vous changez **explicitement** de canal avec `--channel`, OpenClaw aligne
aussi la méthode d'installation :

- `dev` assuré un checkout git (par défaut `~/openclaw`, remplaçable avec `OPENCLAW_GIT_DIR`),
  le met à jour et installe le CLI global depuis ce checkout.
- `stable`/`beta` installe depuis npm en utilisant le dist-tag correspondant.

Astuce : si vous voulez stable + dev en parallèle, gardez deux clones et pointez votre gateway vers celui en stable.

## Plugins et canaux

Lorsque vous changez de canal avec `openclaw update`, OpenClaw synchronise aussi les sources de plugins :

- `dev` préfère les plugins intégrés du checkout git.
- `stable` et `beta` restaurent les paquets de plugins installés via npm.

## Bonnes pratiques d'étiquetage

- Étiquetez les releases sur lesquelles vous voulez que les checkouts git se positionnent (`vYYYY.M.D` pour stable, `vYYYY.M.D-beta.N` pour beta).
- `vYYYY.M.D.beta.N` est aussi reconnu pour compatibilité, mais préférez `-beta.N`.
- Les anciens tags `vYYYY.M.D-<patch>` sont toujours reconnus comme stable (non-beta).
- Gardez les tags immuables : ne déplacez jamais et ne réutilisez jamais un tag.
- Les dist-tags npm restent la source de vérité pour les installations npm :
  - `latest` = stable
  - `beta` = build candidat
  - `dev` = snapshot de main (optionnel)

## Disponibilité de l'application macOS

Les builds beta et dev peuvent **ne pas** inclure de release d'application macOS. C'est normal :

- Le tag git et le dist-tag npm peuvent toujours être publiés.
- Mentionnez « pas de build macOS pour cette beta » dans les notes de version ou le changelog.
