---
summary: "Étapes de signature pour les builds de debogage macOS générés par les scripts d'empaquetage"
read_when:
  - Compilation ou signature de builds de debogage mac
title: "Signature macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/signing.md
  workflow: manual
---

# Signature mac (builds de debogage)

Cette application est généralement compilee depuis [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), qui maintenant :

- définit un identifiant de bundle de debogage stable : `ai.openclaw.mac.debug`
- écrit le Info.plist avec ce bundle id (surcharge via `BUNDLE_ID=...`)
- appelle [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) pour signer le binaire principal et le bundle d'application afin que macOS traite chaque recompilation comme le même bundle signe et conserve les permissions TCC (notifications, accessibilité, enregistrement d'ecran, micro, parole). Pour des permissions stables, utilisez une vraie identité de signature ; la signature ad-hoc est optionnelle et fragile (voir [Permissions macOS](/platforms/mac/permissions)).
- utilisé `CODESIGN_TIMESTAMP=auto` par défaut ; cela active les horodatages de confiance pour les signatures Developer ID. Définissez `CODESIGN_TIMESTAMP=off` pour ignorer l'horodatage (builds de debogage hors ligne).
- injecte les metadonnées de build dans Info.plist : `OpenClawBuildTimestamp` (UTC) et `OpenClawGitCommit` (hash court) pour que le panneau About puisse afficher le build, git, et le canal debug/release.
- **L'empaquetage nécessite Node 22+** : le script exécute des builds TS et le build de l'Interface de contrôle.
- lit `SIGN_IDENTITY` depuis l'environnement. Ajoutez `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (ou votre certificat Developer ID Application) a votre shell rc pour toujours signer avec votre certificat. La signature ad-hoc nécessite un opt-in explicite via `ALLOW_ADHOC_SIGNING=1` ou `SIGN_IDENTITY="-"` (non recommandé pour le test de permissions).
- exécute un audit de Team ID après la signature et échoue si un Mach-O dans le bundle d'application est signe par un Team ID different. Définissez `SKIP_TEAM_ID_CHECK=1` pour contourner.

## Utilisation

```bash
# depuis la racine du repo
scripts/package-mac-app.sh               # selectionne automatiquement l'identite ; erreur si aucune trouvee
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # vrai certificat
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (les permissions ne persisteront pas)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # ad-hoc explicite (meme mise en garde)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # contournement dev-only pour incompatibilite Team ID Sparkle
```

### Note sur la signature ad-hoc

Lors de la signature avec `SIGN_IDENTITY="-"` (ad-hoc), le script désactive automatiquement le **Hardened Runtime** (`--options runtime`). C'est nécessaire pour eviter les crashs lorsque l'application tente de charger des frameworks intégrés (comme Sparkle) qui ne partagent pas le même Team ID. Les signatures ad-hoc cassent également la persistance des permissions TCC ; voir [Permissions macOS](/platforms/mac/permissions) pour les étapes de récupération.

## Metadonnées de build pour About

`package-mac-app.sh` estampille le bundle avec :

- `OpenClawBuildTimestamp` : ISO8601 UTC au moment de l'empaquetage
- `OpenClawGitCommit` : hash git court (ou `unknown` si indisponible)

L'onglet About lit ces clés pour afficher la version, la date de build, le commit git, et si c'est un build de debogage (via `#if DEBUG`). Lancez l'empaqueteur pour actualiser ces valeurs après des modifications de code.

## Pourquoi

Les permissions TCC sont liées a l'identifiant de bundle _et_ a la signature de code. Les builds de debogage non signes avec des UUID changeants faisaient que macOS oubliait les octrois après chaque recompilation. Signer les binaires (ad-hoc par défaut) et garder un bundle id/chemin fixe (`dist/OpenClaw.app`) preserve les octrois entre les builds, correspondant a l'approche VibeTunnel.
