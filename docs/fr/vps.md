---
summary: "Hub d'hébergement VPS pour OpenClaw (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Vous souhaitez exécuter le Gateway dans le cloud
  - Vous avez besoin d'un aperçu rapide des guides VPS/hébergement
title: "Hébergement VPS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/vps.md
  workflow: manual
---

# Hébergement VPS

Ce hub renvoie vers les guides VPS/hébergement pris en charge et explique le fonctionnement
des déploiements cloud à haut niveau.

## Choisir un fournisseur

- **Railway** (un clic + configuration navigateur) : [Railway](/install/railway)
- **Northflank** (un clic + configuration navigateur) : [Northflank](/install/northflank)
- **Oracle Cloud (Always Free)** : [Oracle](/platforms/oracle) — 0$/mois (Always Free, ARM ; la capacité/inscription peut être capricieuse)
- **Fly.io** : [Fly.io](/install/fly)
- **Hetzner (Docker)** : [Hetzner](/install/hetzner)
- **GCP (Compute Engine)** : [GCP](/install/gcp)
- **exe.dev** (VM + proxy HTTPS) : [exe.dev](/install/exe-dev)
- **AWS (EC2/Lightsail/free tier)** : fonctionne bien aussi. Guide vidéo :
  [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## Fonctionnement des configurations cloud

- Le **Gateway s'exécute sur le VPS** et possède l'état + l'espace de travail.
- Vous vous connectez depuis votre laptop/téléphone via l'**UI de contrôle** ou **Tailscale/SSH**.
- Traitez le VPS comme la source de vérité et **sauvegardez** l'état + l'espace de travail.
- Sécurité par défaut : gardez le Gateway sur loopback et accédez-y via tunnel SSH ou Tailscale Serve.
  Si vous liez sur `lan`/`tailnet`, exigez `gateway.auth.token` ou `gateway.auth.password`.

Accès distant : [Gateway distant](/gateway/remote)
Hub plateformes : [Plateformes](/platforms)

## Agent d'entreprise partagé sur un VPS

C'est une configuration valide quand les utilisateurs sont dans une même frontière de confiance (par exemple une équipe d'entreprise), et que l'agent est strictement professionnel.

- Gardez-le sur un runtime dédié (VPS/VM/conteneur + utilisateur/comptes OS dédiés).
- Ne connectez pas ce runtime à des comptes Apple/Google personnels ni à des profils de navigateur/gestionnaire de mots de passe personnels.
- Si les utilisateurs sont adversaires entre eux, séparez par gateway/hôte/utilisateur OS.

Détails du modèle de sécurité : [Sécurité](/gateway/security)

## Utiliser des nœuds avec un VPS

Vous pouvez garder le Gateway dans le cloud et appairer des **nœuds** sur vos appareils locaux
(Mac/iOS/Android/headless). Les nœuds fournissent l'écran/caméra/canvas local et les capacités `system.run`
tandis que le Gateway reste dans le cloud.

Docs : [Nœuds](/nodes), [CLI Nœuds](/cli/nodes)
