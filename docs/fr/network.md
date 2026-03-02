---
summary: "Hub réseau : surfaces du Gateway, appairage, découverte et sécurité"
read_when:
  - Vous avez besoin de la vue d'ensemble de l'architecture réseau + sécurité
  - Vous déboguez l'accès local vs tailnet ou l'appairage
  - Vous voulez la liste canonique des docs réseau
title: "Réseau"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/network.md
  workflow: manual
---

# Hub réseau

Ce hub regroupe les docs principales sur la façon dont OpenClaw connecte, appaire et sécurise
les appareils en localhost, LAN et tailnet.

## Modèle central

- [Architecture du Gateway](/concepts/architecture)
- [Protocole du Gateway](/gateway/protocol)
- [Guide du Gateway](/gateway)
- [Surfaces web + modes de liaison](/web)

## Appairage + identité

- [Vue d'ensemble de l'appairage (DM + nœuds)](/channels/pairing)
- [Appairage de nœuds par le Gateway](/gateway/pairing)
- [CLI Appareils (appairage + rotation de tokens)](/cli/devices)
- [CLI Appairage (approbations DM)](/cli/pairing)

Confiance locale :

- Les connexions locales (loopback ou l'adresse tailnet propre de l'hôte du Gateway) peuvent être
  auto-approuvées pour l'appairage afin de garder une UX fluide sur le même hôte.
- Les clients tailnet/LAN non locaux nécessitent toujours une approbation d'appairage explicite.

## Découverte + transports

- [Découverte et transports](/gateway/discovery)
- [Bonjour / mDNS](/gateway/bonjour)
- [Accès distant (SSH)](/gateway/remote)
- [Tailscale](/gateway/tailscale)

## Nœuds + transports

- [Vue d'ensemble des nœuds](/nodes)
- [Protocole Bridge (nœuds legacy)](/gateway/bridge-protocol)
- [Guide nœud : iOS](/platforms/ios)
- [Guide nœud : Android](/platforms/android)

## Sécurité

- [Vue d'ensemble de la sécurité](/gateway/security)
- [Référence de configuration du Gateway](/gateway/configuration)
- [Dépannage](/gateway/troubleshooting)
- [Doctor](/gateway/doctor)
