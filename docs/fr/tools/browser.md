---
summary: "Service de contrôle de navigateur intégré + commandes d'action"
read_when:
  - Ajout d'automatisation de navigateur contrôlée par l'agent
  - Débogage de l'interférence d'openclaw avec votre propre Chrome
  - Implémentation des paramètres et du cycle de vie du navigateur dans l'application macOS
title: "Navigateur (géré par OpenClaw)"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/browser.md
  workflow: manual
---

# Navigateur (géré par openclaw)

OpenClaw peut exécuter un **profil Chrome/Brave/Edge/Chromium dédié** que l'agent contrôle.
Il est isolé de votre navigateur personnel et est géré via un petit service de contrôle local à l'intérieur du Gateway (loopback uniquement).

Vue débutant :

- Considérez-le comme un **navigateur séparé, réservé à l'agent**.
- Le profil `openclaw` ne **touche pas** votre profil de navigateur personnel.
- L'agent peut **ouvrir des onglets, lire des pages, cliquer et taper** dans une voie sûre.
- Le profil `chrome` par défaut utilisé le **navigateur Chromium par défaut du système** via le relais d'extension ; passez à `openclaw` pour le navigateur géré isolé.

## Ce que vous obtenez

- Un profil de navigateur séparé nommé **openclaw** (accent orange par défaut).
- Contrôle déterministe des onglets (lister/ouvrir/focus/fermer).
- Actions de l'agent (cliquer/taper/glisser/sélectionner), snapshots, captures d'écran, PDFs.
- Support multi-profils optionnel (`openclaw`, `work`, `remote`, ...).

Ce navigateur n'est **pas** votre navigateur quotidien. C'est une surface sûre et isolée pour l'automatisation et la vérification par l'agent.

## Démarrage rapide

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Si vous obtenez « Browser disabled », activez-le dans la configuration (voir ci-dessous) et redémarrez le Gateway.

## Profils : `openclaw` vs `chrome`

- `openclaw` : navigateur géré et isolé (aucune extension requise).
- `chrome` : relais d'extension vers votre **navigateur système** (nécessite que l'extension OpenClaw soit attachée à un onglet).

Définissez `browser.defaultProfile: "openclaw"` si vous voulez le mode gère par défaut.

## Configuration

Les paramètres du navigateur sont dans `~/.openclaw/openclaw.json`.

```json5
{
  browser: {
    enabled: true, // défaut : true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // mode réseau de confiance par défaut
      // allowPrivateNetwork: true, // alias legacy
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // override legacy mono-profil
    remoteCdpTimeoutMs: 1500, // timeout HTTP CDP distant (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // timeout handshake WebSocket CDP distant (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

Notes :

- Le service de contrôle du navigateur se lie au loopback sur un port dérivé de `gateway.port` (défaut : `18791`, soit gateway + 2). Le relais utilisé le port suivant (`18792`).
- Si vous remplacez le port du Gateway (`gateway.port` ou `OPENCLAW_GATEWAY_PORT`), les ports dérivés du navigateur se décalent pour rester dans la même « famille ».
- `cdpUrl` est par défaut le port du relais quand non défini.
- `remoteCdpTimeoutMs` s'applique aux vérifications d'accessibilité CDP distant (non-loopback).
- `remoteCdpHandshakeTimeoutMs` s'applique aux vérifications d'accessibilité WebSocket CDP distant.
- La navigation/ouverture d'onglet du navigateur est protégée contre le SSRF avant la navigation et re-vérifiée au mieux sur l'URL finale `http(s)` après navigation.
- `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` est par défaut `true` (modèle de réseau de confiance). Définissez-le à `false` pour une navigation strictement publique.
- `browser.ssrfPolicy.allowPrivateNetwork` reste supporté comme alias legacy pour la compatibilité.
- `attachOnly: true` signifie « ne jamais lancer de navigateur local ; s'attacher uniquement s'il est déjà en cours d'exécution. »
- `color` + `color` par profil teintent l'interface du navigateur pour que vous puissiez voir quel profil est actif.
- Le profil par défaut est `chrome` (relais d'extension). Utilisez `defaultProfile: "openclaw"` pour le navigateur géré.
- Ordre de détection automatique : navigateur par défaut du système s'il est basé sur Chromium ; sinon Chrome → Brave → Edge → Chromium → Chrome Canary.
- Les profils locaux `openclaw` auto-assignent `cdpPort`/`cdpUrl` — ne les définissez que pour le CDP distant.

## Utiliser Brave (ou un autre navigateur basé sur Chromium)

Si votre navigateur **par défaut du système** est basé sur Chromium (Chrome/Brave/Edge/etc), OpenClaw l'utilisé automatiquement. Définissez `browser.executablePath` pour remplacer la détection automatique :

Exemple CLI :

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```json5
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## Contrôle local vs distant

- **Contrôle local (défaut) :** le Gateway démarre le service de contrôle en loopback et peut lancer un navigateur local.
- **Contrôle distant (node host) :** exécutez un node host sur la machine qui a le navigateur ; le Gateway redirige les actions du navigateur via un proxy vers celui-ci.
- **CDP distant :** définissez `browser.profiles.<name>.cdpUrl` (ou `browser.cdpUrl`) pour s'attacher à un navigateur basé sur Chromium distant. Dans ce cas, OpenClaw ne lancera pas de navigateur local.

Les URLs CDP distantes peuvent inclure l'authentification :

- Tokens en paramètre de requête (par ex. `https://provider.example?token=<token>`)
- Auth HTTP Basic (par ex. `https://user:pass@provider.example`)

OpenClaw préserve l'authentification lors des appels aux endpoints `/json/*` et lors de la connexion au WebSocket CDP. Préférez les variables d'environnement ou les gestionnaires de secrets pour les tokens plutôt que de les committer dans les fichiers de configuration.

## Proxy navigateur node (zéro configuration par défaut)

Si vous exécutez un **node host** sur la machine qui a votre navigateur, OpenClaw peut auto-router les appels d'outils du navigateur vers ce node sans configuration supplémentaire du navigateur. C'est le chemin par défaut pour les gateways distants.

Notes :

- Le node host expose son serveur de contrôle de navigateur local via une **commande proxy**.
- Les profils viennent de la propre configuration `browser.profiles` du node (identique au local).
- Désactivez si vous ne le voulez pas :
  - Sur le node : `nodeHost.browserProxy.enabled=false`
  - Sur le gateway : `gateway.nodes.browser.mode="off"`

## Browserless (CDP distant hébergé)

[Browserless](https://browserless.io) est un service Chromium hébergé qui expose des endpoints CDP via HTTPS. Vous pouvez pointer un profil de navigateur OpenClaw vers un endpoint de région Browserless et vous authentifier avec votre clé API.

Exemple :

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

Notes :

- Remplacez `<BROWSERLESS_API_KEY>` par votre vrai token Browserless.
- Choisissez l'endpoint de région qui correspond à votre compte Browserless (voir leur documentation).

## Sécurité

Idées clés :

- Le contrôle du navigateur est en loopback uniquement ; l'accès passe par l'auth du Gateway ou le couplage node.
- Si le contrôle du navigateur est activé et qu'aucune auth n'est configurée, OpenClaw auto-génère `gateway.auth.token` au démarrage et le persiste dans la configuration.
- Gardez le Gateway et les node hosts sur un réseau privé (Tailscale) ; évitez l'exposition publique.
- Traitez les URLs/tokens CDP distants comme des secrets ; préférez les variables d'environnement ou un gestionnaire de secrets.

Conseils pour le CDP distant :

- Préférez les endpoints HTTPS et les tokens à courte durée de vie quand possible.
- Évitez d'intégrer des tokens à longue durée de vie directement dans les fichiers de configuration.

## Profils (multi-navigateur)

OpenClaw supporte plusieurs profils nommés (configurations de routage). Les profils peuvent être :

- **Géré par openclaw** : une instance de navigateur Chromium dédiée avec son propre répertoire de données utilisateur + port CDP
- **Distant** : une URL CDP explicite (navigateur Chromium en cours d'exécution ailleurs)
- **Relais d'extension** : vos onglets Chrome existants via le relais local + l'extension Chrome

Défauts :

- Le profil `openclaw` est auto-créé s'il est manquant.
- Le profil `chrome` est intégré pour le relais d'extension Chrome (pointe vers `http://127.0.0.1:18792` par défaut).
- Les ports CDP locaux s'allouent depuis **18800–18899** par défaut.
- Supprimer un profil déplace son répertoire de données local dans la Corbeille.

Tous les endpoints de contrôle acceptent `?profile=<name>` ; le CLI utilisé `--browser-profile`.

## Relais d'extension Chrome (utiliser votre Chrome existant)

OpenClaw peut aussi piloter **vos onglets Chrome existants** (pas d'instance Chrome « openclaw » séparée) via un relais CDP local + une extension Chrome.

Guide complet : [Extension Chrome](/tools/chrome-extension)

Flux :

- Le Gateway s'exécute localement (même machine) ou un node host s'exécute sur la machine du navigateur.
- Un **serveur relais** local écoute sur une `cdpUrl` en loopback (défaut : `http://127.0.0.1:18792`).
- Vous cliquez sur l'icône de l'extension **OpenClaw Browser Relay** sur un onglet pour l'attacher (elle ne s'attache pas automatiquement).
- L'agent contrôle cet onglet via l'outil `browser` normal, en sélectionnant le bon profil.

Si le Gateway s'exécute ailleurs, exécutez un node host sur la machine du navigateur pour que le Gateway puisse rediriger les actions du navigateur via un proxy.

### Sessions sandboxées

Si la session de l'agent est sandboxée, l'outil `browser` peut utiliser par défaut `target="sandbox"` (navigateur du bac à sable).
La prise de contrôle du relais d'extension Chrome nécessite le contrôle du navigateur hôte, donc soit :

- exécutez la session sans bac à sable, soit
- définissez `agents.defaults.sandbox.browser.allowHostControl: true` et utilisez `target="host"` lors de l'appel de l'outil.

### Installation

1. Charger l'extension (dev/non packagée) :

```bash
openclaw browser extension install
```

- Chrome → `chrome://extensions` → activer le « Mode développeur »
- « Charger l'extension non empaquetée » → sélectionner le répertoire affiché par `openclaw browser extension path`
- Épingler l'extension, puis cliquer dessus sur l'onglet que vous voulez contrôler (le badge affiche `ON`).

2. L'utiliser :

- CLI : `openclaw browser --browser-profile chrome tabs`
- Outil de l'agent : `browser` avec `profile="chrome"`

Optionnel : si vous voulez un nom ou un port relais différent, créez votre propre profil :

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Notes :

- Ce mode s'appuie sur Playwright-on-CDP pour la plupart des opérations (captures d'écran/snapshots/actions).
- Détachez en cliquant à nouveau sur l'icône de l'extension.

## Garanties d'isolation

- **Répertoire de données utilisateur dédié** : ne touche jamais votre profil de navigateur personnel.
- **Ports dédiés** : évite `9222` pour prévenir les collisions avec les workflows de développement.
- **Contrôle d'onglets déterministe** : cible les onglets par `targetId`, pas par « dernier onglet ».

## Sélection du navigateur

Au lancement local, OpenClaw choisit le premier disponible :

1. Chrome
2. Brave
3. Edge
4. Chromium
5. Chrome Canary

Vous pouvez remplacer avec `browser.executablePath`.

Plateformes :

- macOS : vérifie `/Applications` et `~/Applications`.
- Linux : cherche `google-chrome`, `brave`, `microsoft-edge`, `chromium`, etc.
- Windows : vérifie les emplacements d'installation courants.

## API de contrôle (optionnelle)

Pour les intégrations locales uniquement, le Gateway expose une petite API HTTP en loopback :

- Statut/démarrer/arrêter : `GET /`, `POST /start`, `POST /stop`
- Onglets : `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
- Snapshot/capture d'écran : `GET /snapshot`, `POST /screenshot`
- Actions : `POST /navigate`, `POST /act`
- Hooks : `POST /hooks/file-chooser`, `POST /hooks/dialog`
- Téléchargements : `POST /download`, `POST /wait/download`
- Débogage : `GET /console`, `POST /pdf`
- Débogage : `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
- Réseau : `POST /response/body`
- État : `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
- État : `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
- Paramètres : `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/média`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Tous les endpoints acceptent `?profile=<name>`.

Si l'auth du gateway est configurée, les routes HTTP du navigateur nécessitent aussi l'auth :

- `Authorization: Bearer <gateway token>`
- `x-openclaw-password: <gateway password>` ou auth HTTP Basic avec ce mot de passe

### Prérequis Playwright

Certaines fonctionnalités (navigate/act/AI snapshot/rôle snapshot, captures d'écran d'éléments, PDF) nécessitent Playwright. Si Playwright n'est pas installé, ces endpoints retournent une erreur 501 claire. Les snapshots ARIA et les captures d'écran basiques fonctionnent toujours pour le Chrome géré par openclaw.
Pour le driver relais d'extension Chrome, les snapshots ARIA et les captures d'écran nécessitent Playwright.

Si vous voyez `Playwright is not available in this gateway build`, installez le package Playwright complet (pas `playwright-core`) et redémarrez le gateway, ou réinstallez OpenClaw avec le support navigateur.

#### Installation Docker de Playwright

Si votre Gateway s'exécute dans Docker, évitez `npx playwright` (conflits de surcharge npm). Utilisez plutôt le CLI fourni :

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Pour persister les téléchargements de navigateurs, définissez `PLAYWRIGHT_BROWSERS_PATH` (par exemple, `/home/node/.cache/ms-playwright`) et assurez-vous que `/home/node` est persisté via `OPENCLAW_HOME_VOLUME` ou un montage bind. Voir [Docker](/install/docker).

## Comment ça fonctionne (interne)

Flux de haut niveau :

- Un petit **serveur de contrôle** accepte les requêtes HTTP.
- Il se connecte aux navigateurs basés sur Chromium (Chrome/Brave/Edge/Chromium) via **CDP**.
- Pour les actions avancées (cliquer/taper/snapshot/PDF), il utilise **Playwright** par-dessus CDP.
- Quand Playwright est manquant, seules les opérations sans Playwright sont disponibles.

Ce design garde l'agent sur une interface stable et déterministe tout en vous permettant de permuter entre navigateurs et profils locaux/distants.

## Référence rapide CLI

Toutes les commandes acceptent `--browser-profile <name>` pour cibler un profil spécifique.
Toutes les commandes acceptent aussi `--json` pour une sortie lisible par machine (payloads stables).

Basiques :

- `openclaw browser status`
- `openclaw browser start`
- `openclaw browser stop`
- `openclaw browser tabs`
- `openclaw browser tab`
- `openclaw browser tab new`
- `openclaw browser tab select 2`
- `openclaw browser tab close 2`
- `openclaw browser open https://example.com`
- `openclaw browser focus abcd1234`
- `openclaw browser close abcd1234`

Inspection :

- `openclaw browser screenshot`
- `openclaw browser screenshot --full-page`
- `openclaw browser screenshot --ref 12`
- `openclaw browser screenshot --ref e12`
- `openclaw browser snapshot`
- `openclaw browser snapshot --format aria --limit 200`
- `openclaw browser snapshot --interactive --compact --depth 6`
- `openclaw browser snapshot --efficient`
- `openclaw browser snapshot --labels`
- `openclaw browser snapshot --selector "#main" --interactive`
- `openclaw browser snapshot --frame "iframe#main" --interactive`
- `openclaw browser console --level error`
- `openclaw browser errors --clear`
- `openclaw browser requests --filter api --clear`
- `openclaw browser pdf`
- `openclaw browser responsebody "**/api" --max-chars 5000`

Actions :

- `openclaw browser navigate https://example.com`
- `openclaw browser resize 1280 720`
- `openclaw browser click 12 --double`
- `openclaw browser click e12 --double`
- `openclaw browser type 23 "hello" --submit`
- `openclaw browser press Enter`
- `openclaw browser hover 44`
- `openclaw browser scrollintoview e12`
- `openclaw browser drag 10 11`
- `openclaw browser select 9 OptionA OptionB`
- `openclaw browser download e12 report.pdf`
- `openclaw browser waitfordownload report.pdf`
- `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
- `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
- `openclaw browser dialog --accept`
- `openclaw browser wait --text "Done"`
- `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
- `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
- `openclaw browser highlight e12`
- `openclaw browser trace start`
- `openclaw browser trace stop`

État :

- `openclaw browser cookies`
- `openclaw browser cookies set session abc123 --url "https://example.com"`
- `openclaw browser cookies clear`
- `openclaw browser storage local get`
- `openclaw browser storage local set theme dark`
- `openclaw browser storage session clear`
- `openclaw browser set offline on`
- `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
- `openclaw browser set credentials user pass`
- `openclaw browser set credentials --clear`
- `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
- `openclaw browser set geo --clear`
- `openclaw browser set média dark`
- `openclaw browser set timezone America/New_York`
- `openclaw browser set locale en-US`
- `openclaw browser set device "iPhone 14"`

Notes :

- `upload` et `dialog` sont des appels **d'armement** ; exécutez-les avant le clic/appui de touche qui déclenche le sélecteur/dialogue.
- Les chemins de sortie des téléchargements et traces sont contraints aux racines temporaires OpenClaw :
  - traces : `/tmp/openclaw` (repli : `${os.tmpdir()}/openclaw`)
  - téléchargements : `/tmp/openclaw/downloads` (repli : `${os.tmpdir()}/openclaw/downloads`)
- Les chemins d'upload sont contraints à une racine temporaire d'uploads OpenClaw :
  - uploads : `/tmp/openclaw/uploads` (repli : `${os.tmpdir()}/openclaw/uploads`)
- `upload` peut aussi définir des entrées fichier directement via `--input-ref` ou `--element`.
- `snapshot` :
  - `--format ai` (défaut quand Playwright est installé) : retourne un snapshot IA avec des refs numériques (`aria-ref="<n>"`).
  - `--format aria` : retourne l'arbre d'accessibilité (pas de refs ; inspection uniquement).
  - `--efficient` (ou `--mode efficient`) : preset compact de rôle snapshot (interactive + compact + depth + maxChars réduit).
  - Défaut de configuration (outil/CLI uniquement) : définissez `browser.snapshotDefaults.mode: "efficient"` pour utiliser les snapshots efficaces quand l'appelant ne passe pas de mode (voir [Configuration du Gateway](/gateway/configuration#browser-openclaw-managed-browser)).
  - Les options de rôle snapshot (`--interactive`, `--compact`, `--depth`, `--selector`) forcent un snapshot basé sur les rôles avec des refs comme `ref=e12`.
  - `--frame "<sélecteur iframe>"` scope les rôle snapshots à un iframe (se combine avec les rôle refs comme `e12`).
  - `--interactive` produit une liste plate et facile à sélectionner des éléments interactifs (idéal pour piloter des actions).
  - `--labels` ajouté une capture d'écran du viewport uniquement avec des labels ref superposés (affiche `MEDIA:<path>`).
- `click`/`type`/etc nécessitent un `ref` du `snapshot` (soit numérique `12` soit rôle ref `e12`). Les sélecteurs CSS ne sont intentionnellement pas supportés pour les actions.

## Snapshots et refs

OpenClaw supporte deux styles de « snapshot » :

- **Snapshot IA (refs numériques)** : `openclaw browser snapshot` (défaut ; `--format ai`)
  - Sortie : un snapshot texte qui inclut des refs numériques.
  - Actions : `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
  - En interne, le ref est résolu via le `aria-ref` de Playwright.

- **Rôle snapshot (rôle refs comme `e12`)** : `openclaw browser snapshot --interactive` (ou `--compact`, `--depth`, `--selector`, `--frame`)
  - Sortie : une liste/arbre basé sur les rôles avec `[ref=e12]` (et optionnellement `[nth=1]`).
  - Actions : `openclaw browser click e12`, `openclaw browser highlight e12`.
  - En interne, le ref est résolu via `getByRole(...)` (plus `nth()` pour les doublons).
  - Ajoutez `--labels` pour inclure une capture d'écran du viewport avec des labels `e12` superposés.

Comportement des refs :

- Les refs ne sont **pas stables entre les navigations** ; si quelque chose échoue, relancez `snapshot` et utilisez un ref frais.
- Si le rôle snapshot a été pris avec `--frame`, les rôle refs sont scopés à cet iframe jusqu'au prochain rôle snapshot.

## Power-ups d'attente

Vous pouvez attendre plus que du temps/texte :

- Attendre une URL (globs supportés par Playwright) :
  - `openclaw browser wait --url "**/dash"`
- Attendre un état de chargement :
  - `openclaw browser wait --load networkidle`
- Attendre un prédicat JS :
  - `openclaw browser wait --fn "window.ready===true"`
- Attendre qu'un sélecteur devienne visible :
  - `openclaw browser wait "#main"`

Ceux-ci peuvent être combinés :

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## Workflows de débogage

Quand une action échoue (par ex. « not visible », « strict mode violation », « covered ») :

1. `openclaw browser snapshot --interactive`
2. Utilisez `click <ref>` / `type <ref>` (préférez les rôle refs en mode interactif)
3. Si ça échoue encore : `openclaw browser highlight <ref>` pour voir ce que Playwright cible
4. Si la page se comporte étrangement :
   - `openclaw browser errors --clear`
   - `openclaw browser requests --filter api --clear`
5. Pour un débogage approfondi : enregistrez une trace :
   - `openclaw browser trace start`
   - reproduisez le problème
   - `openclaw browser trace stop` (affiche `TRACE:<path>`)

## Sortie JSON

`--json` est pour le scripting et l'outillage structuré.

Exemples :

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Les rôle snapshots en JSON incluent `refs` plus un petit bloc `stats` (lignes/caractères/refs/interactifs) pour que les outils puissent raisonner sur la taille et la densité du payload.

## État et variables d'environnement

Ceux-ci sont utiles pour les workflows « faire en sorte que le site se comporte comme X » :

- Cookies : `cookies`, `cookies set`, `cookies clear`
- Stockage : `storage local|session get|set|clear`
- Hors ligne : `set offline on|off`
- En-têtes : `set headers --headers-json '{"X-Debug":"1"}'` (legacy `set headers --json '{"X-Debug":"1"}'` reste supporté)
- Auth HTTP basic : `set credentials user pass` (ou `--clear`)
- Géolocalisation : `set geo <lat> <lon> --origin "https://example.com"` (ou `--clear`)
- Média : `set média dark|light|no-préférence|none`
- Fuseau horaire / locale : `set timezone ...`, `set locale ...`
- Appareil / viewport :
  - `set device "iPhone 14"` (presets d'appareils Playwright)
  - `set viewport 1280 720`

## Sécurité & vie privée

- Le profil de navigateur openclaw peut contenir des sessions connectées ; traitez-le comme sensible.
- `browser act kind=evaluate` / `openclaw browser evaluate` et `wait --fn` exécutent du JavaScript arbitraire dans le contexte de la page. L'injection de prompt peut orienter cela. Désactivez-le avec `browser.evaluateEnabled=false` si vous n'en avez pas besoin.
- Pour les connexions et notes anti-bot (X/Twitter, etc.), voir [Connexion navigateur + publication X/Twitter](/tools/browser-login).
- Gardez le Gateway/node host privé (loopback ou tailnet uniquement).
- Les endpoints CDP distants sont puissants ; faites-les passer par un tunnel et protégez-les.

Exemple de mode strict (bloquer les destinations privées/internes par défaut) :

```json5
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // autorisation exacte optionnelle
    },
  },
}
```

## Dépannage

Pour les problèmes spécifiques à Linux (notamment le Chromium snap), voir
[Dépannage du navigateur](/tools/browser-linux-troubleshooting).

## Outils de l'agent + fonctionnement du contrôle

L'agent obtient **un outil** pour l'automatisation du navigateur :

- `browser` — status/start/stop/tabs/open/focus/close/snapshot/screenshot/navigate/act

Comment ça se mappe :

- `browser snapshot` retourne un arbre UI stable (IA ou ARIA).
- `browser act` utilisé les IDs `ref` du snapshot pour cliquer/taper/glisser/sélectionner.
- `browser screenshot` capture les pixels (page entière ou élément).
- `browser` accepte :
  - `profile` pour choisir un profil de navigateur nommé (openclaw, chrome, ou CDP distant).
  - `target` (`sandbox` | `host` | `node`) pour sélectionner où le navigateur réside.
  - Dans les sessions sandboxées, `target: "host"` nécessite `agents.defaults.sandbox.browser.allowHostControl=true`.
  - Si `target` est omis : les sessions sandboxées utilisent `sandbox` par défaut, les sessions non-sandbox utilisent `host` par défaut.
  - Si un node compatible navigateur est connecté, l'outil peut auto-router vers celui-ci sauf si vous fixez `target="host"` ou `target="node"`.

Cela garde l'agent déterministe et évite les sélecteurs fragiles.
