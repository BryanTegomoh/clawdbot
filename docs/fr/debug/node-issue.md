---
summary: Notes et solutions de contournement pour le crash Node + tsx "__name is not a function"
read_when:
  - Debogage de scripts de dev Node uniquement ou échecs du mode watch
  - Investigation des crashes du chargeur tsx/esbuild dans OpenClaw
title: "Crash Node + tsx"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/debug/node-issue.md
  workflow: manual
---

# Crash Node + tsx "\_\_name is not a function"

## Résumé

L'exécution d'OpenClaw via Node avec `tsx` échoue au démarrage avec :

```
[openclaw] Failed to start CLI: TypeError: __name is not a function
    at createSubsystemLogger (.../src/logging/subsystem.ts:203:25)
    at .../src/agents/auth-profiles/constants.ts:25:20
```

Cela a commence après le passage des scripts de dev de Bun a `tsx` (commit `2871657e`, 2026-01-06). Le même chemin d'exécution fonctionnait avec Bun.

## Environnement

- Node : v25.x (observe sur v25.3.0)
- tsx : 4.21.0
- OS : macOS (reproduction probable également sur d'autres plateformes exécutant Node 25)

## Reproduction (Node uniquement)

```bash
# in repo root
node --version
pnpm install
node --import tsx src/entry.ts status
```

## Reproduction minimale dans le dépôt

```bash
node --import tsx scripts/repro/tsx-name-repro.ts
```

## Vérification de la version de Node

- Node 25.3.0 : échoue
- Node 22.22.0 (Homebrew `node@22`) : échoue
- Node 24 : pas encore installe ici ; vérification nécessaire

## Notes / hypothese

- `tsx` utilise esbuild pour transformer TS/ESM. L'option `keepNames` d'esbuild émet un helper `__name` et encapsule les définitions de fonctions avec `__name(...)`.
- Le crash indique que `__name` existe mais n'est pas une fonction a l'exécution, ce qui implique que le helper est manquant ou écrasé pour ce module dans le chemin du chargeur Node 25.
- Des problèmes similaires avec le helper `__name` ont ete signales dans d'autres consommateurs d'esbuild lorsque le helper est manquant ou réécrit.

## Historique de regression

- `2871657e` (2026-01-06) : scripts passes de Bun a tsx pour rendre Bun optionnel.
- Avant cela (chemin Bun), `openclaw status` et `gateway:watch` fonctionnaient.

## Solutions de contournement

- Utiliser Bun pour les scripts de dev (retour temporaire actuel).
- Utiliser Node + tsc watch, puis exécuter la sortie compilee :

  ```bash
  pnpm exec tsc --watch --preserveWatchOutput
  node --watch openclaw.mjs status
  ```

- Confirme localement : `pnpm exec tsc -p tsconfig.json` + `node openclaw.mjs status` fonctionne sur Node 25.
- Désactiver keepNames d'esbuild dans le chargeur TS si possible (empêche l'insertion du helper `__name`) ; tsx n'expose pas actuellement cette option.
- Tester Node LTS (22/24) avec `tsx` pour voir si le problème est spécifique a Node 25.

## Références

- [https://opennext.js.org/cloudflare/howtos/keep_names](https://opennext.js.org/cloudflare/howtos/keep_names)
- [https://esbuild.github.io/api/#keep-names](https://esbuild.github.io/api/#keep-names)
- [https://github.com/evanw/esbuild/issues/1031](https://github.com/evanw/esbuild/issues/1031)

## Prochaines étapes

- Reproduire sur Node 22/24 pour confirmer la regression Node 25.
- Tester une version nightly de `tsx` ou épingler une version anterieure si une regression connue existe.
- Si le problème se reproduit sur Node LTS, soumettre une reproduction minimale en amont avec la trace de pile `__name`.
