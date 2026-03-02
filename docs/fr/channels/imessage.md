---
summary: "Prise en charge ancienne d'iMessage via imsg (JSON-RPC sur stdio). Les nouvelles installations doivent utiliser BlueBubbles."
read_when:
  - Configuration de la prise en charge d'iMessage
  - Debogage de l'envoi/reception iMessage
title: "iMessage"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/imessage.md
  workflow: manual
---

# iMessage (ancien : imsg)

<Warning>
Pour les nouveaux deploiements iMessage, utilisez <a href="/channels/bluebubbles">BlueBubbles</a>.

L'intégration `imsg` est ancienne et pourra être supprimée dans une future version.
</Warning>

Statut : intégration CLI externe ancienne. Le Gateway lance `imsg rpc` et communique via JSON-RPC sur stdio (pas de daemon/port séparé).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommandé)" icon="message-circle" href="/channels/bluebubbles">
    Chemin iMessage préféré pour les nouvelles installations.
  </Card>
  <Card title="Appairage" icon="link" href="/channels/pairing">
    Les messages privés iMessage utilisent par défaut le mode appairage.
  </Card>
  <Card title="Référence de configuration" icon="settings" href="/gateway/configuration-reference#imessage">
    Référence complète des champs iMessage.
  </Card>
</CardGroup>

## Configuration rapide

<Tabs>
  <Tab title="Mac local (chemin rapide)">
    <Steps>
      <Step title="Installer et vérifier imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

      </Step>

      <Step title="Configurer OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

      </Step>

      <Step title="Démarrer le gateway">

```bash
openclaw gateway
```

      </Step>

      <Step title="Approuver le premier appairage de message privé (dmPolicy par défaut)">

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        Les demandes d'appairage expirent après 1 heure.
      </Step>
    </Steps>

  </Tab>

  <Tab title="Mac distant via SSH">
    OpenClaw ne nécessite qu'un `cliPath` compatible stdio, vous pouvez donc pointer `cliPath` vers un script wrapper qui se connecte en SSH a un Mac distant et exécute `imsg`.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    Configuration recommandee lorsque les pieces jointes sont activées :

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // utilise pour les recuperations de pieces jointes SCP
      includeAttachments: true,
      // Optionnel : remplacer les racines autorisees pour les pieces jointes.
      // Les valeurs par defaut incluent /Users/*/Library/Messages/Attachments
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
    },
  },
}
```

    Si `remoteHost` n'est pas défini, OpenClaw tente de le détecter automatiquement en analysant le script wrapper SSH.
    `remoteHost` doit être `host` ou `user@host` (pas d'espaces ni d'options SSH).
    OpenClaw utilise la vérification stricte de la clé hôte pour SCP, donc la clé hôte du relais doit déjà exister dans `~/.ssh/known_hosts`.
    Les chemins des pieces jointes sont valides par rapport aux racines autorisées (`attachmentRoots` / `remoteAttachmentRoots`).

  </Tab>
</Tabs>

## Prerequis et permissions (macOS)

- Messages doit être connecté sur le Mac executant `imsg`.
- L'accès complet au disque est requis pour le contexte de processus executant OpenClaw/`imsg` (accès a la base de donnees Messages).
- La permission d'automatisation est requise pour envoyer des messages via Messages.app.

<Tip>
Les permissions sont accordees par contexte de processus. Si le gateway s'exécute sans interface graphique (LaunchAgent/SSH), exécutez une commande interactive unique dans le même contexte pour déclencher les invites :

```bash
imsg chats --limit 1
# ou
imsg send <handle> "test"
```

</Tip>

## Contrôle d'accès et routage

<Tabs>
  <Tab title="Politique de messages privés">
    `channels.imessage.dmPolicy` contrôle les messages directs :

    - `pairing` (par défaut)
    - `allowlist`
    - `open` (nécessite que `allowFrom` inclue `"*"`)
    - `disabled`

    Champ de liste autorisée : `channels.imessage.allowFrom`.

    Les entrées de liste autorisée peuvent être des identifiants ou des cibles de discussion (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

  </Tab>

  <Tab title="Politique de groupe + mentions">
    `channels.imessage.groupPolicy` contrôle la gestion des groupes :

    - `allowlist` (par défaut lorsque configuré)
    - `open`
    - `disabled`

    Liste autorisée des expéditeurs de groupe : `channels.imessage.groupAllowFrom`.

    Repli d'exécution : si `groupAllowFrom` n'est pas défini, les vérifications d'expéditeurs de groupe iMessage se rabattent sur `allowFrom` lorsqu'il est disponible.
    Note d'exécution : si `channels.imessage` est complètement absent, l'exécution se rabat sur `groupPolicy="allowlist"` et journalise un avertissement (même si `channels.defaults.groupPolicy` est défini).

    Filtrage par mention pour les groupes :

    - iMessage n'a pas de métadonnées de mention natives
    - la détection de mention utilise les modèles regex (`agents.list[].groupChat.mentionPatterns`, repli `messages.groupChat.mentionPatterns`)
    - sans modèles configurés, le filtrage par mention ne peut pas être applique

    Les commandes de contrôle des expéditeurs autorisés peuvent contourner le filtrage par mention dans les groupes.

  </Tab>

  <Tab title="Sessions et réponses deterministes">
    - Les messages privés utilisent le routage direct ; les groupes utilisent le routage de groupe.
    - Avec la valeur par défaut `session.dmScope=main`, les messages privés iMessage se regroupent dans la session principale de l'agent.
    - Les sessions de groupe sont isolees (`agent:<agentId>:imessage:group:<chat_id>`).
    - Les réponses sont acheminees vers iMessage en utilisant les métadonnées de canal/cible d'origine.

    Comportement de type groupe :

    Certains fils iMessage multi-participants peuvent arriver avec `is_group=false`.
    Si ce `chat_id` est explicitement configuré sous `channels.imessage.groups`, OpenClaw le traite comme du trafic de groupe (filtrage de groupe + isolation de session de groupe).

  </Tab>
</Tabs>

## Modèles de déploiement

<AccordionGroup>
  <Accordion title="Utilisateur macOS de bot dédié (identité iMessage séparée)">
    Utilisez un Apple ID et un utilisateur macOS dédiés pour que le trafic du bot soit isole de votre profil Messages personnel.

    Flux typique :

    1. Creez/connectez un utilisateur macOS dédié.
    2. Connectez-vous a Messages avec l'Apple ID du bot dans cet utilisateur.
    3. Installez `imsg` dans cet utilisateur.
    4. Creez un wrapper SSH pour qu'OpenClaw puisse exécuter `imsg` dans le contexte de cet utilisateur.
    5. Pointez `channels.imessage.accounts.<id>.cliPath` et `.dbPath` vers ce profil utilisateur.

    La première exécution peut necessiter des approbations via l'interface graphique (Automatisation + Accès complet au disque) dans cette session utilisateur de bot.

  </Accordion>

  <Accordion title="Mac distant via Tailscale (exemple)">
    Topologie courante :

    - le gateway s'exécute sur Linux/VM
    - iMessage + `imsg` s'exécute sur un Mac dans votre tailnet
    - le wrapper `cliPath` utilisé SSH pour exécuter `imsg`
    - `remoteHost` activé les recuperations de pieces jointes SCP

    Exemple :

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    Utilisez des clés SSH pour que SSH et SCP soient non interactifs.
    Assurez-vous que la clé hôte est approuvee en premier (par exemple `ssh bot@mac-mini.tailnet-1234.ts.net`) pour que `known_hosts` soit renseigne.

  </Accordion>

  <Accordion title="Modèle multi-comptes">
    iMessage prend en charge la configuration par compte sous `channels.imessage.accounts`.

    Chaque compte peut remplacer des champs tels que `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, les paramètres d'historique et les listes autorisées de racines de pieces jointes.

  </Accordion>
</AccordionGroup>

## Médias, découpage et cibles de livraison

<AccordionGroup>
  <Accordion title="Pieces jointes et médias">
    - l'ingestion de pieces jointes entrantes est optionnelle : `channels.imessage.includeAttachments`
    - les chemins de pieces jointes distantes peuvent être récupérés via SCP lorsque `remoteHost` est défini
    - les chemins de pieces jointes doivent correspondre aux racines autorisées :
      - `channels.imessage.attachmentRoots` (local)
      - `channels.imessage.remoteAttachmentRoots` (mode SCP distant)
      - modèle de racine par défaut : `/Users/*/Library/Messages/Attachments`
    - SCP utilisé la vérification stricte de la clé hôte (`StrictHostKeyChecking=yes`)
    - la taille des médias sortants utilisé `channels.imessage.mediaMaxMb` (par défaut 16 Mo)
  </Accordion>

  <Accordion title="Découpage sortant">
    - limité de découpage du texte : `channels.imessage.textChunkLimit` (par défaut 4000)
    - mode de découpage : `channels.imessage.chunkMode`
      - `length` (par défaut)
      - `newline` (découpage par paragraphe en priorité)
  </Accordion>

  <Accordion title="Formats d'adressage">
    Cibles explicites preferees :

    - `chat_id:123` (recommandé pour un routage stable)
    - `chat_guid:...`
    - `chat_identifier:...`

    Les cibles par identifiant sont également prises en charge :

    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`

```bash
imsg chats --limit 20
```

  </Accordion>
</AccordionGroup>

## Écritures de configuration

iMessage autorisé les écritures de configuration initiees par le canal par défaut (pour `/config set|unset` lorsque `commands.config: true`).

Désactiver :

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Dépannage

<AccordionGroup>
  <Accordion title="imsg introuvable ou RPC non pris en charge">
    Validez le binaire et le support RPC :

```bash
imsg rpc --help
openclaw channels status --probe
```

    Si la sonde indique que RPC n'est pas pris en charge, mettez a jour `imsg`.

  </Accordion>

  <Accordion title="Les messages privés sont ignorés">
    Verifiez :

    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - les approbations d'appairage (`openclaw pairing list imessage`)

  </Accordion>

  <Accordion title="Les messages de groupe sont ignorés">
    Verifiez :

    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - le comportement de la liste autorisée `channels.imessage.groups`
    - la configuration des modèles de mention (`agents.list[].groupChat.mentionPatterns`)

  </Accordion>

  <Accordion title="Les pieces jointes distantes échouent">
    Verifiez :

    - `channels.imessage.remoteHost`
    - `channels.imessage.remoteAttachmentRoots`
    - l'authentification par clé SSH/SCP depuis l'hôte du gateway
    - la clé hôte existant dans `~/.ssh/known_hosts` sur l'hôte du gateway
    - la lisibilite du chemin distant sur le Mac executant Messages

  </Accordion>

  <Accordion title="Les invites de permission macOS ont ete manquees">
    Re-exécutez dans un terminal GUI interactif dans le même contexte utilisateur/session et approuvez les invites :

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    Confirmez que l'accès complet au disque + l'automatisation sont accordés pour le contexte de processus qui exécute OpenClaw/`imsg`.

  </Accordion>
</AccordionGroup>

## Pointeurs de référence de configuration

- [Référence de configuration - iMessage](/gateway/configuration-reference#imessage)
- [Configuration du Gateway](/gateway/configuration)
- [Appairage](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
