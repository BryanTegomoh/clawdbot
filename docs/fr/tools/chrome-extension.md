---
summary: "Extension Chrome : laisser OpenClaw piloter votre onglet Chrome existant"
read_when:
  - Vous souhaitez que l'agent pilote un onglet Chrome existant (bouton de barre d'outils)
  - Vous avez besoin d'un Gateway distant + automatisation du navigateur local via Tailscale
  - Vous souhaitez comprendre les implications de sécurité de la prise de contrôle du navigateur
title: "Extension Chrome"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/chrome-extension.md
  workflow: manual
---

# Extension Chrome (relais navigateur)

L'extension Chrome OpenClaw permet à l'agent de contrôler vos **onglets Chrome existants** (votre fenêtre Chrome normale) au lieu de lancer un profil Chrome géré par openclaw séparé.

L'attachement/détachement se fait via un **simple bouton de barre d'outils Chrome**.

## Concept

Il y a trois composants :

- **Service de contrôle du navigateur** (Gateway ou node) : l'API que l'agent/outil appelle (via le Gateway)
- **Serveur relais local** (CDP loopback) : fait le pont entre le serveur de contrôle et l'extension (`http://127.0.0.1:18792` par défaut)
- **Extension Chrome MV3** : s'attache à l'onglet actif via `chrome.debugger` et transmet les messages CDP au relais

OpenClaw contrôle ensuite l'onglet attaché via la surface d'outil `browser` normale (en sélectionnant le bon profil).

## Installation / chargement (non empaqueté)

1. Installez l'extension dans un chemin local stable :

```bash
openclaw browser extension install
```

2. Affichez le chemin du répertoire de l'extension installée :

```bash
openclaw browser extension path
```

3. Chrome → `chrome://extensions`

- Activez le « Mode développeur »
- « Charger l'extension non empaquetée » → sélectionnez le répertoire affiché ci-dessus

4. Épinglez l'extension.

## Mises à jour (pas d'étape de build)

L'extension est livrée dans la release OpenClaw (paquet npm) sous forme de fichiers statiques. Il n'y a pas d'étape de « build » séparée.

Après la mise à jour d'OpenClaw :

- Relancez `openclaw browser extension install` pour rafraîchir les fichiers installés sous votre répertoire d'état OpenClaw.
- Chrome → `chrome://extensions` → cliquez sur « Recharger » sur l'extension.

## Utilisation (configurer le token gateway une fois)

OpenClaw inclut un profil navigateur intégré nommé `chrome` qui cible le relais d'extension sur le port par défaut.

Avant le premier attachement, ouvrez les Options de l'extension et configurez :

- `Port` (défaut `18792`)
- `Gateway token` (doit correspondre à `gateway.auth.token` / `OPENCLAW_GATEWAY_TOKEN`)

Utilisation :

- CLI : `openclaw browser --browser-profile chrome tabs`
- Outil agent : `browser` avec `profile="chrome"`

Si vous souhaitez un nom différent ou un port de relais différent, créez votre propre profil :

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### Ports Gateway personnalisés

Si vous utilisez un port gateway personnalisé, le port du relais d'extension est automatiquement dérivé :

**Port du relais d'extension = Port Gateway + 3**

Exemple : si `gateway.port: 19001`, alors :

- Port du relais d'extension : `19004` (gateway + 3)

Configurez l'extension pour utiliser le port de relais dérivé dans la page Options de l'extension.

## Attacher / détacher (bouton de barre d'outils)

- Ouvrez l'onglet que vous souhaitez qu'OpenClaw contrôle.
- Cliquez sur l'icône de l'extension.
  - Le badge affiche `ON` quand attaché.
- Cliquez à nouveau pour détacher.

## Quel onglet contrôle-t-il ?

- Il ne contrôle **pas** automatiquement « l'onglet que vous regardez ».
- Il contrôle **uniquement le(s) onglet(s) que vous avez explicitement attaché(s)** en cliquant sur le bouton de la barre d'outils.
- Pour changer : ouvrez l'autre onglet et cliquez sur l'icône de l'extension.

## Badge + erreurs courantes

- `ON` : attaché ; OpenClaw peut piloter cet onglet.
- `…` : connexion au relais local.
- `!` : relais non joignable/non authentifié (le plus fréquent : serveur relais non lance, ou token gateway manquant/incorrect).

Si vous voyez `!` :

- Assurez-vous que le Gateway fonctionne localement (configuration par défaut), ou lancez un node host sur cette machine si le Gateway fonctionne ailleurs.
- Ouvrez la page Options de l'extension ; elle valide l'accessibilité du relais + l'authentification du token gateway.

## Gateway distant (utiliser un node host)

### Gateway local (même machine que Chrome) : généralement **aucune étape supplémentaire**

Si le Gateway fonctionne sur la même machine que Chrome, il démarre le service de contrôle du navigateur sur loopback et lance automatiquement le serveur relais. L'extension communique avec le relais local ; les appels CLI/outil vont au Gateway.

### Gateway distant (Gateway sur une autre machine) : **lancer un node host**

Si votre Gateway fonctionne sur une autre machine, lancez un node host sur la machine qui exécute Chrome. Le Gateway transmettra les actions navigateur à ce node ; l'extension + le relais restent locaux à la machine du navigateur.

Si plusieurs nodes sont connectés, épinglez-en un avec `gateway.nodes.browser.node` ou définissez `gateway.nodes.browser.mode`.

## Bac à sable (conteneurs d'outils)

Si votre session d'agent est sandboxée (`agents.defaults.sandbox.mode != "off"`), l'outil `browser` peut être restreint :

- Par défaut, les sessions sandboxées ciblent souvent le **navigateur du bac à sable** (`target="sandbox"`), pas votre Chrome hôte.
- La prise de contrôle par le relais d'extension Chrome nécessite le contrôle du serveur de navigateur **hôte**.

Options :

- Le plus simple : utilisez l'extension depuis une session/agent **non sandboxée**.
- Ou autorisez le contrôle du navigateur hôte pour les sessions sandboxées :

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Puis assurez-vous que l'outil n'est pas refusé par la politique d'outils, et (si nécessaire) appelez `browser` avec `target="host"`.

Débogage : `openclaw sandbox explain`

## Conseils d'accès distant

- Gardez le Gateway et le node host sur le même tailnet ; évitez d'exposer les ports de relais au LAN ou à l'Internet public.
- Appariez les nodes intentionnellement ; désactivez le routage proxy navigateur si vous ne souhaitez pas le contrôle distant (`gateway.nodes.browser.mode="off"`).

## Fonctionnement de « extension path »

`openclaw browser extension path` affiche le répertoire **installé** sur disque contenant les fichiers de l'extension.

Le CLI ne fournit intentionnellement **pas** un chemin `node_modules`. Lancez toujours `openclaw browser extension install` d'abord pour copier l'extension vers un emplacement stable sous votre répertoire d'état OpenClaw.

Si vous déplacez ou supprimez ce répertoire d'installation, Chrome marquera l'extension comme défectueuse jusqu'à ce que vous la rechargiez depuis un chemin valide.

## Implications de sécurité (lisez ceci)

Ceci est puissant et risqué. Traitez-le comme donner au modèle « les mains sur votre navigateur ».

- L'extension utilise l'API debugger de Chrome (`chrome.debugger`). Quand attaché, le modèle peut :
  - cliquer/taper/naviguer dans cet onglet
  - lire le contenu de la page
  - accéder à tout ce que la session connectée de l'onglet peut accéder
- **Ce n'est pas isolé** comme le profil gère par openclaw dédié.
  - Si vous vous attachez à votre profil/onglet quotidien, vous accordez l'accès à l'état de ce compte.

Recommandations :

- Préférez un profil Chrome dédié (séparé de votre navigation personnelle) pour l'utilisation du relais d'extension.
- Gardez le Gateway et tous les node hosts uniquement sur le tailnet ; comptez sur l'authentification Gateway + l'appariement des nodes.
- Évitez d'exposer les ports de relais sur le LAN (`0.0.0.0`) et évitez Funnel (public).
- Le relais bloque les origines non-extension et exige l'authentification par token gateway pour `/cdp` et `/extension`.

Liens connexes :

- Présentation de l'outil navigateur : [Navigateur](/tools/browser)
- Audit de sécurité : [Sécurité](/gateway/security)
- Configuration Tailscale : [Tailscale](/gateway/tailscale)
