---
summary: "Prise en charge du canal WhatsApp, contrôles d'accès, comportement de livraison et opérations"
read_when:
  - Travail sur le comportement du canal WhatsApp/web ou le routage de la boite de reception
title: "WhatsApp"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/whatsapp.md
  workflow: manual
---

# WhatsApp (canal Web)

Statut : prêt pour la production via WhatsApp Web (Baileys). Le Gateway possède la ou les sessions liées.

<CardGroup cols={3}>
  <Card title="Appairage" icon="link" href="/channels/pairing">
    La politique de messages privés par défaut est l'appairage pour les expéditeurs inconnus.
  </Card>
  <Card title="Dépannage des canaux" icon="wrench" href="/channels/troubleshooting">
    Diagnostics et guides de réparation inter-canaux.
  </Card>
  <Card title="Configuration du Gateway" icon="settings" href="/gateway/configuration">
    Modèles et exemples complets de configuration de canal.
  </Card>
</CardGroup>

## Configuration rapide

<Steps>
  <Step title="Configurer la politique d'accès WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

  </Step>

  <Step title="Lier WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    Pour un compte spécifique :

```bash
openclaw channels login --channel whatsapp --account work
```

  </Step>

  <Step title="Démarrer le gateway">

```bash
openclaw gateway
```

  </Step>

  <Step title="Approuver la première demande d'appairage (si vous utilisez le mode appairage)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    Les demandes d'appairage expirent après 1 heure. Les demandes en attente sont limitées a 3 par canal.

  </Step>
</Steps>

<Note>
OpenClaw recommandé d'utiliser WhatsApp sur un numéro séparé lorsque c'est possible. (Les métadonnées du canal et le flux d'intégration sont optimisés pour cette configuration, mais les configurations avec un numéro personnel sont également prises en charge.)
</Note>

## Modèles de déploiement

<AccordionGroup>
  <Accordion title="Numéro dédié (recommandé)">
    C'est le mode opérationnel le plus propre :

    - identité WhatsApp séparée pour OpenClaw
    - listes autorisées de messages privés et limités de routage plus claires
    - moins de risques de confusion avec le self-chat

    Modèle de politique minimal :

    ```json5
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Repli sur numéro personnel">
    L'intégration prend en charge le mode numéro personnel et écrit une base compatible avec le self-chat :

    - `dmPolicy: "allowlist"`
    - `allowFrom` inclut votre numéro personnel
    - `selfChatMode: true`

    A l'exécution, les protections de self-chat s'appuient sur le numéro self lié et `allowFrom`.

  </Accordion>

  <Accordion title="Périmètre du canal WhatsApp Web uniquement">
    Le canal de la plateforme de messagerie est base sur WhatsApp Web (`Baileys`) dans l'architecture actuelle des canaux OpenClaw.

    Il n'y a pas de canal de messagerie WhatsApp Twilio séparé dans le registre des canaux de discussion intégré.

  </Accordion>
</AccordionGroup>

## Modèle d'exécution

- Le Gateway possède le socket WhatsApp et la boucle de reconnexion.
- Les envois sortants nécessitent un écouteur WhatsApp actif pour le compte cible.
- Les discussions de statut et de diffusion sont ignorées (`@status`, `@broadcast`).
- Les discussions directes utilisent les règles de session de messages privés (`session.dmScope` ; par défaut `main` regroupe les messages privés vers la session principale de l'agent).
- Les sessions de groupe sont isolees (`agent:<agentId>:whatsapp:group:<jid>`).

## Contrôle d'accès et activation

<Tabs>
  <Tab title="Politique de messages privés">
    `channels.whatsapp.dmPolicy` contrôle l'accès aux discussions directes :

    - `pairing` (par défaut)
    - `allowlist`
    - `open` (nécessite que `allowFrom` inclue `"*"`)
    - `disabled`

    `allowFrom` accepte les numéros au format E.164 (normalisés en interne).

    Remplacement multi-comptes : `channels.whatsapp.accounts.<id>.dmPolicy` (et `allowFrom`) prennent le pas sur les valeurs par défaut au niveau du canal pour ce compte.

    Details du comportement a l'exécution :

    - les appairages sont persistés dans le magasin d'autorisation du canal et fusionnés avec le `allowFrom` configuré
    - si aucune liste autorisée n'est configurée, le numéro self lié est autorisé par défaut
    - les messages privés sortants `fromMe` ne sont jamais auto-appairés

  </Tab>

  <Tab title="Politique de groupe + listes autorisées">
    L'accès aux groupes a deux couches :

    1. **Liste autorisée d'appartenance au groupe** (`channels.whatsapp.groups`)
       - si `groups` est omis, tous les groupes sont éligibles
       - si `groups` est présent, il agit comme une liste autorisée de groupes (`"*"` autorisé)

    2. **Politique d'expéditeur de groupe** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open` : liste autorisée des expéditeurs contournee
       - `allowlist` : l'expéditeur doit correspondre a `groupAllowFrom` (ou `*`)
       - `disabled` : bloquer tous les messages entrants de groupe

    Repli de la liste autorisée des expéditeurs :

    - si `groupAllowFrom` n'est pas défini, l'exécution se rabat sur `allowFrom` lorsqu'il est disponible
    - les listes autorisées des expéditeurs sont evaluees avant l'activation par mention/réponse

    Note : si aucun bloc `channels.whatsapp` n'existe du tout, le repli de la politique de groupe a l'exécution est `allowlist` (avec un avertissement dans les logs), même si `channels.defaults.groupPolicy` est défini.

  </Tab>

  <Tab title="Mentions + /activation">
    Les réponses de groupe nécessitent une mention par défaut.

    La détection de mention inclut :

    - les mentions WhatsApp explicites de l'identité du bot
    - les modèles regex de mention configurés (`agents.list[].groupChat.mentionPatterns`, repli `messages.groupChat.mentionPatterns`)
    - la détection implicite de réponse au bot (l'expéditeur de la réponse correspond a l'identité du bot)

    Note de sécurité :

    - la citation/réponse satisfait uniquement le filtrage par mention ; elle n'accorde **pas** l'autorisation de l'expéditeur
    - avec `groupPolicy: "allowlist"`, les expéditeurs non autorisés sont toujours bloqués même s'ils répondent au message d'un utilisateur autorisé

    Commande d'activation au niveau de la session :

    - `/activation mention`
    - `/activation always`

    `activation` met a jour l'état de la session (pas la configuration globale). Elle est réservée au propriétaire.

  </Tab>
</Tabs>

## Comportement du numéro personnel et du self-chat

Lorsque le numéro self lié est également présent dans `allowFrom`, les protections de self-chat WhatsApp s'activent :

- ignorer les accuses de reception pour les tours de self-chat
- ignorer le comportement de déclenchement automatique par mention-JID qui vous pinguerait autrement
- si `messages.responsePrefix` n'est pas défini, les réponses en self-chat utilisent par défaut `[{identity.name}]` ou `[openclaw]`

## Normalisation des messages et contexte

<AccordionGroup>
  <Accordion title="Enveloppe entrante + contexte de réponse">
    Les messages WhatsApp entrants sont enveloppes dans l'enveloppe entrante partagee.

    Si une réponse citee existe, le contexte est ajouté sous cette forme :

    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```

    Les champs de métadonnées de réponse sont également remplis lorsqu'ils sont disponibles (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, JID/E.164 de l'expéditeur).

  </Accordion>

  <Accordion title="Espaces réservés pour les médias et extraction de localisation/contact">
    Les messages entrants contenant uniquement des médias sont normalisés avec des espaces réservés tels que :

    - `<média:image>`
    - `<média:video>`
    - `<média:audio>`
    - `<média:document>`
    - `<média:sticker>`

    Les payloads de localisation et de contact sont normalisés en contexte textuel avant le routage.

  </Accordion>

  <Accordion title="Injection de l'historique de groupe en attente">
    Pour les groupes, les messages non traites peuvent être mis en mémoire tampon et injectés comme contexte lorsque le bot est finalement déclenché.

    - limité par défaut : `50`
    - configuration : `channels.whatsapp.historyLimit`
    - repli : `messages.groupChat.historyLimit`
    - `0` désactive

    Marqueurs d'injection :

    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`

  </Accordion>

  <Accordion title="Accuses de reception">
    Les accuses de reception sont activés par défaut pour les messages WhatsApp entrants acceptés.

    Désactiver globalement :

    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```

    Remplacement par compte :

    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```

    Les tours de self-chat ignorent les accuses de reception même lorsqu'ils sont activés globalement.

  </Accordion>
</AccordionGroup>

## Livraison, découpage et médias

<AccordionGroup>
  <Accordion title="Découpage du texte">
    - limité de découpage par défaut : `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - le mode `newline` préféré les limités de paragraphe (lignes vides), puis se rabat sur le découpage par longueur
  </Accordion>

  <Accordion title="Comportement des médias sortants">
    - prend en charge les payloads d'image, video, audio (note vocale PTT) et document
    - `audio/ogg` est réécrit en `audio/ogg; codecs=opus` pour la compatibilité des notes vocales
    - la lecture de GIF animés est prise en charge via `gifPlayback: true` sur les envois video
    - les légendes sont appliquées au premier élément média lors de l'envoi de payloads de réponse multi-médias
    - la source média peut être HTTP(S), `file://` ou des chemins locaux
  </Accordion>

  <Accordion title="Limités de taille des médias et comportement de repli">
    - limité de sauvegarde des médias entrants : `channels.whatsapp.mediaMaxMb` (par défaut `50`)
    - limité des médias sortants pour les réponses automatiques : `agents.defaults.mediaMaxMb` (par défaut `5MB`)
    - les images sont auto-optimisees (redimensionnement/balayage de qualité) pour respecter les limités
    - en cas d'échec d'envoi de média, le repli du premier élément envoie un avertissement textuel au lieu de supprimer la réponse silencieusement
  </Accordion>
</AccordionGroup>

## Réactions d'accusé de reception

WhatsApp prend en charge les reactions d'accusé de reception immédiat sur les messages entrants via `channels.whatsapp.ackReaction`.

```json5
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Notes de comportement :

- envoyées immédiatement après l'acceptation du message entrant (avant la réponse)
- les échecs sont journalises mais ne bloquent pas la livraison normale de la réponse
- le mode de groupe `mentions` reagit sur les tours declenches par mention ; l'activation de groupe `always` agit comme un contournement pour cette vérification
- WhatsApp utilisé `channels.whatsapp.ackReaction` (l'ancien `messages.ackReaction` n'est pas utilisé ici)

## Multi-comptes et identifiants

<AccordionGroup>
  <Accordion title="Sélection de compte et valeurs par défaut">
    - les identifiants de compte proviennent de `channels.whatsapp.accounts`
    - sélection de compte par défaut : `default` s'il est présent, sinon le premier identifiant de compte configuré (trie)
    - les identifiants de compte sont normalisés en interne pour la recherche
  </Accordion>

  <Accordion title="Chemins des identifiants et compatibilité historique">
    - chemin d'authentification actuel : `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - fichier de sauvegarde : `creds.json.bak`
    - l'authentification historique par défaut dans `~/.openclaw/credentials/` est toujours reconnue/migree pour les flux de compte par défaut
  </Accordion>

  <Accordion title="Comportement de déconnexion">
    `openclaw channels logout --channel whatsapp [--account <id>]` efface l'état d'authentification WhatsApp pour ce compte.

    Dans les répertoires d'authentification historiques, `oauth.json` est preserve tandis que les fichiers d'authentification Baileys sont supprimés.

  </Accordion>
</AccordionGroup>

## Outils, actions et écritures de configuration

- La prise en charge des outils d'agent inclut l'action de reaction WhatsApp (`react`).
- Contrôles d'action :
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Les écritures de configuration initiees par le canal sont activées par défaut (désactiver via `channels.whatsapp.configWrites=false`).

## Dépannage

<AccordionGroup>
  <Accordion title="Non lie (QR requis)">
    Symptôme : le statut du canal indique non lie.

    Correctif :

    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```

  </Accordion>

  <Accordion title="Lié mais déconnecté / boucle de reconnexion">
    Symptôme : compte lié avec des déconnexions ou des tentatives de reconnexion répétées.

    Correctif :

    ```bash
    openclaw doctor
    openclaw logs --follow
    ```

    Si nécessaire, reliez avec `channels login`.

  </Accordion>

  <Accordion title="Pas d'écouteur actif lors de l'envoi">
    Les envois sortants échouent immédiatement lorsqu'il n'existe aucun écouteur de gateway actif pour le compte cible.

    Assurez-vous que le gateway est en cours d'exécution et que le compte est lié.

  </Accordion>

  <Accordion title="Messages de groupe ignorés de maniere inattendue">
    Verifiez dans cet ordre :

    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - entrées de la liste autorisée `groups`
    - filtrage par mention (`requireMention` + modèles de mention)
    - clés en double dans `openclaw.json` (JSON5) : les entrées ultérieures remplacent les précédentes, donc gardez un seul `groupPolicy` par périmètre

  </Accordion>

  <Accordion title="Avertissement de runtime Bun">
    Le runtime du gateway WhatsApp doit utiliser Node. Bun est signale comme incompatible pour un fonctionnement stable du gateway WhatsApp/Telegram.
  </Accordion>
</AccordionGroup>

## Pointeurs de référence de configuration

Référence principale :

- [Référence de configuration - WhatsApp](/gateway/configuration-reference#whatsapp)

Champs WhatsApp a fort signal :

- accès : `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- livraison : `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-comptes : `accounts.<id>.enabled`, `accounts.<id>.authDir`, remplacements au niveau du compte
- opérations : `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- comportement de session : `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## Liens associés

- [Appairage](/channels/pairing)
- [Routage des canaux](/channels/channel-routing)
- [Routage multi-agent](/concepts/multi-agent)
- [Dépannage](/channels/troubleshooting)
