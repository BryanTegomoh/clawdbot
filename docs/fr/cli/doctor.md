---
summary: "Référence CLI pour `openclaw doctor` (vérifications de santé + réparations guidées)"
read_when:
  - Vous avez des problèmes de connectivité/authentification et souhaitez des corrections guidées
  - Vous avez mis à jour et souhaitez une vérification de cohérence
title: "doctor"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/doctor.md
  workflow: manual
---

# `openclaw doctor`

Vérifications de santé + corrections rapides pour le Gateway et les canaux.

Liens :

- Dépannage : [Troubleshooting](/gateway/troubleshooting)
- Audit de sécurité : [Security](/gateway/security)

## Exemples

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Notes :

- Les invites interactives (comme les corrections trousseau/OAuth) ne s'exécutent que lorsque stdin est un TTY et que `--non-interactive` n'est **pas** défini. Les exécutions headless (cron, Telegram, sans terminal) ignoreront les invites.
- `--fix` (alias de `--repair`) écrit une sauvegarde dans `~/.openclaw/openclaw.json.bak` et supprime les clés de configuration inconnues, listant chaque suppression.
- Les vérifications d'intégrité de l'état détectent désormais les fichiers de transcription orphelins dans le répertoire des sessions et peuvent les archiver en `.deleted.<timestamp>` pour récupérer de l'espace en toute sécurité.
- Doctor inclut une vérification de disponibilité de la recherche mémoire et peut recommander `openclaw configuré --section model` lorsque les identifiants d'embedding sont manquants.
- Si le mode bac à sable est activé mais que Docker n'est pas disponible, doctor signale un avertissement à fort signal avec remédiation (`installer Docker` ou `openclaw config set agents.defaults.sandbox.mode off`).

## macOS : surcharges env `launchctl`

Si vous avez précédemment exécute `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (ou `...PASSWORD`), cette valeur surcharge votre fichier de configuration et peut causer des erreurs "unauthorized" persistantes.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
