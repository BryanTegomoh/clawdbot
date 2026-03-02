---
summary: "Comment le Gateway, les nœuds et l'hôte canvas se connectent."
read_when:
  - Vous souhaitez une vue concise du modèle réseau du Gateway
title: "Modèle réseau"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/network-model.md
  workflow: manual
---

La plupart des opérations passent par le Gateway (`openclaw gateway`), un processus unique longue durée qui possède les connexions de canaux et le plan de contrôle WebSocket.

## Règles fondamentales

- Un Gateway par hôte est recommandé. C'est le seul processus autorisé à posséder la session WhatsApp Web. Pour les bots de secours ou l'isolation stricte, exécutez plusieurs gateways avec des profils et ports isolés. Voir [Gateways multiples](/gateway/multiple-gateways).
- Loopback d'abord : le Gateway WS est par défaut sur `ws://127.0.0.1:18789`. L'assistant génère un jeton gateway par défaut, même pour le loopback. Pour l'accès tailnet, exécutez `openclaw gateway --bind tailnet --token ...` car les jetons sont requis pour les binds non-loopback.
- Les nœuds se connectent au Gateway WS via LAN, tailnet ou SSH selon les besoins. Le bridge TCP legacy est obsolète.
- L'hôte canvas est servi par le serveur HTTP du Gateway sur le **même port** que le Gateway (par défaut `18789`) :
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    Lorsque `gateway.auth` est configuré et que le Gateway se lie au-delà du loopback, ces routes sont protégées par l'authentification Gateway. Les clients nœuds utilisent des URLs de capacité liées au nœud et attachées à leur session WS activé. Voir [Configuration Gateway](/gateway/configuration) (`canvasHost`, `gateway`).
- L'utilisation distante se fait typiquement par tunnel SSH ou VPN tailnet. Voir [Accès distant](/gateway/remote) et [Découverte](/gateway/discovery).
