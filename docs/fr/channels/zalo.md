---
summary: "Statut de prise en charge du bot Zalo, fonctionnalités et configuration"
read_when:
  - Vous travaillez sur les fonctionnalités Zalo ou les webhooks
title: "Zalo"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/channels/zalo.md
  workflow: manual
---

# Zalo (Bot API)

Statut : expérimental. Les Messages privés sont pris en charge ; la gestion des groupes est disponible avec des contrôles de politique de groupe explicites.

## Plugin requis

Zalo est livré comme un plugin et n'est pas inclus dans l'installation de base.

- Installation via CLI : `openclaw plugins install @openclaw/zalo`
- Ou sélectionnez **Zalo** pendant l'onboarding et confirmez l'invite d'installation
- Détails : [Plugins](/tools/plugin)

## Configuration rapide (débutant)

1. Installez le plugin Zalo :
   - Depuis un checkout source : `openclaw plugins install ./extensions/zalo`
   - Depuis npm (si publié) : `openclaw plugins install @openclaw/zalo`
   - Ou choisissez **Zalo** dans l'onboarding et confirmez l'invite d'installation
2. Définissez le token :
   - Env : `ZALO_BOT_TOKEN=...`
   - Ou config : `channels.zalo.botToken: "..."`.
3. Redémarrez le Gateway (ou terminez l'onboarding).
4. L'accès aux Messages privés est en mode appairage par défaut ; approuvez le code d'appairage au premier contact.

Configuration minimale :

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## Description

Zalo est une application de messagerie centrée sur le Vietnam ; son Bot API permet au Gateway d'exécuter un bot pour les conversations 1:1.
C'est une bonne option pour le support ou les notifications où vous souhaitez un routage déterministe vers Zalo.

- Un canal Zalo Bot API détenu par le Gateway.
- Routage déterministe : les réponses reviennent vers Zalo ; le modèle ne choisit jamais les canaux.
- Les Messages privés partagent la session principale de l'agent.
- Les groupes sont pris en charge avec des contrôles de politique (`groupPolicy` + `groupAllowFrom`) et utilisent par défaut un comportement de liste autorisée en mode sécurisé.

## Configuration (chemin rapide)

### 1) Créer un token de bot (Zalo Bot Platform)

1. Allez sur [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) et connectez-vous.
2. Créez un nouveau bot et configurez ses paramètres.
3. Copiez le token du bot (format : `12345689:abc-xyz`).

### 2) Configurer le token (env ou config)

Exemple :

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

Option env : `ZALO_BOT_TOKEN=...` (fonctionne pour le compte par défaut uniquement).

Support multi-compte : utilisez `channels.zalo.accounts` avec des tokens par compte et un `name` optionnel.

3. Redémarrez le Gateway. Zalo démarre lorsqu'un token est résolu (env ou config).
4. L'accès aux Messages privés est en mode appairage par défaut. Approuvez le code lorsque le bot est contacté pour la première fois.

## Fonctionnement (comportement)

- Les messages entrants sont normalisés dans l'enveloppe de canal partagée avec des espaces réservés pour les médias.
- Les réponses sont toujours routées vers le même chat Zalo.
- Long-polling par défaut ; mode webhook disponible avec `channels.zalo.webhookUrl`.

## Limités

- Le texte sortant est découpé à 2000 caractères (limité de l'API Zalo).
- Les téléchargements/envois de médias sont limités par `channels.zalo.mediaMaxMb` (par défaut 5).
- Le streaming est bloqué par défaut car la limité de 2000 caractères rend le streaming moins utile.

## Contrôle d'accès (Messages privés)

### Accès aux Messages privés

- Par défaut : `channels.zalo.dmPolicy = "pairing"`. Les expéditeurs inconnus reçoivent un code d'appairage ; les messages sont ignorés jusqu'à approbation (les codes expirent après 1 heure).
- Approbation via :
  - `openclaw pairing list zalo`
  - `openclaw pairing approve zalo <CODE>`
- L'appairage est l'échange de token par défaut. Détails : [Appairage](/channels/pairing)
- `channels.zalo.allowFrom` accepte les identifiants utilisateur numériques (pas de recherche de nom d'utilisateur disponible).

## Contrôle d'accès (Groupes)

- `channels.zalo.groupPolicy` contrôle la gestion des messages entrants de groupe : `open | allowlist | disabled`.
- Le comportement par défaut est sécurisé : `allowlist`.
- `channels.zalo.groupAllowFrom` restreint quels identifiants d'expéditeurs peuvent déclencher le bot dans les groupes.
- Si `groupAllowFrom` n'est pas défini, Zalo utilisé `allowFrom` en repli pour les vérifications d'expéditeur.
- `groupPolicy: "disabled"` bloque tous les messages de groupe.
- `groupPolicy: "open"` autorisé tout membre du groupe (filtré par mention).
- Note d'exécution : si `channels.zalo` est complètement absent, l'exécution revient à `groupPolicy="allowlist"` par sécurité.

## Long-polling vs webhook

- Par défaut : long-polling (pas d'URL publique requise).
- Mode webhook : définissez `channels.zalo.webhookUrl` et `channels.zalo.webhookSecret`.
  - Le secret du webhook doit faire 8-256 caractères.
  - L'URL du webhook doit utiliser HTTPS.
  - Zalo envoie les événements avec l'en-tête `X-Bot-Api-Secret-Token` pour la vérification.
  - Le HTTP du Gateway gère les requêtes webhook sur `channels.zalo.webhookPath` (par défaut le chemin de l'URL webhook).
  - Les requêtes doivent utiliser `Content-Type: application/json` (ou les types de médias `+json`).
  - Les événements en double (`event_name + message_id`) sont ignorés pendant une courte fenêtre de rejeu.
  - Le trafic en rafale est limité en débit par chemin/source et peut retourner HTTP 429.

**Note :** getUpdates (polling) et webhook sont mutuellement exclusifs selon la documentation de l'API Zalo.

## Types de messages pris en charge

- **Messages texte** : support complet avec découpage à 2000 caractères.
- **Messages image** : téléchargement et traitement des images entrantes ; envoi d'images via `sendPhoto`.
- **Stickers** : journalisés mais non entièrement traités (pas de réponse de l'agent).
- **Types non pris en charge** : journalisés (par exemple, messages d'utilisateurs protégés).

## Fonctionnalités

| Fonctionnalité   | Statut                                                              |
| ---------------- | ------------------------------------------------------------------- |
| Messages privés  | Pris en charge                                                      |
| Groupes          | Pris en charge avec contrôles de politique (liste autorisée par défaut) |
| Médias (images)  | Pris en charge                                                      |
| Réactions        | Non pris en charge                                                  |
| Fils             | Non pris en charge                                                  |
| Sondages         | Non pris en charge                                                  |
| Commandes natives | Non pris en charge                                                 |
| Streaming        | Bloqué (limité de 2000 caractères)                                  |

## Cibles de livraison (CLI/cron)

- Utilisez un identifiant de chat comme cible.
- Exemple : `openclaw message send --channel zalo --target 123456789 --message "hi"`.

## Dépannage

**Le bot ne répond pas :**

- Vérifiez que le token est valide : `openclaw channels status --probe`
- Vérifiez que l'expéditeur est approuvé (appairage ou allowFrom)
- Consultez les logs du Gateway : `openclaw logs --follow`

**Le webhook ne reçoit pas d'événements :**

- Assurez-vous que l'URL du webhook utilisé HTTPS
- Vérifiez que le token secret fait 8-256 caractères
- Confirmez que le point de terminaison HTTP du Gateway est accessible sur le chemin configuré
- Vérifiez que le polling getUpdates n'est pas en cours d'exécution (ils sont mutuellement exclusifs)

## Référence de configuration (Zalo)

Configuration complète : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.zalo.enabled` : activer/désactiver le démarrage du canal.
- `channels.zalo.botToken` : token du bot depuis Zalo Bot Platform.
- `channels.zalo.tokenFile` : lire le token depuis un chemin de fichier.
- `channels.zalo.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : pairing).
- `channels.zalo.allowFrom` : liste autorisée de Messages privés (identifiants utilisateur). `open` nécessite `"*"`. L'assistant demandera des identifiants numériques.
- `channels.zalo.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
- `channels.zalo.groupAllowFrom` : liste autorisée d'expéditeurs de groupe (identifiants utilisateur). Utilisé `allowFrom` en repli quand non défini.
- `channels.zalo.mediaMaxMb` : limité de média entrant/sortant (Mo, par défaut 5).
- `channels.zalo.webhookUrl` : activer le mode webhook (HTTPS requis).
- `channels.zalo.webhookSecret` : secret du webhook (8-256 caractères).
- `channels.zalo.webhookPath` : chemin du webhook sur le serveur HTTP du Gateway.
- `channels.zalo.proxy` : URL du proxy pour les requêtes API.

Options multi-compte :

- `channels.zalo.accounts.<id>.botToken` : token par compte.
- `channels.zalo.accounts.<id>.tokenFile` : fichier de token par compte.
- `channels.zalo.accounts.<id>.name` : nom d'affichage.
- `channels.zalo.accounts.<id>.enabled` : activer/désactiver le compte.
- `channels.zalo.accounts.<id>.dmPolicy` : politique de Message privé par compte.
- `channels.zalo.accounts.<id>.allowFrom` : liste autorisée par compte.
- `channels.zalo.accounts.<id>.groupPolicy` : politique de groupe par compte.
- `channels.zalo.accounts.<id>.groupAllowFrom` : liste autorisée d'expéditeurs de groupe par compte.
- `channels.zalo.accounts.<id>.webhookUrl` : URL de webhook par compte.
- `channels.zalo.accounts.<id>.webhookSecret` : secret de webhook par compte.
- `channels.zalo.accounts.<id>.webhookPath` : chemin de webhook par compte.
- `channels.zalo.accounts.<id>.proxy` : URL de proxy par compte.
