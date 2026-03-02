---
summary: "Manifeste de plugin et exigences du schema JSON (validation stricte de la configuration)"
read_when:
  - Vous construisez un plugin OpenClaw
  - Vous devez fournir un schema de configuration de plugin ou deboguer des erreurs de validation de plugin
title: "Manifeste de plugin"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/plugins/manifest.md
  workflow: manual
---

# Manifeste de plugin (openclaw.plugin.json)

Chaque plugin **doit** fournir un fichier `openclaw.plugin.json` a la **racine du plugin**.
OpenClaw utilise ce manifeste pour valider la configuration **sans exécuter le
code du plugin**. Les manifestes manquants ou invalides sont traites comme des erreurs de plugin et bloquent
la validation de la configuration.

Consultez le guide complet du système de plugins : [Plugins](/tools/plugin).

## Champs requis

```json
{
  "id": "voice-call",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

Clés requises :

- `id` (string) : identifiant canonique du plugin.
- `configSchema` (object) : schema JSON pour la configuration du plugin (en ligne).

Clés optionnelles :

- `kind` (string) : type de plugin (exemple : `"memory"`).
- `channels` (array) : identifiants de canaux enregistres par ce plugin (exemple : `["matrix"]`).
- `providers` (array) : identifiants de fournisseurs enregistres par ce plugin.
- `skills` (array) : répertoires de Skills a charger (relatifs a la racine du plugin).
- `name` (string) : nom d'affichage du plugin.
- `description` (string) : résumé court du plugin.
- `uiHints` (object) : etiquettes/espaces réservés/indicateurs de sensibilité des champs de configuration pour le rendu de l'interface.
- `version` (string) : version du plugin (a titre informatif).

## Exigences du schema JSON

- **Chaque plugin doit fournir un schema JSON**, même s'il n'accepte aucune configuration.
- Un schema vide est acceptable (par exemple, `{ "type": "object", "additionalProperties": false }`).
- Les schemas sont valides au moment de la lecture/écriture de la configuration, pas a l'exécution.

## Comportement de validation

- Les clés `channels.*` inconnues sont des **erreurs**, sauf si l'identifiant du canal est déclaré par
  un manifeste de plugin.
- `plugins.entries.<id>`, `plugins.allow`, `plugins.deny` et `plugins.slots.*`
  doivent referencer des identifiants de plugin **detectables**. Les identifiants inconnus sont des **erreurs**.
- Si un plugin est installe mais possède un manifeste ou un schema casse ou manquant,
  la validation échoue et Doctor signale l'erreur du plugin.
- Si la configuration du plugin existe mais que le plugin est **désactivé**, la configuration est conservee et
  un **avertissement** est affiche dans Doctor et les journaux.

## Notes

- Le manifeste est **requis pour tous les plugins**, y compris les chargements depuis le système de fichiers local.
- L'exécution charge toujours le module du plugin séparément ; le manifeste sert uniquement a la
  découverte et a la validation.
- Si votre plugin depend de modules natifs, documentez les étapes de compilation et les
  exigences de liste d'autorisation du gestionnaire de paquets (par exemple, `allow-build-scripts` de pnpm
  - `pnpm rebuild <package>`).
