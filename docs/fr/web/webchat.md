---
summary: "Hôte statique WebChat en boucle locale et utilisation du WS du Gateway pour l'interface de chat"
read_when:
  - Débogage ou configuration de l'accès WebChat
title: "WebChat"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/web/webchat.md
  workflow: manual
---

# WebChat (Interface WebSocket du Gateway)

Statut : l'interface de chat SwiftUI macOS/iOS communique directement avec le WebSocket du Gateway.

## Présentation

- Une interface de chat native pour le gateway (pas de navigateur intégré et pas de serveur statique local).
- Utilisé les mêmes sessions et règles de routage que les autres canaux.
- Routage déterministe : les réponses retournent toujours vers le WebChat.

## Démarrage rapide

1. Démarrez le gateway.
2. Ouvrez l'interface WebChat (application macOS/iOS) ou l'onglet chat de l'Interface de contrôle.
3. Assurez-vous que l'authentification du gateway est configurée (requise par défaut, même en boucle locale).

## Fonctionnement (comportement)

- L'interface se connecte au WebSocket du Gateway et utilise `chat.history`, `chat.send` et `chat.inject`.
- `chat.history` est limité en taille pour la stabilité : le Gateway peut tronquer les champs texte longs, omettre les métadonnées lourdes et remplacer les entrées surdimensionnées par `[chat.history omitted: message too large]`.
- `chat.inject` ajouté une note d'assistant directement à la transcription et la diffuse à l'interface (pas d'exécution d'agent).
- Les exécutions annulées peuvent conserver la sortie partielle de l'assistant visible dans l'interface.
- Le Gateway conserve le texte partiel annulé de l'assistant dans l'historique de transcription lorsqu'une sortie tamponnée existe, et marque ces entrées avec des métadonnées d'annulation.
- L'historique est toujours récupéré depuis le gateway (pas de surveillance de fichier local).
- Si le gateway est injoignable, le WebChat est en lecture seule.

## Panneau d'outils des agents de l'Interface de contrôle

- Le panneau Outils `/agents` de l'Interface de contrôle récupère un catalogue d'exécution via `tools.catalog` et étiquette chaque
  outil comme `core` ou `plugin:<id>` (plus `optional` pour les outils de plugin optionnels).
- Si `tools.catalog` n'est pas disponible, le panneau se rabat sur une liste statique intégrée.
- Le panneau modifie la configuration de profil et de surcharge, mais l'accès effectif à l'exécution suit toujours la priorité
  des politiques (`allow`/`deny`, surcharges par agent et par fournisseur/canal).

## Utilisation distante

- Le mode distant fait passer le WebSocket par un tunnel du gateway via SSH/Tailscale.
- Vous n'avez pas besoin d'exécuter un serveur WebChat séparé.

## Référence de configuration (WebChat)

Configuration complète : [Configuration](/gateway/configuration)

Options de canal :

- Pas de bloc `webchat.*` dédié. Le WebChat utilisé le point de terminaison du gateway + les paramètres d'authentification ci-dessous.

Options globales associées :

- `gateway.port`, `gateway.bind` : hôte/port WebSocket.
- `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password` : authentification WebSocket (jeton/mot de passe).
- `gateway.auth.mode: "trusted-proxy"` : authentification par proxy de confiance pour les clients navigateur (voir [Authentification par proxy de confiance](/gateway/trusted-proxy-auth)).
- `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password` : cible du gateway distant.
- `session.*` : stockage de session et valeurs par défaut de la clé principale.
