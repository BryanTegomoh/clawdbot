---
summary: "Règles de gestion des sessions, clés et persistance pour les chats"
read_when:
  - Modification de la gestion ou du stockage des sessions
title: "Gestion des sessions"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: concepts/session.md
  workflow: manual
---

# Gestion des sessions

OpenClaw traite **une session de conversation directe par agent** comme principale. Les conversations directes se replient vers `agent:<agentId>:<mainKey>` (par défaut `main`), tandis que les chats de groupe/canal obtiennent leurs propres clés. `session.mainKey` est respecté.

Utilisez `session.dmScope` pour contrôler comment les **messages directs** sont regroupés :

- `main` (par défaut) : tous les DM partagent la session principale pour la continuité.
- `per-peer` : isoler par identifiant d'expéditeur à travers les canaux.
- `per-channel-peer` : isoler par canal + expéditeur (recommandé pour les boîtes de réception multi-utilisateurs).
- `per-account-channel-peer` : isoler par compte + canal + expéditeur (recommandé pour les boîtes de réception multi-comptes).
  Utilisez `session.identityLinks` pour mapper les identifiants de pairs préfixés par fournisseur à une identité canonique afin que la même personne partage une session DM entre les canaux avec `per-peer`, `per-channel-peer` ou `per-account-channel-peer`.

## Mode DM sécurisé (recommandé pour les configurations multi-utilisateurs)

> **Avertissement de sécurité :** Si votre agent peut recevoir des DM de **plusieurs personnes**, vous devriez sérieusement envisager d'activer le mode DM sécurisé. Sans celui-ci, tous les utilisateurs partagent le même contexte de conversation, ce qui peut entraîner des fuites d'informations privées entre utilisateurs.

**Exemple du problème avec les paramètres par défaut :**

- Alice (`<SENDER_A>`) envoie un message à votre agent sur un sujet privé (par exemple, un rendez-vous médical)
- Bob (`<SENDER_B>`) envoie un message à votre agent en demandant « De quoi parlions-nous ? »
- Comme les deux DM partagent la même session, le modèle peut répondre à Bob en utilisant le contexte précédent d'Alice.

**La solution :** Définissez `dmScope` pour isoler les sessions par utilisateur :

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    // Mode DM sécurisé : isoler le contexte DM par canal + expéditeur.
    dmScope: "per-channel-peer",
  },
}
```

**Quand activer ce mode :**

- Vous avez des approbations d'appairage pour plus d'un expéditeur
- Vous utilisez une liste autorisée de DM avec plusieurs entrées
- Vous définissez `dmPolicy: "open"`
- Plusieurs numéros de téléphone ou comptes peuvent envoyer des messages à votre agent

Notes :

- Par défaut, `dmScope: "main"` assuré la continuité (tous les DM partagent la session principale). Cela convient pour les configurations mono-utilisateur.
- La configuration initiale en CLI local écrit `session.dmScope: "per-channel-peer"` par défaut lorsque la valeur n'est pas définie (les valeurs explicites existantes sont préservées).
- Pour les boîtes de réception multi-comptes sur le même canal, préférez `per-account-channel-peer`.
- Si la même personne vous contacte sur plusieurs canaux, utilisez `session.identityLinks` pour fusionner ses sessions DM en une seule identité canonique.
- Vous pouvez vérifier vos paramètres DM avec `openclaw security audit` (voir [sécurité](/cli/security)).

## Le Gateway est la source de vérité

Tout l'état de session est **détenu par le Gateway** (le « maître » OpenClaw). Les clients UI (application macOS, WebChat, etc.) doivent interroger le Gateway pour obtenir la liste des sessions et le nombre de tokens au lieu de lire les fichiers locaux.

- En **mode distant**, le stockage de session qui vous intéresse se trouve sur l'hôte du Gateway distant, pas sur votre Mac.
- Les compteurs de tokens affichés dans les interfaces proviennent des champs du stockage du Gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Les clients n'analysent pas les transcriptions JSONL pour « corriger » les totaux.

## Où réside l'état

- Sur l'**hôte du Gateway** :
  - Fichier de stockage : `~/.openclaw/agents/<agentId>/sessions/sessions.json` (par agent).
- Transcriptions : `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (les sessions de topics Telegram utilisent `.../<SessionId>-topic-<threadId>.jsonl`).
- Le stockage est un mappage `sessionKey -> { sessionId, updatedAt, ... }`. Supprimer des entrées est sans risque ; elles sont recréées à la demande.
- Les entrées de groupe peuvent inclure `displayName`, `channel`, `subject`, `room` et `space` pour étiqueter les sessions dans les interfaces.
- Les entrées de session incluent des métadonnées `origin` (étiquette + indications de routage) pour que les interfaces puissent expliquer d'où provient une session.
- OpenClaw ne lit **pas** les dossiers de session hérités de Pi/Tau.

## Maintenance

OpenClaw applique une maintenance du stockage de sessions pour maintenir `sessions.json` et les artefacts de transcription dans des limités raisonnables au fil du temps.

### Valeurs par défaut

- `session.maintenance.mode` : `warn`
- `session.maintenance.pruneAfter` : `30d`
- `session.maintenance.maxEntries` : `500`
- `session.maintenance.rotateBytes` : `10mb`
- `session.maintenance.resetArchiveRetention` : par défaut identique à `pruneAfter` (`30d`)
- `session.maintenance.maxDiskBytes` : non défini (désactivé)
- `session.maintenance.highWaterBytes` : par défaut `80%` de `maxDiskBytes` lorsque le budget disque est activé

### Fonctionnement

La maintenance s'exécute lors des écritures dans le stockage de sessions, et vous pouvez la déclencher à la demande avec `openclaw sessions cleanup`.

- `mode: "warn"` : signale ce qui serait supprimé mais ne modifie pas les entrées/transcriptions.
- `mode: "enforce"` : applique le nettoyage dans cet ordre :
  1. élaguer les entrées obsolètes plus anciennes que `pruneAfter`
  2. limiter le nombre d'entrées à `maxEntries` (les plus anciennes en premier)
  3. archiver les fichiers de transcription pour les entrées supprimées qui ne sont plus référencées
  4. purger les anciennes archives `*.deleted.<timestamp>` et `*.reset.<timestamp>` selon la politique de rétention
  5. effectuer une rotation de `sessions.json` lorsqu'il dépasse `rotateBytes`
  6. si `maxDiskBytes` est défini, appliquer le budget disque vers `highWaterBytes` (artefacts les plus anciens en premier, puis sessions les plus anciennes)

### Avertissement de performance pour les grands stockages

Les grands stockages de sessions sont courants dans les installations à fort volume. Le travail de maintenance s'effectué sur le chemin d'écriture, donc les très grands stockages peuvent augmenter la latence d'écriture.

Ce qui augmente le plus le coût :

- des valeurs très élevées pour `session.maintenance.maxEntries`
- de longues fenêtres `pruneAfter` qui conservent les entrées obsolètes
- de nombreux artefacts de transcription/archive dans `~/.openclaw/agents/<agentId>/sessions/`
- l'activation des budgets disque (`maxDiskBytes`) sans limités raisonnables d'élagage/plafonnement

Que faire :

- utilisez `mode: "enforce"` en production pour que la croissance soit automatiquement limitée
- définissez à la fois des limités temporelles et de nombre (`pruneAfter` + `maxEntries`), pas une seule
- définissez `maxDiskBytes` + `highWaterBytes` pour des limités strictes dans les grands déploiements
- gardez `highWaterBytes` significativement en dessous de `maxDiskBytes` (la valeur par défaut est 80%)
- exécutez `openclaw sessions cleanup --dry-run --json` après des modifications de configuration pour vérifier l'impact prévu avant d'appliquer
- pour les sessions fréquemment activés, passez `--activated-key` lors du nettoyage manuel

### Exemples de personnalisation

Utiliser une politique d'application conservatrice :

```json5
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "45d",
      maxEntries: 800,
      rotateBytes: "20mb",
      resetArchiveRetention: "14d",
    },
  },
}
```

Activer un budget disque strict pour le répertoire de sessions :

```json5
{
  session: {
    maintenance: {
      mode: "enforce",
      maxDiskBytes: "1gb",
      highWaterBytes: "800mb",
    },
  },
}
```

Configurer pour des installations plus importantes (exemple) :

```json5
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "14d",
      maxEntries: 2000,
      rotateBytes: "25mb",
      maxDiskBytes: "2gb",
      highWaterBytes: "1.6gb",
    },
  },
}
```

Prévisualiser ou forcer la maintenance depuis le CLI :

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

## Élagage de session

OpenClaw supprime les **anciens résultats d'outils** du contexte en mémoire juste avant les appels au LLM par défaut.
Cela ne **réécrit pas** l'historique JSONL. Voir [/concepts/session-pruning](/concepts/session-pruning).

## Vidage mémoire avant compaction

Lorsqu'une session approche de la compaction automatique, OpenClaw peut exécuter un **tour silencieux de vidage mémoire**
qui rappelle au modèle d'écrire des notes durables sur le disque. Cela ne s'exécute que lorsque
l'espace de travail est accessible en écriture. Voir [Mémoire](/concepts/memory) et
[Compaction](/concepts/compaction).

## Correspondance transports vers clés de session

- Les conversations directes suivent `session.dmScope` (par défaut `main`).
  - `main` : `agent:<agentId>:<mainKey>` (continuité entre appareils/canaux).
    - Plusieurs numéros de téléphone et canaux peuvent correspondre à la même clé principale d'agent ; ils agissent comme des transports vers une seule conversation.
  - `per-peer` : `agent:<agentId>:dm:<peerId>`.
  - `per-channel-peer` : `agent:<agentId>:<channel>:dm:<peerId>`.
  - `per-account-channel-peer` : `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (accountId par défaut : `default`).
  - Si `session.identityLinks` correspond à un identifiant de pair préfixé par fournisseur (par exemple `telegram:123`), la clé canonique remplace `<peerId>` pour que la même personne partage une session entre les canaux.
- Les chats de groupe isolent l'état : `agent:<agentId>:<channel>:group:<id>` (les salons/canaux utilisent `agent:<agentId>:<channel>:channel:<id>`).
  - Les topics de forum Telegram ajoutent `:topic:<threadId>` à l'identifiant de groupe pour l'isolation.
  - Les clés héritées `group:<id>` sont toujours reconnues pour la migration.
- Les contextes entrants peuvent encore utiliser `group:<id>` ; le canal est déduit du `Provider` et normalisé vers la forme canonique `agent:<agentId>:<channel>:group:<id>`.
- Autres sources :
  - Tâches cron : `cron:<job.id>`
  - Webhooks : `hook:<uuid>` (sauf si explicitement défini par le webhook)
  - Exécutions de nœuds : `node-<nodeId>`

## Cycle de vie

- Politique de réinitialisation : les sessions sont réutilisées jusqu'à leur expiration, et l'expiration est évaluée lors du prochain message entrant.
- Réinitialisation quotidienne : par défaut **4h00 heure locale sur l'hôte du Gateway**. Une session est obsolète lorsque sa dernière mise à jour est antérieure à la dernière heure de réinitialisation quotidienne.
- Réinitialisation par inactivité (optionnel) : `idleMinutes` ajouté une fenêtre d'inactivité glissante. Lorsque les réinitialisations quotidienne et par inactivité sont configurées, **la première à expirer** force une nouvelle session.
- Mode inactivité seule (hérité) : si vous définissez `session.idleMinutes` sans aucune configuration `session.reset`/`resetByType`, OpenClaw reste en mode inactivité seule pour la rétrocompatibilité.
- Surcharges par type (optionnel) : `resetByType` vous permet de surcharger la politique pour les sessions `direct`, `group` et `thread` (thread = fils Slack/Discord, topics Telegram, fils Matrix lorsque fournis par le connecteur).
- Surcharges par canal (optionnel) : `resetByChannel` surcharge la politique de réinitialisation pour un canal (s'applique à tous les types de session pour ce canal et prend le pas sur `reset`/`resetByType`).
- Déclencheurs de réinitialisation : `/new` ou `/reset` exact (plus tout extra dans `resetTriggers`) démarrent un nouvel identifiant de session et transmettent le reste du message. `/new <model>` accepte un alias de modèle, `provider/model`, ou un nom de fournisseur (correspondance approximative) pour définir le modèle de la nouvelle session. Si `/new` ou `/reset` est envoyé seul, OpenClaw exécute un court tour de salutation pour confirmer la réinitialisation.
- Réinitialisation manuelle : supprimez des clés spécifiques du stockage ou supprimez la transcription JSONL ; le prochain message les recrée.
- Les tâches cron isolées créent toujours un nouveau `sessionId` par exécution (pas de réutilisation par inactivité).

## Politique d'envoi (optionnel)

Bloquer la livraison pour des types de session spécifiques sans lister les identifiants individuels.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // Correspondance sur la clé de session brute (incluant le préfixe `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

Surcharge à l'exécution (propriétaire uniquement) :

- `/send on` : autoriser pour cette session
- `/send off` : refuser pour cette session
- `/send inherit` : supprimer la surcharge et utiliser les règles de configuration
  Envoyez ces commandes comme messages autonomes pour qu'elles soient prises en compte.

## Configuration (exemple de renommage optionnel)

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // garder les clés de groupe séparées
    dmScope: "main", // continuité DM (définir per-channel-peer/per-account-channel-peer pour les boîtes partagées)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // Valeurs par défaut : mode=daily, atHour=4 (heure locale de l'hôte du Gateway).
      // Si vous définissez aussi idleMinutes, la première expiration gagne.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## Inspection

- `openclaw status` : affiche le chemin de stockage et les sessions récentes.
- `openclaw sessions --json` : exporte toutes les entrées (filtrer avec `--activated <minutes>`).
- `openclaw gateway call sessions.list --params '{}'` : récupère les sessions depuis le Gateway en cours d'exécution (utilisez `--url`/`--token` pour l'accès au Gateway distant).
- Envoyez `/status` comme message autonome dans le chat pour voir si l'agent est joignable, quelle proportion du contexte de session est utilisée, les bascules thinking/verbose actuelles, et quand vos identifiants web WhatsApp ont été rafraîchis pour la dernière fois (aide à repérer les besoins de reconnexion).
- Envoyez `/context list` ou `/context detail` pour voir ce qui se trouve dans le prompt système et les fichiers d'espace de travail injectés (et les plus gros contributeurs au contexte).
- Envoyez `/stop` (ou des phrases d'arrêt autonomes comme `stop`, `stop action`, `stop run`, `stop openclaw`) pour interrompre l'exécution en cours, vider les suivis en file d'attente pour cette session, et arrêter toute exécution de sous-agent lancée depuis celle-ci (la réponse inclut le nombre d'arrêts).
- Envoyez `/compact` (instructions optionnelles) comme message autonome pour résumer le contexte ancien et libérer de l'espace dans la fenêtre. Voir [/concepts/compaction](/concepts/compaction).
- Les transcriptions JSONL peuvent être ouvertes directement pour examiner les tours complets.

## Conseils

- Réservez la clé principale au trafic 1:1 ; laissez les groupes garder leurs propres clés.
- Lors de l'automatisation du nettoyage, supprimez les clés individuelles au lieu du stockage entier pour préserver le contexte ailleurs.

## Métadonnées d'origine de session

Chaque entrée de session enregistre sa provenance (au mieux) dans `origin` :

- `label` : étiquette lisible (résolue à partir de l'étiquette de conversation + sujet/canal du groupe)
- `provider` : identifiant de canal normalisé (extensions incluses)
- `from`/`to` : identifiants de routage bruts provenant de l'enveloppe entrante
- `accountId` : identifiant de compte du fournisseur (en multi-comptes)
- `threadId` : identifiant de fil/topic lorsque le canal le prend en charge
  Les champs origin sont renseignés pour les messages directs, les canaux et les groupes. Si un
  connecteur ne met à jour que le routage de livraison (par exemple, pour maintenir une session DM principale
  à jour), il devrait quand même fournir le contexte entrant pour que la session conserve ses
  métadonnées explicatives. Les extensions peuvent le faire en envoyant `ConversationLabel`,
  `GroupSubject`, `GroupChannel`, `GroupSpace` et `SenderName` dans le contexte
  entrant et en appelant `recordSessionMetaFromInbound` (ou en passant le même contexte
  à `updateLastRoute`).
