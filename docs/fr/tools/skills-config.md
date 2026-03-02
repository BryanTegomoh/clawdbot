---
summary: "Schéma de configuration des Skills et exemples"
read_when:
  - Ajout ou modification de la configuration des Skills
  - Ajustement de la liste d'autorisation des Skills intégrés ou du comportement d'installation
title: "Configuration des Skills"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/skills-config.md
  workflow: manual
---

# Configuration des Skills

Toute la configuration liée aux Skills se trouve sous `skills` dans `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (le runtime du Gateway reste Node ; bun non recommandé)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // ou chaîne en clair
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## Champs

- `allowBundled` : liste d'autorisation optionnelle pour les Skills **intégrés** uniquement. Lorsqu'elle est définie, seuls les Skills intégrés de la liste sont éligibles (les Skills gérés/workspace ne sont pas affectés).
- `load.extraDirs` : répertoires de Skills supplémentaires à scanner (priorité la plus basse).
- `load.watch` : surveiller les dossiers de Skills et rafraîchir le snapshot des Skills (défaut : true).
- `load.watchDebounceMs` : anti-rebond pour les événements du watcher de Skills en millisecondes (défaut : 250).
- `install.preferBrew` : préférer les installateurs brew lorsque disponibles (défaut : true).
- `install.nodeManager` : préférence d'installateur node (`npm` | `pnpm` | `yarn` | `bun`, défaut : npm).
  Cela n'affecte que les **installations de Skills** ; le runtime du Gateway devrait rester Node
  (Bun non recommandé pour WhatsApp/Telegram).
- `entries.<skillKey>` : remplacements par Skill.

Champs par Skill :

- `enabled` : définir à `false` pour désactiver un Skill même s'il est intégré/installé.
- `env` : variables d'environnement injectées pour l'exécution de l'agent (uniquement si non déjà définies).
- `apiKey` : raccourci optionnel pour les Skills qui déclarent une variable d'environnement principale.
  Supporte une chaîne en clair ou un objet SecretRef (`{ source, provider, id }`).

## Notes

- Les clés sous `entries` correspondent au nom du Skill par défaut. Si un Skill définit
  `metadata.openclaw.skillKey`, utilisez cette clé à la place.
- Les changements de Skills sont pris en compte au prochain tour d'agent lorsque le watcher est activé.

### Skills sandboxés + variables d'environnement

Lorsqu'une session est **sandboxée**, les processus de Skills s'exécutent dans Docker. Le bac à sable
n'hérite **pas** du `process.env` de l'hôte.

Utilisez l'un des éléments suivants :

- `agents.defaults.sandbox.docker.env` (ou par agent `agents.list[].sandbox.docker.env`)
- intégrez les variables dans votre image de bac à sable personnalisée

Les `env` et `skills.entries.<skill>.env/apiKey` globaux s'appliquent aux exécutions **hôte** uniquement.
