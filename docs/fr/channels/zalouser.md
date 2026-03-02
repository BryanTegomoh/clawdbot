---
summary: "Support du compte personnel Zalo via zca-cli (connexion QR), capacités et configuration"
read_when:
  - Configuration de Zalo Personnel pour OpenClaw
  - Debogage de la connexion ou du flux de messages Zalo Personnel
title: "Zalo Personnel"
x-i18n:
  generated_at: "2026-02-25"
  model: "claude-opus-4-6"
  provider: "anthropic"
  source_path: "docs/channels/zalouser.md"
  workflow: "manual"
---

# Zalo Personnel (non officiel)

Statut : experimental. Cette intégration automatise un **compte Zalo personnel** via `zca-cli`.

> **Avertissement :** Il s'agit d'une intégration non officielle qui peut entrainer la suspension/le bannissement du compte. Utilisez a vos propres risques.

## Plugin requis

Zalo Personnel est distribue en tant que plugin et n'est pas inclus dans l'installation de base.

- Installation via CLI : `openclaw plugins install @openclaw/zalouser`
- Ou depuis un checkout source : `openclaw plugins install ./extensions/zalouser`
- Details : [Plugins](/tools/plugin)

## Prerequis : zca-cli

La machine Gateway doit avoir le binaire `zca` disponible dans le `PATH`.

- Verifiez : `zca --version`
- Si absent, installez zca-cli (voir `extensions/zalouser/README.md` ou la documentation upstream de zca-cli).

## Configuration rapide (debutant)

1. Installez le plugin (voir ci-dessus).
2. Connexion (QR, sur la machine Gateway) :
   - `openclaw channels login --channel zalouser`
   - Scannez le code QR dans le terminal avec l'application mobile Zalo.
3. Activez le canal :

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4. Redemarrez le Gateway (ou terminez la configuration initiale).
5. L'accès DM est en mode appairage par défaut ; approuvez le code d'appairage au premier contact.

## Ce que c'est

- Utilisé `zca listen` pour recevoir les messages entrants.
- Utilisé `zca msg ...` pour envoyer les réponses (texte/média/lien).
- Concu pour les cas d'utilisation de "compte personnel" ou l'API Zalo Bot n'est pas disponible.

## Nommage

L'identifiant du canal est `zalouser` pour rendre explicite le fait qu'il automatise un **compte utilisateur personnel Zalo** (non officiel). Nous gardons `zalo` réservé pour une potentielle future intégration officielle de l'API Zalo.

## Trouver les IDs (répertoire)

Utilisez le CLI du répertoire pour decouvrir les pairs/groupes et leurs IDs :

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## Limités

- Le texte sortant est decoupe a environ 2000 caractères (limités du client Zalo).
- Le streaming est bloque par défaut.

## Contrôle d'accès (DM)

`channels.zalouser.dmPolicy` supporte : `pairing | allowlist | open | disabled` (défaut : `pairing`).
`channels.zalouser.allowFrom` accepte les IDs utilisateur ou les noms. L'assistant resout les noms en IDs via `zca friend find` quand disponible.

Approbation via :

- `openclaw pairing list zalouser`
- `openclaw pairing approve zalouser <code>`

## Accès aux groupes (optionnel)

- Par défaut : `channels.zalouser.groupPolicy = "open"` (groupes autorisés). Utilisez `channels.defaults.groupPolicy` pour remplacer la valeur par défaut quand non défini.
- Restreindre a une liste d'autorisation avec :
  - `channels.zalouser.groupPolicy = "allowlist"`
  - `channels.zalouser.groups` (les clés sont des IDs de groupe ou des noms)
- Bloquer tous les groupes : `channels.zalouser.groupPolicy = "disabled"`.
- L'assistant de configuration peut demander les listes d'autorisation de groupes.
- Au démarrage, OpenClaw resout les noms de groupes/utilisateurs dans les listes d'autorisation en IDs et journalise la correspondance ; les entrées non resolues sont conservees telles quelles.

Exemple :

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

## Multi-comptes

Les comptes sont associés aux profils zca. Exemple :

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## Dépannage

**`zca` introuvable :**

- Installez zca-cli et assurez-vous qu'il est dans le `PATH` pour le processus Gateway.

**La connexion ne persiste pas :**

- `openclaw channels status --probe`
- Reconnexion : `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`
