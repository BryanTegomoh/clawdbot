---
summary: "Journalisation OpenClaw : fichier de diagnostics rotatif + drapeaux de confidentialité du journal unifie"
read_when:
  - Capture des logs macOS ou investigation sur la journalisation de données privées
  - Debogage de problèmes de voice wake/cycle de vie de session
title: "Journalisation macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/logging.md
  workflow: manual
---

# Journalisation (macOS)

## Fichier de diagnostics rotatif (panneau Debug)

OpenClaw route les logs de l'application macOS via swift-log (journalisation unifiee par défaut) et peut écrire un journal local rotatif sur disque lorsque vous avez besoin d'une capture durable.

- Verbosite : **Panneau Debug → Logs → App logging → Verbosity**
- Activer : **Panneau Debug → Logs → App logging → « Write rolling diagnostics log (JSONL) »**
- Emplacement : `~/Library/Logs/OpenClaw/diagnostics.jsonl` (rotation automatique ; les anciens fichiers sont suffixes avec `.1`, `.2`, ...)
- Effacer : **Panneau Debug → Logs → App logging → « Clear »**

Notes :

- C'est **désactivé par défaut**. N'activez que pendant un debogage actif.
- Traitez le fichier comme sensible ; ne le partagez pas sans examen prealable.

## Donnees privées de la journalisation unifiee sur macOS

La journalisation unifiee masque la plupart des charges utiles sauf si un sous-système opte pour `privacy -off`. Selon l'article de Peter sur les [particularites de confidentialité de la journalisation](https://steipete.me/posts/2025/logging-privacy-shenanigans) macOS (2025), cela est contrôle par un plist dans `/Library/Préférences/Logging/Subsystems/` indexe par le nom du sous-système. Seules les nouvelles entrées de log prennent en compte le drapeau, activez-le donc avant de reproduire un problème.

## Activer pour OpenClaw (`ai.openclaw`)

- Ecrivez le plist dans un fichier temporaire d'abord, puis installez-le de maniere atomique en tant que root :

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

- Aucun redémarrage n'est requis ; logd détecté le fichier rapidement, mais seules les nouvelles lignes de log incluront les charges utiles privées.
- Consultez la sortie enrichie avec le helper existant, par exemple `./scripts/clawlog.sh --category WebChat --last 5m`.

## Désactiver après le debogage

- Supprimez la surcharge : `sudo rm /Library/Préférences/Logging/Subsystems/ai.openclaw.plist`.
- Exécutez optionnellement `sudo log config --reload` pour forcer logd a abandonner la surcharge immédiatement.
- N'oubliez pas que cette surface peut inclure des numéros de téléphone et des corps de messages ; ne gardez le plist en place que lorsque vous avez activement besoin du detail supplementaire.
