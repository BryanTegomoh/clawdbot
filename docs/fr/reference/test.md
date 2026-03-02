---
summary: "Comment exécuter les tests localement (vitest) et quand utiliser les modes force/coverage"
read_when:
  - Exécution ou correction de tests
title: "Tests"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/test.md
  workflow: manual
---

# Tests

- Kit de test complet (suites, live, Docker) : [Testing](/help/testing)

- `pnpm test:force` : Tue tout processus Gateway persistant occupant le port de contrôle par défaut, puis exécute la suite Vitest complète avec un port Gateway isolé afin que les tests serveur ne rentrent pas en conflit avec une instance en cours d'exécution. Utilisez ceci quand une exécution Gateway précédente a laissé le port 18789 occupé.
- `pnpm test:coverage` : Exécute la suite unitaire avec la couverture V8 (via `vitest.unit.config.ts`). Les seuils globaux sont de 70% pour les lignes/branches/fonctions/instructions. La couverture exclut les points d'entrée fortement intégrés (câblage CLI, passerelles gateway/telegram, serveur statique webchat) pour garder l'objectif centré sur la logique testable unitairement.
- `pnpm test` sur Node 24+ : OpenClaw désactive automatiquement `vmForks` de Vitest et utilise `forks` pour éviter `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`. Vous pouvez forcer le comportement avec `OPENCLAW_TEST_VM_FORKS=0|1`.
- `pnpm test:e2e` : Exécute les tests de fumée de bout en bout du Gateway (multi-instance WS/HTTP/appariement de nœuds). Par défaut `vmForks` + workers adaptatifs dans `vitest.e2e.config.ts` ; ajustez avec `OPENCLAW_E2E_WORKERS=<n>` et définissez `OPENCLAW_E2E_VERBOSE=1` pour les logs verbeux.
- `pnpm test:live` : Exécute les tests live fournisseur (minimax/zai). Nécessite des clés API et `LIVE=1` (ou `*_LIVE_TEST=1` spécifique au fournisseur) pour ne pas être ignoré.

## Gate PR locale

Pour les vérifications locales de gate/atterrissage de PR, exécutez :

- `pnpm check`
- `pnpm build`
- `pnpm test`
- `pnpm check:docs`

Si `pnpm test` est instable sur un hôte chargé, relancez une fois avant de le traiter comme une régression, puis isolez avec `pnpm vitest run <path/to/test>`. Pour les hôtes limités en mémoire, utilisez :

- `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## Benchmark de latence modèle (clés locales)

Script : [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

Utilisation :

- `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
- Variables d'environnement optionnelles : `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
- Prompt par défaut : "Reply with a single word: ok. No punctuation or extra text."

Dernière exécution (2025-12-31, 20 exécutions) :

- minimax médiane 1279ms (min 1114, max 2431)
- opus médiane 2454ms (min 1224, max 3170)

## Onboarding E2E (Docker)

Docker est optionnel ; ceci n'est nécessaire que pour les tests de fumée d'onboarding conteneurisés.

Flux complet de démarrage à froid dans un conteneur Linux propre :

```bash
scripts/e2e/onboard-docker.sh
```

Ce script pilote l'assistant interactif via un pseudo-tty, vérifie les fichiers de configuration/espace de travail/session, puis démarre le Gateway et exécute `openclaw health`.

## Test de fumée d'import QR (Docker)

Vérifie que `qrcode-terminal` se charge sous Node 22+ dans Docker :

```bash
pnpm test:docker:qr
```
