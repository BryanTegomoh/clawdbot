---
summary: "Gestion de la date et de l'heure dans les enveloppes, prompts, outils et connecteurs"
read_when:
  - Vous modifiez la façon dont les horodatages sont affichés au modèle ou aux utilisateurs
  - Vous déboguez le formatage de l'heure dans les messages ou la sortie du prompt système
title: "Date et heure"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/date-time.md
  workflow: manual
---

# Date et heure

OpenClaw utilise par défaut **l'heure locale de l'hôte pour les horodatages de transport** et **le fuseau horaire de l'utilisateur uniquement dans le prompt système**.
Les horodatages du fournisseur sont préservés afin que les outils conservent leur sémantique native (l'heure actuelle est disponible via `session_status`).

## Enveloppes de messages (local par défaut)

Les messages entrants sont encapsulés avec un horodatage (précision à la minute) :

```
[Provider ... 2026-01-05 16:26 PST] message text
```

Cet horodatage d'enveloppe est **local à l'hôte par défaut**, quel que soit le fuseau horaire du fournisseur.

Vous pouvez modifier ce comportement :

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | fuseau horaire IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on", // "on" | "off"
    },
  },
}
```

- `envelopeTimezone: "utc"` utilise UTC.
- `envelopeTimezone: "local"` utilise le fuseau horaire de l'hôte.
- `envelopeTimezone: "user"` utilise `agents.defaults.userTimezone` (repli sur le fuseau horaire de l'hôte).
- Utilisez un fuseau horaire IANA explicite (ex. `"America/Chicago"`) pour une zone fixe.
- `envelopeTimestamp: "off"` supprime les horodatages absolus des en-têtes d'enveloppe.
- `envelopeElapsed: "off"` supprime les suffixes de temps écoulé (le style `+2m`).

### Exemples

**Local (par défaut) :**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**Fuseau horaire de l'utilisateur :**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Temps écoulé activé :**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

## Prompt système : Date et heure actuelles

Si le fuseau horaire de l'utilisateur est connu, le prompt système inclut une section dédiée
**Date et heure actuelles** avec le **fuseau horaire uniquement** (pas de format d'horloge/heure)
pour maintenir la stabilité du cache de prompt :

```
Time zone: America/Chicago
```

Quand l'agent a besoin de l'heure actuelle, utilisez l'outil `session_status` ; la carte
de statut inclut une ligne d'horodatage.

## Lignes d'événements système (local par défaut)

Les événements système en file d'attente insérés dans le contexte de l'agent sont préfixés d'un horodatage utilisant
la même sélection de fuseau horaire que les enveloppes de messages (par défaut : local à l'hôte).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

### Configurer le fuseau horaire + le format utilisateur

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto", // auto | 12 | 24
    },
  },
}
```

- `userTimezone` définit le **fuseau horaire local de l'utilisateur** pour le contexte du prompt.
- `timeFormat` contrôle l'**affichage 12h/24h** dans le prompt. `auto` suit les préférences de l'OS.

## Détection du format d'heure (auto)

Quand `timeFormat: "auto"`, OpenClaw inspecte la préférence de l'OS (macOS/Windows)
et se replie sur le formatage de la locale. La valeur détectée est **mise en cache par processus**
pour éviter les appels système répétés.

## Payloads d'outils + connecteurs (heure brute du fournisseur + champs normalisés)

Les outils de canal retournent des **horodatages natifs du fournisseur** et ajoutent des champs normalisés pour la cohérence :

- `timestampMs` : millisecondes epoch (UTC)
- `timestampUtc` : chaîne ISO 8601 UTC

Les champs bruts du fournisseur sont préservés pour ne rien perdre.

- Slack : chaînes de type epoch depuis l'API
- Discord : horodatages ISO UTC
- Telegram/WhatsApp : horodatages numériques/ISO spécifiques au fournisseur

Si vous avez besoin de l'heure locale, convertissez-la en aval en utilisant le fuseau horaire connu.

## Documentation associée

- [Prompt système](/concepts/system-prompt)
- [Fuseaux horaires](/concepts/timezone)
- [Messages](/concepts/messages)
