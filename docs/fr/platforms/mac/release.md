---
summary: "Checklist de publication macOS OpenClaw (flux Sparkle, empaquetage, signature)"
read_when:
  - Realisation ou validation d'une publication macOS OpenClaw
  - Mise a jour de l'appcast Sparkle ou des ressources du flux
title: "Publication macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/release.md
  workflow: manual
---

# Publication macOS OpenClaw (Sparkle)

Cette application inclut maintenant les mises a jour automatiques Sparkle. Les builds de publication doivent être signes Developer ID, compresses en zip et publies avec une entrée appcast signee.

## Prerequis

- Certificat Developer ID Application installe (exemple : `Developer ID Application: <Developer Name> (<TEAMID>)`).
- Chemin de la clé privée Sparkle défini dans l'environnement comme `SPARKLE_PRIVATE_KEY_FILE` (chemin vers votre clé privée ed25519 Sparkle ; la clé publique est intégrée dans Info.plist). Si elle est manquante, verifiez `~/.profile`.
- Identifiants de notarisation (profil Keychain ou clé API) pour `xcrun notarytool` si vous voulez une distribution DMG/zip compatible Gatekeeper.
  - Nous utilisons un profil Keychain nomme `openclaw-notary`, créé a partir des variables d'environnement de clé API App Store Connect dans votre profil shell :
    - `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
    - `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
    - `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
- Dependances `pnpm` installees (`pnpm install --config.node-linker=hoisted`).
- Les outils Sparkle sont récupérés automatiquement via SwiftPM dans `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, etc.).

## Compilation et empaquetage

Notes :

- `APP_BUILD` correspond a `CFBundleVersion`/`sparkle:version` ; gardez-le numérique + monotone (pas de `-beta`), sinon Sparkle le compare comme egal.
- Par défaut, l'architecture courante (`$(uname -m)`). Pour les builds de publication/universels, définissez `BUILD_ARCHS="arm64 x86_64"` (ou `BUILD_ARCHS=all`).
- Utilisez `scripts/package-mac-dist.sh` pour les artefacts de publication (zip + DMG + notarisation). Utilisez `scripts/package-mac-app.sh` pour l'empaquetage local/dev.

```bash
# Depuis la racine du repo ; definissez les IDs de publication pour que le flux Sparkle soit active.
# APP_BUILD doit etre numerique + monotone pour la comparaison Sparkle.
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.2.26 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# Zip pour la distribution (inclut les resource forks pour le support delta Sparkle)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.2.26.zip

# Optionnel : construire egalement un DMG style pour les humains (glisser vers /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.2.26.dmg

# Recommandé : compiler + notariser/agrafer zip + DMG
# D'abord, creez un profil Keychain une fois :
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.2.26 \
APP_BUILD="$(git rev-list --count HEAD)" \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# Optionnel : livrer les dSYM avec la publication
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.2.26.dSYM.zip
```

## Entrée appcast

Utilisez le générateur de notes de publication pour que Sparkle affiche des notes HTML formatees :

```bash
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.2.26.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

Génère des notes de publication HTML a partir de `CHANGELOG.md` (via [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) et les intègre dans l'entrée appcast.
Committez le `appcast.xml` mis a jour avec les ressources de publication (zip + dSYM) lors de la publication.

## Publier et vérifier

- Téléchargez `OpenClaw-2026.2.26.zip` (et `OpenClaw-2026.2.26.dSYM.zip`) vers la release GitHub pour le tag `v2026.2.26`.
- Assurez-vous que l'URL brute de l'appcast correspond au flux intégré : `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
- Vérifications :
  - `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` retourne 200.
  - `curl -I <enclosure url>` retourne 200 après le téléchargement des ressources.
  - Sur un build public précédent, lancez « Check for Updates... » depuis l'onglet About et verifiez que Sparkle installe le nouveau build correctement.

Définition de termine : application signee + appcast sont publies, le flux de mise a jour fonctionne depuis une version installee plus ancienne, et les ressources de publication sont attachees a la release GitHub.
