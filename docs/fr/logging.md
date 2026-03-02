---
summary: "Vue d'ensemble de la journalisation : logs fichier, sortie console, suivi CLI et UI de contrôle"
read_when:
  - Vous avez besoin d'une vue d'ensemble accessible de la journalisation
  - Vous souhaitez configurer les niveaux ou formats de log
  - Vous dépannez et avez besoin de trouver les logs rapidement
title: "Journalisation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/logging.md
  workflow: manual
---

# Journalisation

OpenClaw journalise dans deux endroits :

- **Logs fichier** (lignes JSON) écrits par le Gateway.
- **Sortie console** affichée dans les terminaux et l'UI de contrôle.

Cette page explique où se trouvent les logs, comment les lire et comment configurer les niveaux
et formats de log.

## Où se trouvent les logs

Par défaut, le Gateway écrit un fichier de log rotatif sous :

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

La date utilisé le fuseau horaire local de l'hôte du Gateway.

Vous pouvez remplacer cela dans `~/.openclaw/openclaw.json` :

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

## Comment lire les logs

### CLI : suivi en direct (recommandé)

Utilisez le CLI pour suivre le fichier de log du Gateway via RPC :

```bash
openclaw logs --follow
```

Modes de sortie :

- **Sessions TTY** : lignes de log structurées, colorisées et lisibles.
- **Sessions non-TTY** : texte brut.
- `--json` : JSON délimité par ligne (un événement de log par ligne).
- `--plain` : forcer le texte brut en sessions TTY.
- `--no-color` : désactiver les couleurs ANSI.

En mode JSON, le CLI émet des objets tagués par `type` :

- `meta` : métadonnées du flux (fichier, curseur, taille)
- `log` : entrée de log parsée
- `notice` : indices de troncature / rotation
- `raw` : ligne de log non parsée

Si le Gateway est injoignable, le CLI affiche une indication rapide pour exécuter :

```bash
openclaw doctor
```

### UI de contrôle (web)

L'onglet **Logs** de l'UI de contrôle suit le même fichier en utilisant `logs.tail`.
Voir [/web/control-ui](/web/control-ui) pour savoir comment l'ouvrir.

### Logs par canal uniquement

Pour filtrer l'activité d'un canal (WhatsApp/Telegram/etc), utilisez :

```bash
openclaw channels logs --channel whatsapp
```

## Formats de log

### Logs fichier (JSONL)

Chaque ligne du fichier de log est un objet JSON. Le CLI et l'UI de contrôle parsent ces
entrées pour afficher une sortie structurée (heure, niveau, sous-système, message).

### Sortie console

Les logs console sont **sensibles au TTY** et formatés pour la lisibilité :

- Préfixes de sous-système (ex. `gateway/channels/whatsapp`)
- Coloration par niveau (info/warn/error)
- Mode compact ou JSON optionnel

Le formatage console est contrôlé par `logging.consoleStyle`.

## Configurer la journalisation

Toute la configuration de journalisation se trouve sous `logging` dans `~/.openclaw/openclaw.json`.

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": ["sk-.*"]
  }
}
```

### Niveaux de log

- `logging.level` : niveau des **logs fichier** (JSONL).
- `logging.consoleLevel` : niveau de verbosité de la **console**.

Vous pouvez remplacer les deux via la variable d'environnement **`OPENCLAW_LOG_LEVEL`** (ex. `OPENCLAW_LOG_LEVEL=debug`). La variable d'environnement a priorité sur le fichier de config, vous permettant d'augmenter la verbosité pour une seule exécution sans modifier `openclaw.json`. Vous pouvez aussi passer l'option CLI globale **`--log-level <level>`** (par exemple, `openclaw --log-level debug gateway run`), qui remplace la variable d'environnement pour cette commande.

`--verbose` n'affecte que la sortie console ; il ne change pas les niveaux de log fichier.

### Styles de console

`logging.consoleStyle` :

- `pretty` : lisible, coloré, avec horodatages.
- `compact` : sortie plus serrée (idéal pour les longues sessions).
- `json` : JSON par ligne (pour les processeurs de logs).

### Masquage

Les résumés d'outils peuvent masquer les tokens sensibles avant qu'ils n'atteignent la console :

- `logging.redactSensitive` : `off` | `tools` (par défaut : `tools`)
- `logging.redactPatterns` : liste de chaînes regex pour remplacer le jeu par défaut

Le masquage n'affecte que la **sortie console** et ne modifie pas les logs fichier.

## Diagnostics + OpenTelemetry

Les diagnostics sont des événements structurés et lisibles par machine pour les exécutions de modèle **et**
la télémétrie de flux de messages (webhooks, mise en file d'attente, état de session). Ils ne **remplacent pas**
les logs ; ils existent pour alimenter les métriques, traces et autres exporteurs.

Les événements de diagnostics sont émis en processus, mais les exporteurs ne s'attachent que quand
les diagnostics + le plugin exporteur sont activés.

### OpenTelemetry vs OTLP

- **OpenTelemetry (OTel)** : le modèle de données + SDKs pour les traces, métriques et logs.
- **OTLP** : le protocole filaire utilisé pour exporter les données OTel vers un collecteur/backend.
- OpenClaw exporte via **OTLP/HTTP (protobuf)** aujourd'hui.

### Signaux exportés

- **Métriques** : compteurs + histogrammes (utilisation de tokens, flux de messages, file d'attente).
- **Traces** : spans pour l'utilisation de modèle + le traitement webhook/message.
- **Logs** : exportés via OTLP quand `diagnostics.otel.logs` est activé. Le
  volume de logs peut être élevé ; gardez `logging.level` et les filtres d'exporteur à l'esprit.

### Catalogue d'événements diagnostics

Utilisation de modèle :

- `model.usage` : tokens, coût, durée, contexte, fournisseur/modèle/canal, ids de session.

Flux de messages :

- `webhook.received` : entrée webhook par canal.
- `webhook.processed` : webhook traité + durée.
- `webhook.error` : erreurs du gestionnaire webhook.
- `message.queued` : message mis en file d'attente pour traitement.
- `message.processed` : résultat + durée + erreur optionnelle.

File d'attente + session :

- `queue.lane.enqueue` : mise en file de la voie de commandes + profondeur.
- `queue.lane.dequeue` : retrait de file de la voie de commandes + temps d'attente.
- `session.state` : transition d'état de session + raison.
- `session.stuck` : avertissement de session bloquée + âge.
- `run.attempt` : métadonnées de tentative/relance d'exécution.
- `diagnostic.heartbeat` : compteurs agrégés (webhooks/file d'attente/session).

### Activer les diagnostics (sans exporteur)

Utilisez ceci si vous voulez des événements diagnostics disponibles pour les plugins ou les récepteurs personnalisés :

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

### Flags de diagnostics (logs ciblés)

Utilisez les flags pour activer des logs de débogage ciblés supplémentaires sans augmenter `logging.level`.
Les flags sont insensibles à la casse et supportent les wildcards (ex. `telegram.*` ou `*`).

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Remplacement par variable d'environnement (ponctuel) :

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Notes :

- Les logs de flags vont dans le fichier de log standard (même que `logging.file`).
- La sortie est toujours masquée selon `logging.redactSensitive`.
- Guide complet : [/diagnostics/flags](/diagnostics/flags).

### Exporter vers OpenTelemetry

Les diagnostics peuvent être exportés via le plugin `diagnostics-otel` (OTLP/HTTP). Cela
fonctionne avec tout collecteur/backend OpenTelemetry acceptant OTLP/HTTP.

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

Notes :

- Vous pouvez aussi activer le plugin avec `openclaw plugins enable diagnostics-otel`.
- `protocol` supporte actuellement uniquement `http/protobuf`. `grpc` est ignoré.
- Les métriques incluent l'utilisation de tokens, le coût, la taille du contexte, la durée d'exécution et les
  compteurs/histogrammes de flux de messages (webhooks, file d'attente, état de session, profondeur/attente de file).
- Les traces/métriques peuvent être activées/désactivées avec `traces` / `metrics` (par défaut : activé). Les traces
  incluent les spans d'utilisation de modèle plus les spans de traitement webhook/message quand activées.
- Définissez `headers` quand votre collecteur nécessite une authentification.
- Variables d'environnement supportées : `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

### Métriques exportées (noms + types)

Utilisation de modèle :

- `openclaw.tokens` (compteur, attrs : `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
- `openclaw.cost.usd` (compteur, attrs : `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
- `openclaw.run.duration_ms` (histogramme, attrs : `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
- `openclaw.context.tokens` (histogramme, attrs : `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

Flux de messages :

- `openclaw.webhook.received` (compteur, attrs : `openclaw.channel`,
  `openclaw.webhook`)
- `openclaw.webhook.error` (compteur, attrs : `openclaw.channel`,
  `openclaw.webhook`)
- `openclaw.webhook.duration_ms` (histogramme, attrs : `openclaw.channel`,
  `openclaw.webhook`)
- `openclaw.message.queued` (compteur, attrs : `openclaw.channel`,
  `openclaw.source`)
- `openclaw.message.processed` (compteur, attrs : `openclaw.channel`,
  `openclaw.outcome`)
- `openclaw.message.duration_ms` (histogramme, attrs : `openclaw.channel`,
  `openclaw.outcome`)

Files d'attente + sessions :

- `openclaw.queue.lane.enqueue` (compteur, attrs : `openclaw.lane`)
- `openclaw.queue.lane.dequeue` (compteur, attrs : `openclaw.lane`)
- `openclaw.queue.depth` (histogramme, attrs : `openclaw.lane` ou
  `openclaw.channel=heartbeat`)
- `openclaw.queue.wait_ms` (histogramme, attrs : `openclaw.lane`)
- `openclaw.session.state` (compteur, attrs : `openclaw.state`, `openclaw.reason`)
- `openclaw.session.stuck` (compteur, attrs : `openclaw.state`)
- `openclaw.session.stuck_age_ms` (histogramme, attrs : `openclaw.state`)
- `openclaw.run.attempt` (compteur, attrs : `openclaw.attempt`)

### Spans exportés (noms + attributs clés)

- `openclaw.model.usage`
  - `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  - `openclaw.sessionKey`, `openclaw.sessionId`
  - `openclaw.tokens.*` (input/output/cache_read/cache_write/total)
- `openclaw.webhook.processed`
  - `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
- `openclaw.webhook.error`
  - `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
- `openclaw.message.processed`
  - `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
    `openclaw.reason`
- `openclaw.session.stuck`
  - `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

### Échantillonnage + vidage

- Échantillonnage des traces : `diagnostics.otel.sampleRate` (0.0–1.0, spans racine uniquement).
- Intervalle d'export des métriques : `diagnostics.otel.flushIntervalMs` (min 1000ms).

### Notes sur le protocole

- Les endpoints OTLP/HTTP peuvent être définis via `diagnostics.otel.endpoint` ou
  `OTEL_EXPORTER_OTLP_ENDPOINT`.
- Si l'endpoint contient déjà `/v1/traces` ou `/v1/metrics`, il est utilisé tel quel.
- Si l'endpoint contient déjà `/v1/logs`, il est utilisé tel quel pour les logs.
- `diagnostics.otel.logs` activé l'export de logs OTLP pour la sortie du logger principal.

### Comportement d'export des logs

- Les logs OTLP utilisent les mêmes enregistrements structurés écrits dans `logging.file`.
- Respectent `logging.level` (niveau de log fichier). Le masquage console ne s'applique **pas**
  aux logs OTLP.
- Les installations à haut volume devraient préférer l'échantillonnage/filtrage du collecteur OTLP.

## Conseils de dépannage

- **Gateway injoignable ?** Exécutez d'abord `openclaw doctor`.
- **Logs vides ?** Vérifiez que le Gateway est en cours d'exécution et écrit dans le chemin de fichier
  configuré dans `logging.file`.
- **Besoin de plus de détails ?** Définissez `logging.level` sur `debug` ou `trace` et réessayez.
