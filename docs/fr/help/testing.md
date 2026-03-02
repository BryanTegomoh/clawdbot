---
summary: "Kit de test : suites unit/e2e/live, exécuteurs Docker et ce que chaque test couvre"
read_when:
  - Exécution de tests en local ou en CI
  - Ajout de régressions pour les bugs de modèles/fournisseurs
  - Débogage du comportement gateway + agent
title: "Tests"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: help/testing.md
  workflow: manual
---

# Tests

OpenClaw dispose de trois suites Vitest (unit/intégration, e2e, live) et d'un petit ensemble d'exécuteurs Docker.

Ce document est un guide « comment nous testons » :

- Ce que chaque suite couvre (et ce qu'elle ne couvre _pas_ délibérément)
- Quelles commandes exécuter pour les flux de travail courants (local, pré-push, débogage)
- Comment les tests live découvrent les identifiants et sélectionnent les modèles/fournisseurs
- Comment ajouter des régressions pour les problèmes réels de modèles/fournisseurs

## Démarrage rapide

La plupart du temps :

- Gate complète (attendue avant un push) : `pnpm build && pnpm check && pnpm test`

Quand vous touchez aux tests ou voulez plus de confiance :

- Gate de couverture : `pnpm test:coverage`
- Suite E2E : `pnpm test:e2e`

Quand vous déboguez de vrais fournisseurs/modèles (nécessite de vrais identifiants) :

- Suite live (modèles + sondes d'outils/images du Gateway) : `pnpm test:live`

Astuce : quand vous n'avez besoin que d'un seul cas en échec, préférez restreindre les tests live via les variables d'environnement d'allowlist décrites ci-dessous.

## Suites de tests (ce qui s'exécute où)

Considérez les suites comme un « réalisme croissant » (et une instabilité/coût croissants) :

### Unitaire / intégration (par défaut)

- Commande : `pnpm test`
- Configuration : `scripts/test-parallel.mjs` (exécute `vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts`)
- Fichiers : `src/**/*.test.ts`, `extensions/**/*.test.ts`
- Portée :
  - Tests unitaires purs
  - Tests d'intégration en processus (auth du Gateway, routage, outillage, parsing, configuration)
  - Régressions déterministes pour les bugs connus
- Attentes :
  - S'exécute en CI
  - Aucune vraie clé requise
  - Doit être rapide et stable
- Note sur le pool :
  - OpenClaw utilise `vmForks` de Vitest sur Node 22/23 pour des shards unitaires plus rapides.
  - Sur Node 24+, OpenClaw bascule automatiquement sur `forks` classique pour éviter les erreurs de liaison VM de Node (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`).
  - Remplacement manuel avec `OPENCLAW_TEST_VM_FORKS=0` (forcer `forks`) ou `OPENCLAW_TEST_VM_FORKS=1` (forcer `vmForks`).

### E2E (smoke du Gateway)

- Commande : `pnpm test:e2e`
- Configuration : `vitest.e2e.config.ts`
- Fichiers : `src/**/*.e2e.test.ts`
- Paramètres d'exécution par défaut :
  - Utilisé `vmForks` de Vitest pour un démarrage de fichier plus rapide.
  - Utilisé des workers adaptatifs (CI : 2-4, local : 4-8).
  - S'exécute en mode silencieux par défaut pour réduire la surcharge d'E/S console.
- Remplacements utiles :
  - `OPENCLAW_E2E_WORKERS=<n>` pour forcer le nombre de workers (plafonné à 16).
  - `OPENCLAW_E2E_VERBOSE=1` pour réactiver la sortie console verbeuse.
- Portée :
  - Comportement end-to-end du Gateway multi-instances
  - Surfaces WebSocket/HTTP, appariement de nœuds et réseau plus lourd
- Attentes :
  - S'exécute en CI (quand activé dans le pipeline)
  - Aucune vraie clé requise
  - Plus de composants mobiles que les tests unitaires (peut être plus lent)

### Live (vrais fournisseurs + vrais modèles)

- Commande : `pnpm test:live`
- Configuration : `vitest.live.config.ts`
- Fichiers : `src/**/*.live.test.ts`
- Par défaut : **activé** par `pnpm test:live` (définit `OPENCLAW_LIVE_TEST=1`)
- Portée :
  - « Ce fournisseur/modèle fonctionne-t-il réellement _aujourd'hui_ avec de vrais identifiants ? »
  - Détecter les changements de format des fournisseurs, les particularités d'appel d'outils, les problèmes d'authentification et le comportement de limité de débit
- Attentes :
  - Non stable en CI par conception (vrais réseaux, vraies politiques de fournisseurs, quotas, pannes)
  - Coûte de l'argent / utilisé des limités de débit
  - Préférez exécuter des sous-ensembles restreints plutôt que « tout »
  - Les exécutions live sourced `~/.profile` pour récupérer les clés API manquantes
- Rotation des clés API (spécifique au fournisseur) : définissez `*_API_KEYS` avec le format virgule/point-virgule ou `*_API_KEY_1`, `*_API_KEY_2` (par exemple `OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`) ou remplacement par live via `OPENCLAW_LIVE_*_KEY` ; les tests réessaient sur les réponses de limité de débit.

## Quelle suite dois-je exécuter ?

Utilisez ce tableau de décision :

- Modification de logique/tests : exécutez `pnpm test` (et `pnpm test:coverage` si vous avez beaucoup modifié)
- Modification du réseau du Gateway / protocole WS / appariement : ajoutez `pnpm test:e2e`
- Débogage « mon bot est en panne » / échecs spécifiques au fournisseur / appel d'outils : exécutez un `pnpm test:live` restreint

## Live : smoke de modèle (clés de profil)

Les tests live sont divisés en deux couches pour isoler les échecs :

- « Modèle direct » nous dit que le fournisseur/modèle peut répondre du tout avec la clé donnée.
- « Smoke du Gateway » nous dit que le pipeline complet gateway+agent fonctionne pour ce modèle (sessions, historique, outils, politique de bac à sable, etc.).

### Couche 1 : Complétion directe du modèle (sans Gateway)

- Test : `src/agents/models.profiles.live.test.ts`
- Objectif :
  - Énumérer les modèles découverts
  - Utiliser `getApiKeyForModel` pour sélectionner les modèles pour lesquels vous avez des identifiants
  - Exécuter une petite complétion par modèle (et des régressions ciblées si nécessaire)
- Comment activer :
  - `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
- Définissez `OPENCLAW_LIVE_MODELS=modern` (ou `all`, alias pour modern) pour réellement exécuter cette suite ; sinon elle est ignorée pour garder `pnpm test:live` focalisé sur le smoke du Gateway
- Comment sélectionner les modèles :
  - `OPENCLAW_LIVE_MODELS=modern` pour exécuter l'allowlist moderne (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  - `OPENCLAW_LIVE_MODELS=all` est un alias pour l'allowlist moderne
  - ou `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (allowlist par virgule)
- Comment sélectionner les fournisseurs :
  - `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (allowlist par virgule)
- D'où viennent les clés :
  - Par défaut : magasin de profils et solutions de repli par env
  - Définissez `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` pour imposer **uniquement le magasin de profils**
- Pourquoi cela existe :
  - Sépare « l'API du fournisseur est cassée / la clé est invalide » de « le pipeline d'agent du Gateway est cassé »
  - Contient de petites régressions isolées (exemple : replay de raisonnement OpenAI Responses/Codex Responses + flux d'appels d'outils)

### Couche 2 : Smoke du Gateway + agent dev (ce que fait réellement « @openclaw »)

- Test : `src/gateway/gateway-models.profiles.live.test.ts`
- Objectif :
  - Démarrer un Gateway en processus
  - Créer/patcher une session `agent:dev:*` (remplacement de modèle par exécution)
  - Itérer les modèles-avec-clés et vérifier :
    - Réponse « significative » (sans outils)
    - Une vraie invocation d'outil fonctionne (sonde read)
    - Sondes d'outils supplémentaires optionnelles (sonde exec+read)
    - Les chemins de régression OpenAI (tool-call-only → follow-up) continuent de fonctionner
- Détails des sondes (pour expliquer rapidement les échecs) :
  - Sonde `read` : le test écrit un fichier nonce dans l'espace de travail et demande à l'agent de le `read` et de renvoyer le nonce.
  - Sonde `exec+read` : le test demande à l'agent d'`exec`-écrire un nonce dans un fichier temporaire, puis de le `read` en retour.
  - Sonde image : le test attache un PNG généré (chat + code aléatoire) et s'attend à ce que le modèle renvoie `cat <CODE>`.
  - Référence d'implémentation : `src/gateway/gateway-models.profiles.live.test.ts` et `src/gateway/live-image-probe.ts`.
- Comment activer :
  - `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
- Comment sélectionner les modèles :
  - Par défaut : allowlist moderne (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.1, Grok 4)
  - `OPENCLAW_LIVE_GATEWAY_MODELS=all` est un alias pour l'allowlist moderne
  - Ou définissez `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (ou liste par virgule) pour restreindre
- Comment sélectionner les fournisseurs (éviter « OpenRouter everything ») :
  - `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (allowlist par virgule)
- Les sondes d'outils + image sont toujours activés dans ce test live :
  - Sonde `read` + sonde `exec+read` (stress d'outils)
  - La sonde image s'exécute quand le modèle annonce le support d'entrée image
  - Flux (haut niveau) :
    - Le test génère un petit PNG avec « CAT » + code aléatoire (`src/gateway/live-image-probe.ts`)
    - L'envoie via `agent` `attachments: [{ mimeType: "image/png", content: "<base64>" }]`
    - Le Gateway analyse les pièces jointes en `images[]` (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
    - L'agent embarqué transmet un message utilisateur multimodal au modèle
    - Assertion : la réponse contient `cat` + le code (tolérance OCR : erreurs mineures acceptées)

Astuce : pour voir ce que vous pouvez tester sur votre machine (et les ids exacts `provider/model`), exécutez :

```bash
openclaw models list
openclaw models list --json
```

## Live : smoke de setup-token Anthropic

- Test : `src/agents/anthropic.setup-token.live.test.ts`
- Objectif : vérifier que le setup-token CLI Claude Code (ou un profil de setup-token collé) peut compléter un prompt Anthropic.
- Activer :
  - `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
  - `OPENCLAW_LIVE_SETUP_TOKEN=1`
- Sources de jeton (choisissez une) :
  - Profil : `OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
  - Jeton brut : `OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
- Remplacement de modèle (optionnel) :
  - `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

Exemple de configuration :

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## Live : smoke du backend CLI (Claude Code CLI ou autres CLI locaux)

- Test : `src/gateway/gateway-cli-backend.live.test.ts`
- Objectif : valider le pipeline Gateway + agent en utilisant un backend CLI local, sans toucher à votre configuration par défaut.
- Activer :
  - `pnpm test:live` (ou `OPENCLAW_LIVE_TEST=1` si vous invoquez Vitest directement)
  - `OPENCLAW_LIVE_CLI_BACKEND=1`
- Par défaut :
  - Modèle : `claude-cli/claude-sonnet-4-6`
  - Commande : `claude`
  - Args : `["-p","--output-format","json","--dangerously-skip-permissions"]`
- Remplacements (optionnels) :
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
  - `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.3-codex"`
  - `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
  - `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
  - `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` pour envoyer une vraie pièce jointe image (les chemins sont injectés dans le prompt).
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` pour passer les chemins de fichiers image comme arguments CLI au lieu de l'injection dans le prompt.
  - `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (ou `"list"`) pour contrôler comment les arguments image sont passés quand `IMAGE_ARG` est défini.
  - `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` pour envoyer un second tour et valider le flux de reprise.
- `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` pour garder la configuration MCP du CLI Claude Code activée (par défaut, désactive la configuration MCP avec un fichier vide temporaire).

Exemple :

```bash
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### Recettes live recommandées

Les allowlists étroites et explicites sont les plus rapides et les moins instables :

- Modèle unique, direct (sans Gateway) :
  - `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`

- Modèle unique, smoke du Gateway :
  - `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Appel d'outils sur plusieurs fournisseurs :
  - `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

- Focus Google (clé API Gemini + Antigravity) :
  - Gemini (clé API) : `OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
  - Antigravity (OAuth) : `OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

Notes :

- `google/...` utilisé l'API Gemini (clé API).
- `google-antigravity/...` utilisé le pont OAuth Antigravity (point d'accès d'agent de type Cloud Code Assist).
- `google-gemini-cli/...` utilisé le CLI Gemini local sur votre machine (authentification séparée + particularités d'outillage).
- API Gemini vs CLI Gemini :
  - API : OpenClaw appelle l'API Gemini hébergée de Google via HTTP (clé API / auth de profil) ; c'est ce que la plupart des utilisateurs entendent par « Gemini ».
  - CLI : OpenClaw exécute un binaire `gemini` local ; il a sa propre authentification et peut se comporter différemment (streaming/support d'outils/décalage de version).

## Live : matrice de modèles (ce que nous couvrons)

Il n'y a pas de « liste de modèles CI » fixe (le live est opt-in), mais voici les modèles **recommandés** à couvrir régulièrement sur une machine de développement avec des clés.

### Ensemble smoke moderne (appel d'outils + image)

C'est l'exécution « modèles courants » que nous nous attendons à maintenir fonctionnelle :

- OpenAI (non-Codex) : `openai/gpt-5.2` (optionnel : `openai/gpt-5.1`)
- OpenAI Codex : `openai-codex/gpt-5.3-codex` (optionnel : `openai-codex/gpt-5.3-codex-codex`)
- Anthropic : `anthropic/claude-opus-4-6` (ou `anthropic/claude-sonnet-4-5`)
- Google (API Gemini) : `google/gemini-3-pro-preview` et `google/gemini-3-flash-preview` (éviter les anciens modèles Gemini 2.x)
- Google (Antigravity) : `google-antigravity/claude-opus-4-6-thinking` et `google-antigravity/gemini-3-flash`
- Z.AI (GLM) : `zai/glm-4.7`
- MiniMax : `minimax/minimax-m2.1`

Exécuter le smoke du Gateway avec outils + image :
`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.3-codex,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.1" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### Base : appel d'outils (Read + Exec optionnel)

Choisissez au moins un par famille de fournisseurs :

- OpenAI : `openai/gpt-5.2` (ou `openai/gpt-5-mini`)
- Anthropic : `anthropic/claude-opus-4-6` (ou `anthropic/claude-sonnet-4-5`)
- Google : `google/gemini-3-flash-preview` (ou `google/gemini-3-pro-preview`)
- Z.AI (GLM) : `zai/glm-4.7`
- MiniMax : `minimax/minimax-m2.1`

Couverture supplémentaire optionnelle (souhaitable) :

- xAI : `xai/grok-4` (ou le dernier disponible)
- Mistral : `mistral/`... (choisissez un modèle compatible « tools » que vous avez activé)
- Cerebras : `cerebras/`... (si vous avez accès)
- LM Studio : `lmstudio/`... (local ; l'appel d'outils dépend du mode API)

### Vision : envoi d'image (pièce jointe → message multimodal)

Incluez au moins un modèle capable d'image dans `OPENCLAW_LIVE_GATEWAY_MODELS` (variantes compatibles vision Claude/Gemini/OpenAI, etc.) pour exercer la sonde image.

### Agrégateurs / gateways alternatifs

Si vous avez des clés activées, nous supportons aussi les tests via :

- OpenRouter : `openrouter/...` (des centaines de modèles ; utilisez `openclaw models scan` pour trouver des candidats compatibles outils+image)
- OpenCode Zen : `opencode/...` (auth via `OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY`)

Plus de fournisseurs que vous pouvez inclure dans la matrice live (si vous avez des identifiants/configuration) :

- Intégrés : `openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
- Via `models.providers` (points d'accès personnalisés) : `minimax` (cloud/API), plus tout proxy compatible OpenAI/Anthropic (LM Studio, vLLM, LiteLLM, etc.)

Astuce : n'essayez pas de coder en dur « tous les modèles » dans la documentation. La liste faisant autorité est ce que `discoverModels(...)` renvoie sur votre machine + les clés disponibles.

## Identifiants (ne jamais commiter)

Les tests live découvrent les identifiants de la même façon que le CLI. Implications pratiques :

- Si le CLI fonctionne, les tests live devraient trouver les mêmes clés.
- Si un test live dit « no creds », déboguez de la même façon que vous débogueriez `openclaw models list` / la sélection de modèle.

- Magasin de profils : `~/.openclaw/credentials/` (préféré ; ce que signifie « clés de profil » dans les tests)
- Configuration : `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`)

Si vous voulez vous appuyer sur les clés d'environnement (ex. exportées dans votre `~/.profile`), exécutez les tests locaux après `source ~/.profile`, ou utilisez les exécuteurs Docker ci-dessous (ils peuvent monter `~/.profile` dans le conteneur).

## Live Deepgram (transcription audio)

- Test : `src/media-understanding/providers/deepgram/audio.live.test.ts`
- Activer : `DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## Live plan de codage BytePlus

- Test : `src/agents/byteplus.live.test.ts`
- Activer : `BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
- Remplacement de modèle optionnel : `BYTEPLUS_CODING_MODEL=ark-code-latest`

## Exécuteurs Docker (vérifications optionnelles « fonctionne sous Linux »)

Ceux-ci exécutent `pnpm test:live` dans l'image Docker du dépôt, en montant votre répertoire de configuration local et votre espace de travail (et en sourçant `~/.profile` s'il est monté) :

- Modèles directs : `pnpm test:docker:live-models` (script : `scripts/test-live-models-docker.sh`)
- Gateway + agent dev : `pnpm test:docker:live-gateway` (script : `scripts/test-live-gateway-models-docker.sh`)
- Assistant de configuration initiale (TTY, scaffolding complet) : `pnpm test:docker:onboard` (script : `scripts/e2e/onboard-docker.sh`)
- Réseau du Gateway (deux conteneurs, auth WS + santé) : `pnpm test:docker:gateway-network` (script : `scripts/e2e/gateway-network-docker.sh`)
- Plugins (chargement d'extension personnalisée + smoke du registre) : `pnpm test:docker:plugins` (script : `scripts/e2e/plugins-docker.sh`)

Smoke test ACP plain-language thread manuel (hors CI) :

- `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
- Conservez ce script pour les workflows de régression/débogage. Il peut être nécessaire de nouveau pour la validation du routage de threads ACP ; ne le supprimez pas.

Variables d'environnement utiles :

- `OPENCLAW_CONFIG_DIR=...` (par défaut : `~/.openclaw`) monté sur `/home/node/.openclaw`
- `OPENCLAW_WORKSPACE_DIR=...` (par défaut : `~/.openclaw/workspace`) monté sur `/home/node/.openclaw/workspace`
- `OPENCLAW_PROFILE_FILE=...` (par défaut : `~/.profile`) monté sur `/home/node/.profile` et sourcé avant d'exécuter les tests
- `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` pour restreindre l'exécution
- `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` pour s'assurer que les identifiants proviennent du magasin de profils (pas de l'env)

## Vérification de la documentation

Exécutez les vérifications de documentation après les modifications de docs : `pnpm docs:list`.

## Régression hors ligne (sûre pour la CI)

Ce sont des régressions de « vrai pipeline » sans vrais fournisseurs :

- Appel d'outils du Gateway (mock OpenAI, vrai gateway + boucle d'agent) : `src/gateway/gateway.test.ts` (cas : "runs a mock OpenAI tool call end-to-end via gateway agent loop")
- Assistant du Gateway (WS `wizard.start`/`wizard.next`, écrit la configuration + auth imposée) : `src/gateway/gateway.test.ts` (cas : "runs wizard over ws and writes auth token config")

## Évaluations de fiabilité d'agent (Skills)

Nous avons déjà quelques tests sûrs pour la CI qui se comportent comme des « évaluations de fiabilité d'agent » :

- Appels d'outils simulés à travers la vraie boucle gateway + agent (`src/gateway/gateway.test.ts`).
- Flux d'assistant end-to-end qui valident le câblage de session et les effets de configuration (`src/gateway/gateway.test.ts`).

Ce qui manque encore pour les Skills (voir [Skills](/tools/skills)) :

- **Prise de décision :** quand les Skills sont listées dans le prompt, l'agent choisit-il la bonne Skill (ou évite-t-il les non pertinentes) ?
- **Conformité :** l'agent lit-il `SKILL.md` avant utilisation et suit-il les étapes/arguments requis ?
- **Contrats de flux de travail :** scénarios multi-tours qui vérifient l'ordre des appels d'outils, le report de l'historique de session et les limités du bac à sable.

Les futures évaluations devraient rester déterministes en priorité :

- Un exécuteur de scénarios utilisant des fournisseurs simulés pour vérifier les appels d'outils + l'ordre, les lectures de fichiers de Skills et le câblage de session.
- Une petite suite de scénarios centrés sur les Skills (utilisation vs évitement, filtrage, injection de prompt).
- Évaluations live optionnelles (opt-in, conditionnées par env) uniquement après que la suite sûre pour la CI est en place.

## Ajout de régressions (recommandations)

Quand vous corrigez un problème de fournisseur/modèle découvert en live :

- Ajoutez une régression sûre pour la CI si possible (fournisseur mock/stub, ou capturez la transformation exacte de la forme de requête)
- Si c'est intrinsèquement live uniquement (limités de débit, politiques d'auth), gardez le test live restreint et opt-in via variables d'environnement
- Préférez cibler la plus petite couche qui attrape le bug :
  - Bug de conversion/replay de requête du fournisseur → test de modèles directs
  - Bug du pipeline session/historique/outils du Gateway → smoke live du Gateway ou test mock du Gateway sûr pour la CI
