---
summary: "Indicateurs de diagnostic pour des journaux de débogage cibles"
read_when:
  - Vous avez besoin de journaux de débogage cibles sans augmenter les niveaux de journalisation globaux
  - Vous devez capturer des journaux spécifiques a un sous-système pour le support
title: "Indicateurs de diagnostic"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/diagnostics/flags.md
  workflow: manual
---

# Indicateurs de diagnostic

Les indicateurs de diagnostic vous permettent d'activer des journaux de débogage cibles sans activer la journalisation verbeuse partout. Les indicateurs sont optionnels et n'ont aucun effet sauf si un sous-système les vérifié.

## Fonctionnement

- Les indicateurs sont des chaines de caractères (insensibles a la casse).
- Vous pouvez activer les indicateurs dans la configuration ou via une variable d'environnement.
- Les caractères generiques sont pris en charge :
  - `telegram.*` correspond a `telegram.http`
  - `*` activé tous les indicateurs

## Activation via la configuration

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Plusieurs indicateurs :

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Redémarrez le Gateway après avoir modifie les indicateurs.

## Variable d'environnement (ponctuelle)

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Désactiver tous les indicateurs :

```bash
OPENCLAW_DIAGNOSTICS=0
```

## Destination des journaux

Les indicateurs emettent des journaux dans le fichier de journaux de diagnostic standard. Par défaut :

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Si vous définissez `logging.file`, utilisez ce chemin a la place. Les journaux sont au format JSONL (un objet JSON par ligne). La redaction s'applique toujours selon `logging.redactSensitive`.

## Extraction des journaux

Sélectionner le dernier fichier de journaux :

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filtrer les diagnostics HTTP Telegram :

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

Ou suivre en temps reel pendant la reproduction :

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Pour les Gateways distants, vous pouvez également utiliser `openclaw logs --follow` (voir [/cli/logs](/cli/logs)).

## Notes

- Si `logging.level` est défini a un niveau supérieur a `warn`, ces journaux peuvent être supprimés. Le niveau par défaut `info` convient.
- Les indicateurs peuvent être laisses activés en toute sécurité ; ils n'affectent que le volume de journaux pour le sous-système spécifique.
- Utilisez [/logging](/logging) pour modifier les destinations, niveaux et la redaction des journaux.
