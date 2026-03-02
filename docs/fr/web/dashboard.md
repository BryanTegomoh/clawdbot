---
summary: "Tableau de bord du Gateway (Interface de contrôle) : accès et authentification"
read_when:
  - Modification de l'authentification ou des modes d'exposition du Tableau de bord
title: "Tableau de bord"
sidebarTitle: "Tableau de bord"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/web/dashboard.md
  workflow: manual
---

# Tableau de bord (Interface de contrôle)

Le Tableau de bord du Gateway est l'Interface de contrôle dans le navigateur servie à `/` par défaut
(surcharger avec `gateway.controlUi.basePath`).

Ouverture rapide (Gateway local) :

- [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (ou [http://localhost:18789/](http://localhost:18789/))

Références clés :

- [Interface de contrôle](/web/control-ui) pour l'utilisation et les fonctionnalités de l'interface.
- [Tailscale](/gateway/tailscale) pour l'automatisation Serve/Funnel.
- [Surfaces web](/web) pour les modes de liaison et les notes de sécurité.

L'authentification est appliquée lors de la poignée de main WebSocket via `connect.params.auth`
(jeton ou mot de passe). Voir `gateway.auth` dans [Configuration du Gateway](/gateway/configuration).

Note de sécurité : l'Interface de contrôle est une **surface d'administration** (chat, configuration, approbations exec).
Ne l'exposez pas publiquement. L'interface stocke le jeton dans `localStorage` après le premier chargement.
Préférez localhost, Tailscale Serve ou un tunnel SSH.

## Chemin rapide (recommandé)

- Après la configuration initiale, le CLI ouvre automatiquement le Tableau de bord et affiche un lien propre (sans jeton).
- Rouvrir à tout moment : `openclaw dashboard` (copie le lien, ouvre le navigateur si possible, affiche une indication SSH si en mode sans tête).
- Si l'interface demande l'authentification, collez le jeton depuis `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) dans les paramètres de l'Interface de contrôle.

## Bases du jeton (local vs distant)

- **Localhost** : ouvrez `http://127.0.0.1:18789/`.
- **Source du jeton** : `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) ; l'interface stocke une copie dans localStorage après votre connexion.
- **Hors localhost** : utilisez Tailscale Serve (sans jeton pour l'Interface de contrôle/WebSocket si `gateway.auth.allowTailscale: true`, suppose un hôte de gateway de confiance ; les API HTTP nécessitent toujours un jeton/mot de passe), liaison tailnet avec un jeton, ou un tunnel SSH. Voir [Surfaces web](/web).

## Si vous voyez "unauthorized" / 1008

- Assurez-vous que le gateway est joignable (local : `openclaw status` ; distant : tunnel SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/`).
- Récupérez le jeton depuis l'hôte du gateway : `openclaw config get gateway.auth.token` (ou générez-en un : `openclaw doctor --generate-gateway-token`).
- Dans les paramètres du Tableau de bord, collez le jeton dans le champ d'authentification, puis connectez-vous.
