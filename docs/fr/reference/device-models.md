---
summary: "Comment OpenClaw intègre les identifiants de modèles d'appareils Apple pour des noms conviviaux dans l'application macOS."
read_when:
  - Mise à jour des correspondances d'identifiants de modèles d'appareils ou des fichiers NOTICE/licence
  - Modification de l'affichage des noms d'appareils dans l'interface Instances
title: "Base de données des modèles d'appareils"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/device-models.md
  workflow: manual
---

# Base de données des modèles d'appareils (noms conviviaux)

L'application macOS compagnon affiche des noms conviviaux pour les modèles Apple dans l'interface **Instances** en faisant correspondre les identifiants de modèles Apple (ex. `iPad16,6`, `Mac16,6`) à des noms lisibles par l'humain.

La correspondance est intégrée en JSON sous :

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## Source des données

Nous intégrons actuellement la correspondance depuis le dépôt sous licence MIT :

- `kyle-seongwoo-jun/apple-device-identifiers`

Pour garantir des builds déterministes, les fichiers JSON sont épinglés à des commits upstream spécifiques (enregistrés dans `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

## Mise à jour de la base de données

1. Choisissez les commits upstream auxquels vous souhaitez vous épingler (un pour iOS, un pour macOS).
2. Mettez à jour les hashes de commit dans `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3. Re-téléchargez les fichiers JSON, épinglés à ces commits :

```bash
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<commit sha for mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. Assurez-vous que `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` correspond toujours à l'upstream (remplacez-le si la licence upstream change).
5. Vérifiez que l'application macOS compile sans erreur (aucun avertissement) :

```bash
swift build --package-path apps/macos
```
