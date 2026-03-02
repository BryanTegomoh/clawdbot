---
summary: "Gestion des fuseaux horaires pour les agents, les enveloppés et les prompts"
read_when:
  - Vous devez comprendre comment les horodatages sont normalisés pour le modèle
  - Configuration du fuseau horaire utilisateur pour les prompts système
title: "Fuseaux horaires"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/concepts/timezone.md
  workflow: manual
---

# Fuseaux horaires

OpenClaw standardise les horodatages afin que le modèle voie une **heure de référence unique**.

## Enveloppes de messages (local par défaut)

Les messages entrants sont enveloppés dans une enveloppe comme :

```
[Provider ... 2026-01-05 16:26 PST] texte du message
```

L'horodatage dans l'enveloppe est **local à l'hôte par défaut**, avec une précision à la minute.

Vous pouvez surcharger ceci avec :

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

- `envelopeTimezone: "utc"` utilisé UTC.
- `envelopeTimezone: "user"` utilisé `agents.defaults.userTimezone` (revient au fuseau horaire de l'hôte).
- Utilisez un fuseau horaire IANA explicite (par ex. `"Europe/Vienna"`) pour un décalage fixe.
- `envelopeTimestamp: "off"` supprimé les horodatages absolus des en-têtes d'enveloppe.
- `envelopeElapsed: "off"` supprimé les suffixes de temps écoulé (le style `+2m`).

### Exemples

**Local (par défaut) :**

```
[Signal Alice +1555 2026-01-18 00:19 PST] bonjour
```

**Fuseau horaire fixe :**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] bonjour
```

**Temps écoulé :**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] suite
```

## Payloads d'outils (données brutes du fournisseur + champs normalisés)

Les appels d'outils (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) retournent les **horodatages bruts du fournisseur**. Nous attachons également des champs normalisés pour la cohérence :

- `timestampMs` (millisecondes epoch UTC)
- `timestampUtc` (chaine ISO 8601 UTC)

Les champs bruts du fournisseur sont préservés.

## Fuseau horaire utilisateur pour le prompt système

Définissez `agents.defaults.userTimezone` pour indiquer au modèle le fuseau horaire local de l'utilisateur. S'il n'est pas défini, OpenClaw résout le **fuseau horaire de l'hôte à l'exécution** (pas d'écriture de configuration).

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

Le prompt système inclut :

- La section `Current Date & Time` avec l'heure locale et le fuseau horaire
- `Time format: 12-hour` ou `24-hour`

Vous pouvez contrôler le format du prompt avec `agents.defaults.timeFormat` (`auto` | `12` | `24`).

Voir [Date et heure](/date-time) pour le comportement complet et les exemples.
