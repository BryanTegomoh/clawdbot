---
summary: "Envoi de sondages via le Gateway + CLI"
read_when:
  - Ajout ou modification du support des sondages
  - Débogage des envois de sondages depuis le CLI ou le Gateway
title: "Sondages"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/automation/poll.md
  workflow: manual
---

# Sondages

## Canaux supportés

- WhatsApp (canal web)
- Discord
- MS Teams (Adaptive Cards)

## CLI

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Options :

- `--channel` : `whatsapp` (par défaut), `discord`, ou `msteams`
- `--poll-multi` : autoriser la sélection de plusieurs options
- `--poll-duration-hours` : Discord uniquement (par défaut 24 quand omis)

## Gateway RPC

Méthode : `poll`

Paramètres :

- `to` (string, requis)
- `question` (string, requis)
- `options` (string[], requis)
- `maxSelections` (number, optionnel)
- `durationHours` (number, optionnel)
- `channel` (string, optionnel, par défaut : `whatsapp`)
- `idempotencyKey` (string, requis)

## Différences entre canaux

- WhatsApp : 2-12 options, `maxSelections` doit être dans la limité du nombre d'options, ignoré `durationHours`.
- Discord : 2-10 options, `durationHours` plafonné à 1-768 heures (par défaut 24). `maxSelections > 1` activé la sélection multiple ; Discord ne supporte pas un nombre strict de sélections.
- MS Teams : sondages par Adaptive Card (gérés par OpenClaw). Pas d'API de sondage native ; `durationHours` est ignoré.

## Outil d'agent (Message)

Utilisez l'outil `message` avec l'action `poll` (`to`, `pollQuestion`, `pollOption`, optionnellement `pollMulti`, `pollDurationHours`, `channel`).

Note : Discord n'a pas de mode "choisir exactement N" ; `pollMulti` correspond à la sélection multiple.
Les sondages Teams sont rendus comme des Adaptive Cards et nécessitent que le Gateway reste en ligne
pour enregistrer les votes dans `~/.openclaw/msteams-polls.json`.
