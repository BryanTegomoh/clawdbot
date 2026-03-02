---
title: "Checklist de release"
summary: "Checklist de release étape par étape pour npm + application macOS"
read_when:
  - Préparation d'une nouvelle release npm
  - Préparation d'une nouvelle release de l'application macOS
  - Vérification des métadonnées avant publication
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/RELEASING.md
  workflow: manual
---

# Checklist de release (npm + macOS)

Utilisez `pnpm` (Node 22+) depuis la racine du dépôt. Gardez l'arbre de travail propre avant le tagging/publication.

## Déclenchement par l'opérateur

Quand l'opérateur dit « release », faites immédiatement ce preflight (pas de questions supplémentaires sauf blocage) :

- Lire ce document et `docs/platforms/mac/release.md`.
- Charger l'environnement depuis `~/.profile` et confirmer que `SPARKLE_PRIVATE_KEY_FILE` + les variables App Store Connect sont définies (SPARKLE_PRIVATE_KEY_FILE doit être dans `~/.profile`).
- Utiliser les clés Sparkle depuis `~/Library/CloudStorage/Dropbox/Backup/Sparkle` si nécessaire.

1. **Version et métadonnées**

- [ ] Incrémenter la version dans `package.json` (ex. `2026.1.29`).
- [ ] Exécuter `pnpm plugins:sync` pour aligner les versions et changelogs des extensions.
- [ ] Mettre à jour les chaînes CLI/version dans [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) et le user agent Baileys dans [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts).
- [ ] Confirmer les métadonnées du package (name, description, repository, keywords, license) et que la map `bin` pointe vers [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) pour `openclaw`.
- [ ] Si les dépendances ont changé, exécuter `pnpm install` pour que `pnpm-lock.yaml` soit à jour.

2. **Build et artefacts**

- [ ] Si les entrées A2UI ont changé, exécuter `pnpm canvas:a2ui:bundle` et committer tout [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) mis à jour.
- [ ] `pnpm run build` (régénère `dist/`).
- [ ] Vérifier que `files` dans le package npm inclut tous les dossiers `dist/*` requis (notamment `dist/node-host/**` et `dist/acp/**` pour le node headless + ACP CLI).
- [ ] Confirmer que `dist/build-info.json` existe et inclut le hash `commit` attendu (la bannière CLI utilisé ceci pour les installations npm).
- [ ] Optionnel : `npm pack --pack-destination /tmp` après le build ; inspecter le contenu du tarball et le garder à portée pour la release GitHub (ne **pas** le committer).

3. **Changelog et docs**

- [ ] Mettre à jour `CHANGELOG.md` avec les points saillants orientés utilisateur (créer le fichier s'il manque) ; garder les entrées strictement décroissantes par version.
- [ ] S'assurer que les exemples/drapeaux du README correspondent au comportement CLI actuel (notamment les nouvelles commandes ou options).

4. **Validation**

- [ ] `pnpm build`
- [ ] `pnpm check`
- [ ] `pnpm test` (ou `pnpm test:coverage` si vous avez besoin de la sortie de couverture)
- [ ] `pnpm release:check` (vérifie le contenu du npm pack)
- [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (test de fumée d'installation Docker, chemin rapide ; requis avant la release)
  - Si la release npm précédente immédiate est connue comme cassée, définissez `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` ou `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` pour l'étape de pré-installation.
- [ ] (Optionnel) Test de fumée d'installation complet (ajouté non-root + couverture CLI) : `pnpm test:install:smoke`
- [ ] (Optionnel) E2E d'installation (Docker, exécute `curl -fsSL https://openclaw.ai/install.sh | bash`, fait l'onboarding, puis exécute des appels d'outils réels) :
  - `pnpm test:install:e2e:openai` (nécessite `OPENAI_API_KEY`)
  - `pnpm test:install:e2e:anthropic` (nécessite `ANTHROPIC_API_KEY`)
  - `pnpm test:install:e2e` (nécessite les deux clés ; exécute les deux fournisseurs)
- [ ] (Optionnel) Vérification ponctuelle du Gateway web si vos changements affectent les chemins d'envoi/réception.

5. **Application macOS (Sparkle)**

- [ ] Build + signature de l'application macOS, puis zip pour la distribution.
- [ ] Générer l'appcast Sparkle (notes HTML via [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) et mettre à jour `appcast.xml`.
- [ ] Garder le zip de l'application (et le zip dSYM optionnel) prêt à attacher à la release GitHub.
- [ ] Suivre [macOS release](/platforms/mac/release) pour les commandes exactes et variables d'environnement requises.
  - `APP_BUILD` doit être numérique + monotone (pas de `-beta`) pour que Sparkle compare correctement les versions.
  - Si vous notariez, utilisez le profil de trousseau `openclaw-notary` créé à partir des variables d'environnement de l'API App Store Connect (voir [macOS release](/platforms/mac/release)).

6. **Publication (npm)**

- [ ] Confirmer que le statut git est propre ; committer et pusher si nécessaire.
- [ ] `npm login` (vérifier la 2FA) si nécessaire.
- [ ] `npm publish --access public` (utilisez `--tag beta` pour les pré-releases).
- [ ] Vérifier le registre : `npm view openclaw version`, `npm view openclaw dist-tags`, et `npx -y openclaw@X.Y.Z --version` (ou `--help`).

### Dépannage (notes de la release 2.0.0-beta2)

- **npm pack/publish bloque ou produit un tarball énorme** : le bundle de l'application macOS dans `dist/OpenClaw.app` (et les zips de release) sont aspirés dans le package. Corrigez en listant explicitement le contenu de publication via `package.json` `files` (inclure les sous-répertoires dist, docs, skills ; exclure les bundles d'application). Confirmez avec `npm pack --dry-run` que `dist/OpenClaw.app` n'est pas listé.
- **Boucle d'authentification npm web pour dist-tags** : utilisez l'authentification legacy pour obtenir une invite OTP :
  - `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
- **La vérification `npx` échoue avec `ECOMPROMISED: Lock compromised`** : réessayez avec un cache propre :
  - `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
- **Le tag doit être repositionné après un correctif tardif** : mettre à jour le tag en force et le pusher, puis s'assurer que les artefacts de la release GitHub correspondent toujours :
  - `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **Release GitHub + appcast**

- [ ] Tagger et pusher : `git tag vX.Y.Z && git push origin vX.Y.Z` (ou `git push --tags`).
- [ ] Créer/rafraîchir la release GitHub pour `vX.Y.Z` avec le **titre `openclaw X.Y.Z`** (pas juste le tag) ; le corps doit inclure la section **complète** du changelog pour cette version (Points forts + Changements + Correctifs), en ligne (pas de liens nus), et **ne doit pas répéter le titre dans le corps**.
- [ ] Attacher les artefacts : tarball `npm pack` (optionnel), `OpenClaw-X.Y.Z.zip`, et `OpenClaw-X.Y.Z.dSYM.zip` (si généré).
- [ ] Committer le `appcast.xml` mis à jour et le pusher (Sparkle se nourrit depuis main).
- [ ] Depuis un répertoire temporaire propre (pas de `package.json`), exécuter `npx -y openclaw@X.Y.Z send --help` pour confirmer que les points d'entrée installation/CLI fonctionnent.
- [ ] Annoncer/partager les notes de release.

## Portée de publication des plugins (npm)

Nous ne publions que les **plugins npm existants** sous le scope `@openclaw/*`. Les
plugins intégrés qui ne sont pas sur npm restent en **arborescence disque uniquement** (toujours livrés dans
`extensions/**`).

Processus pour dériver la liste :

1. `npm search @openclaw --json` et capturer les noms de packages.
2. Comparer avec les noms dans `extensions/*/package.json`.
3. Publier uniquement l'**intersection** (déjà sur npm).

Liste actuelle des plugins npm (mettre à jour si nécessaire) :

- @openclaw/bluebubbles
- @openclaw/diagnostics-otel
- @openclaw/discord
- @openclaw/feishu
- @openclaw/lobster
- @openclaw/matrix
- @openclaw/msteams
- @openclaw/nextcloud-talk
- @openclaw/nostr
- @openclaw/voice-call
- @openclaw/zalo
- @openclaw/zalouser

Les notes de release doivent aussi mentionner les **nouveaux plugins intégrés optionnels** qui ne sont **pas
activés par défaut** (exemple : `tlon`).
