---
summary: "Surfaces de journalisation, logs fichier, styles de log WS et formatage console"
read_when:
  - Modification de la sortie ou des formats de journalisation
  - Débogage de la sortie CLI ou gateway
title: "Journalisation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/logging.md
  workflow: manual
---

# Journalisation

Pour un aperçu orienté utilisateur (CLI + Interface de contrôle + configuration), voir [/logging](/logging).

OpenClaw a deux « surfaces » de log :

- **Sortie console** (ce que vous voyez dans le terminal / Debug UI).
- **Logs fichier** (lignes JSON) écrits par le logger du gateway.

## Logger basé sur fichier

- Le fichier de log rotatif par défaut se trouve sous `/tmp/openclaw/` (un fichier par jour) : `openclaw-YYYY-MM-DD.log`
  - La date utilisé le fuseau horaire local de l'hôte gateway.
- Le chemin du fichier de log et le niveau peuvent être configurés via `~/.openclaw/openclaw.json` :
  - `logging.file`
  - `logging.level`

Le format du fichier est un objet JSON par ligne.

L'onglet Logs de l'Interface de contrôle suit ce fichier via le gateway (`logs.tail`).
Le CLI peut faire la même chose :

```bash
openclaw logs --follow
```

**Verbose vs. niveaux de log**

- Les **logs fichier** sont contrôlés exclusivement par `logging.level`.
- `--verbose` n'affecte que la **verbosité console** (et le style de log WS) ; il **n'augmente pas** le niveau de log fichier.
- Pour capturer les détails uniquement verbose dans les logs fichier, définissez `logging.level` sur `debug` ou `trace`.

## Capture console

Le CLI capture `console.log/info/warn/error/debug/trace` et les écrit dans les logs fichier, tout en continuant d'imprimer sur stdout/stderr.

Vous pouvez ajuster la verbosité console indépendamment via :

- `logging.consoleLevel` (par défaut `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## Masquage des résumés d'outils

Les résumés d'outils verbeux (par ex. `🛠️ Exec: ...`) peuvent masquer les jetons sensibles avant qu'ils n'atteignent le flux console. C'est **uniquement pour les outils** et n'altère pas les logs fichier.

- `logging.redactSensitive` : `off` | `tools` (par défaut : `tools`)
- `logging.redactPatterns` : tableau de chaînes regex (remplace les valeurs par défaut)
  - Utilisez des chaînes regex brutes (auto `gi`), ou `/pattern/flags` si vous avez besoin de flags personnalisés.
  - Les correspondances sont masquées en gardant les 6 premiers + 4 derniers caractères (longueur >= 18), sinon `***`.
  - Les valeurs par défaut couvrent les assignations de clés courantes, les flags CLI, les champs JSON, les en-têtes bearer, les blocs PEM et les préfixes de jetons populaires.

## Logs WebSocket du gateway

Le gateway affiche les logs du protocole WebSocket en deux modes :

- **Mode normal (sans `--verbose`)** : seuls les résultats RPC « intéressants » sont affichés :
  - erreurs (`ok=false`)
  - appels lents (seuil par défaut : `>= 50ms`)
  - erreurs d'analyse
- **Mode verbose (`--verbose`)** : affiche tout le trafic WS requête/réponse.

### Style de log WS

`openclaw gateway` supporte un interrupteur de style par gateway :

- `--ws-log auto` (par défaut) : le mode normal est optimisé ; le mode verbose utilise une sortie compacte
- `--ws-log compact` : sortie compacte (requête/réponse appariées) en verbose
- `--ws-log full` : sortie complète par trame en verbose
- `--compact` : alias pour `--ws-log compact`

Exemples :

```bash
# optimisé (erreurs/lents uniquement)
openclaw gateway

# afficher tout le trafic WS (appairé)
openclaw gateway --verbose --ws-log compact

# afficher tout le trafic WS (méta complet)
openclaw gateway --verbose --ws-log full
```

## Formatage console (journalisation par sous-système)

Le formateur console est **conscient du TTY** et affiche des lignes cohérentes et préfixées.
Les loggers de sous-système gardent la sortie groupée et scannable.

Comportement :

- **Préfixes de sous-système** sur chaque ligne (par ex. `[gateway]`, `[canvas]`, `[tailscale]`)
- **Couleurs de sous-système** (stables par sous-système) plus coloration de niveau
- **Couleur lorsque la sortie est un TTY ou que l'environnement ressemble à un terminal riche** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respecte `NO_COLOR`
- **Préfixes de sous-système raccourcis** : supprimé le `gateway/` + `channels/` en tête, garde les 2 derniers segments (par ex. `whatsapp/outbound`)
- **Sous-loggers par sous-système** (préfixe auto + champ structuré `{ subsystem }`)
- **`logRaw()`** pour la sortie QR/UX (pas de préfixe, pas de formatage)
- **Styles console** (par ex. `pretty | compact | json`)
- **Niveau de log console** séparé du niveau de log fichier (le fichier garde le détail complet lorsque `logging.level` est défini sur `debug`/`trace`)
- **Corps des messages WhatsApp** journalisés au niveau `debug` (utilisez `--verbose` pour les voir)

Cela maintient les logs fichier existants stables tout en rendant la sortie interactive scannable.
