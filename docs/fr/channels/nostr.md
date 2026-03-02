---
summary: "Canal DM Nostr via messages chiffrés NIP-04"
read_when:
  - Vous voulez qu'OpenClaw recoive des DM via Nostr
  - Vous configurez une messagerie decentralisee
title: "Nostr"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/nostr.md"
  workflow: "manual"
---

# Nostr

**Statut :** Plugin optionnel (désactivé par défaut).

Nostr est un protocole decentralise pour les réseaux sociaux. Ce canal permet a OpenClaw de recevoir et répondre aux messages directs chiffrés (DM) via NIP-04.

## Installation (a la demande)

### Configuration initiale (recommandé)

- L'assistant de configuration initiale (`openclaw onboard`) et `openclaw channels add` listent les plugins de canaux optionnels.
- Sélectionner Nostr vous invite a installer le plugin a la demande.

Valeurs par défaut d'installation :

- **Canal dev + checkout git disponible :** utilisé le chemin local du plugin.
- **Stable/Beta :** téléchargé depuis npm.

Vous pouvez toujours remplacer le choix dans l'invite.

### Installation manuelle

```bash
openclaw plugins install @openclaw/nostr
```

Utiliser un checkout local (workflows de développement) :

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Redemarrez le Gateway après avoir installe ou activé des plugins.

## Configuration rapide

1. Generez une paire de clés Nostr (si nécessaire) :

```bash
# En utilisant nak
nak key generate
```

2. Ajoutez a la configuration :

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3. Exportez la clé :

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4. Redemarrez le Gateway.

## Référence de configuration

| Clé          | Type     | Défaut                                      | Description                                |
| ------------ | -------- | ------------------------------------------- | ------------------------------------------ |
| `privateKey` | string   | requis                                      | Clé privée au format `nsec` ou hexadecimal |
| `relays`     | string[] | `['wss://relay.damus.io', 'wss://nos.lol']` | URLs de relais (WebSocket)                 |
| `dmPolicy`   | string   | `pairing`                                   | Politique d'accès DM                       |
| `allowFrom`  | string[] | `[]`                                        | Clés publiques des expéditeurs autorisés   |
| `enabled`    | boolean  | `true`                                      | Activer/désactiver le canal                |
| `name`       | string   | -                                           | Nom d'affichage                            |
| `profile`    | object   | -                                           | Métadonnées de profil NIP-01               |

## Métadonnées de profil

Les donnees de profil sont publiees comme un événement NIP-01 `kind:0`. Vous pouvez les gérer depuis l'interface de contrôle (Channels -> Nostr -> Profile) ou les définir directement dans la configuration.

Exemple :

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Personal assistant DM bot",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

Notes :

- Les URLs de profil doivent utiliser `https://`.
- L'importation depuis les relais fusionne les champs et preserve les surcharges locales.

## Contrôle d'accès

### Politiques DM

- **pairing** (défaut) : les expéditeurs inconnus reçoivent un code d'appairage.
- **allowlist** : seules les clés publiques dans `allowFrom` peuvent envoyer des DM.
- **open** : DM entrants publics (nécessite `allowFrom: ["*"]`).
- **disabled** : ignorer les DM entrants.

### Exemple de liste d'autorisation

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## Formats de clés

Formats acceptés :

- **Clé privée :** `nsec...` ou hexadecimal 64 caractères
- **Clés publiques (`allowFrom`) :** `npub...` ou hexadecimal

## Relais

Defauts : `relay.damus.io` et `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

Conseils :

- Utilisez 2-3 relais pour la redondance.
- Evitez trop de relais (latence, duplication).
- Les relais payants peuvent ameliorer la fiabilité.
- Les relais locaux conviennent pour les tests (`ws://localhost:7777`).

## Support du protocole

| NIP    | Statut      | Description                               |
| ------ | ----------- | ----------------------------------------- |
| NIP-01 | Pris en charge | Format d'événement de base + métadonnées de profil |
| NIP-04 | Pris en charge | DM chiffrés (`kind:4`)                   |
| NIP-17 | Prevu       | DM enveloppes (gift-wrapped)              |
| NIP-44 | Prevu       | Chiffrement versionne                     |

## Tests

### Relais local

```bash
# Demarrer strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### Test manuel

1. Notez la clé publique du bot (npub) depuis les logs.
2. Ouvrez un client Nostr (Damus, Amethyst, etc.).
3. Envoyez un DM a la clé publique du bot.
4. Verifiez la réponse.

## Dépannage

### Messages non reçus

- Verifiez que la clé privée est valide.
- Assurez-vous que les URLs de relais sont joignables et utilisent `wss://` (ou `ws://` pour local).
- Confirmez que `enabled` n'est pas `false`.
- Verifiez les logs du Gateway pour les erreurs de connexion aux relais.

### Réponses non envoyées

- Verifiez que le relais accepte les écritures.
- Verifiez la connectivité sortante.
- Surveillez les limités de debit des relais.

### Réponses en double

- Attendu lors de l'utilisation de plusieurs relais.
- Les messages sont dedupliques par ID d'événement ; seule la première livraison déclenche une réponse.

## Sécurité

- Ne commitez jamais les clés privées.
- Utilisez les variables d'environnement pour les clés.
- Envisagez `allowlist` pour les bots en production.

## Limitations (MVP)

- Messages directs uniquement (pas de discussions de groupe).
- Pas de pieces jointes média.
- NIP-04 uniquement (NIP-17 gift-wrap prevu).
