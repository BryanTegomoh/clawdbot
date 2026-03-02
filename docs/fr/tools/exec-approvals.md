---
summary: "Approbations exec, listes d'autorisation et invites d'échappement du bac à sable"
read_when:
  - Configuration des approbations exec ou des listes d'autorisation
  - Implémentation de l'interface d'approbation exec dans l'application macOS
  - Examen des invites d'échappement du bac à sable et leurs implications
title: "Approbations Exec"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/exec-approvals.md
  workflow: manual
---

# Approbations exec

Les approbations exec sont le **garde-fou de l'application compagnon / node host** pour permettre à un agent sandboxé d'exécuter des commandes sur un vrai hôte (`gateway` ou `node`). Considérez-les comme un verrouillage de sécurité : les commandes ne sont autorisées que quand la politique + la liste d'autorisation + l'approbation utilisateur (optionnelle) sont toutes d'accord. Les approbations exec sont **en plus** de la politique d'outils et du gating elevated (sauf si elevated est défini à `full`, ce qui ignore les approbations). La politique effective est la **plus stricte** entre `tools.exec.*` et les défauts des approbations ; si un champ d'approbation est omis, la valeur `tools.exec` est utilisée.

Si l'interface de l'application compagnon n'est **pas disponible**, toute requête nécessitant une invite est résolue par le **repli ask** (défaut : deny).

## Où cela s'applique

Les approbations exec sont appliquées localement sur l'hôte d'exécution :

- **hôte gateway** → processus `openclaw` sur la machine du gateway
- **node host** → runner de node (application compagnon macOS ou node host headless)

Note sur le modèle de confiance :

- Les appelants authentifiés par le Gateway sont des opérateurs de confiance pour ce Gateway.
- Les nodes appariés étendent cette capacité d'opérateur de confiance sur le node host.
- Les approbations exec réduisent le risque d'exécution accidentelle, mais ne constituent pas une frontière d'authentification par utilisateur.

Division macOS :

- Le **service node host** transmet `system.run` à l'**application macOS** via IPC local.
- L'**application macOS** applique les approbations + exécute la commande dans le contexte de l'interface.

## Paramètres et stockage

Les approbations résident dans un fichier JSON local sur l'hôte d'exécution :

`~/.openclaw/exec-approvals.json`

Exemple de schéma :

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## Boutons de politique

### Security (`exec.security`)

- **deny** : bloquer toutes les requêtes exec hôte.
- **allowlist** : autoriser uniquement les commandes dans la liste d'autorisation.
- **full** : tout autoriser (équivalent au mode elevated).

### Ask (`exec.ask`)

- **off** : ne jamais demander.
- **on-miss** : demander uniquement quand la liste d'autorisation ne correspond pas.
- **always** : demander à chaque commande.

### Repli ask (`askFallback`)

Si une invite est requise mais qu'aucune interface n'est joignable, le repli décide :

- **deny** : bloquer.
- **allowlist** : autoriser uniquement si la liste d'autorisation correspond.
- **full** : autoriser.

## Liste d'autorisation (par agent)

Les listes d'autorisation sont **par agent**. Si plusieurs agents existent, basculez l'agent que vous modifiez dans l'application macOS. Les patterns sont des **correspondances glob insensibles à la casse**. Les patterns doivent résoudre vers des **chemins de binaires** (les entrées uniquement par nom de base sont ignorées). Les entrées héritées `agents.default` sont migrées vers `agents.main` au chargement.

Exemples :

- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

Chaque entrée de la liste d'autorisation suit :

- **id** UUID stable utilisé pour l'identité dans l'interface (optionnel)
- **last used** horodatage
- **last used command**
- **last resolved path**

## Auto-autorisation des CLI de Skills

Quand **Auto-allow skill CLIs** est activé, les exécutables référencés par les Skills connus sont traités comme dans la liste d'autorisation sur les nodes (node macOS ou node host headless). Ceci utilise `skills.bins` via le RPC du Gateway pour récupérer la liste de binaires des Skills. Désactivez ceci si vous voulez des listes d'autorisation manuelles strictes.

Notes importantes sur la confiance :

- C'est une **liste d'autorisation implicite de commodité**, séparée des entrées manuelles de la liste d'autorisation par chemin.
- Elle est prévue pour les environnements d'opérateurs de confiance où le Gateway et le node sont dans la même frontière de confiance.
- Si vous exigez une confiance explicite stricte, gardez `autoAllowSkills: false` et utilisez uniquement des entrées manuelles de liste d'autorisation par chemin.

## Safe bins (stdin uniquement)

`tools.exec.safeBins` définit une petite liste de binaires **stdin uniquement** (par exemple `jq`) qui peuvent s'exécuter en mode liste d'autorisation **sans** entrées de liste d'autorisation explicites. Les safe bins rejettent les arguments de fichier positionnels et les tokens de type chemin, donc ils ne peuvent opérer que sur le flux entrant. Traitez ceci comme un chemin rapide étroit pour les filtrés de flux, pas une liste de confiance générale. N'ajoutez **pas** de binaires interpréteurs ou runtime (par exemple `python3`, `node`, `ruby`, `bash`, `sh`, `zsh`) aux `safeBins`. Si une commande peut évaluer du code, exécuter des sous-commandes, ou lire des fichiers par conception, préférez les entrées de liste d'autorisation explicites et gardez les invites d'approbation activées. Les safe bins personnalisés doivent définir un profil explicite dans `tools.exec.safeBinProfiles.<bin>`. La validation est déterministe à partir de la forme argv uniquement (pas de vérification d'existence de fichier sur le système de fichiers hôte), ce qui empêche le comportement d'oracle d'existence de fichier à partir des différences autoriser/refuser. Les options orientées fichier sont refusées pour les safe bins par défaut (par exemple `sort -o`, `sort --output`, `sort --files0-from`, `sort --compress-program`, `sort --random-source`, `sort --temporary-directory`/`-T`, `wc --files0-from`, `jq -f/--from-file`, `grep -f/--file`). Les safe bins appliquent aussi une politique de flags explicite par binaire pour les options qui cassent le comportement stdin uniquement (par exemple `sort -o/--output/--compress-program` et les flags récursifs de grep). Les options longues sont validées en fail-closed en mode safe-bin : les flags inconnus et les abréviations ambiguës sont rejetés. Flags refusés par profil safe-bin :

<!-- SAFE_BIN_DENIED_FLAGS:START -->

- `grep` : `--dereference-recursive`, `--directories`, `--exclude-from`, `--file`, `--recursive`, `-R`, `-d`, `-f`, `-r`
- `jq` : `--argfile`, `--from-file`, `--library-path`, `--rawfile`, `--slurpfile`, `-L`, `-f`
- `sort` : `--compress-program`, `--files0-from`, `--output`, `--random-source`, `--temporary-directory`, `-T`, `-o`
- `wc` : `--files0-from`
<!-- SAFE_BIN_DENIED_FLAGS:END -->

Les safe bins forcent aussi les tokens argv à être traités comme du **texte littéral** au moment de l'exécution (pas de globbing et pas d'expansion de `$VARS`) pour les segments stdin uniquement, donc les patterns comme `*` ou `$HOME/...` ne peuvent pas être utilisés pour contourner les lectures de fichiers. Les safe bins doivent aussi résoudre depuis des répertoires de binaires de confiance (défauts système plus `tools.exec.safeBinTrustedDirs` optionnel). Les entrées `PATH` ne sont jamais auto-approuvées. Les répertoires de confiance par défaut pour les safe bins sont intentionnellement minimaux : `/bin`, `/usr/bin`. Si votre exécutable safe-bin réside dans des chemins de gestionnaire de paquets/utilisateur (par exemple `/opt/homebrew/bin`, `/usr/local/bin`, `/opt/local/bin`, `/snap/bin`), ajoutez-les explicitement à `tools.exec.safeBinTrustedDirs`. Le chaînage shell et les redirections ne sont pas auto-autorisés en mode liste d'autorisation.

Le chaînage shell (`&&`, `||`, `;`) est autorisé quand chaque segment de niveau supérieur satisfait la liste d'autorisation (y compris les safe bins ou l'auto-autorisation des Skills). Les redirections restent non supportées en mode liste d'autorisation. La substitution de commande (`$()` / backticks) est rejetée lors de l'analyse de la liste d'autorisation, y compris à l'intérieur des guillemets doubles ; utilisez des guillemets simples si vous avez besoin de texte `$()` littéral. Sur les approbations de l'application compagnon macOS, le texte shell brut contenant de la syntaxe de contrôle ou d'expansion shell (`&&`, `||`, `;`, `|`, `` ` ``, `$`, `<`, `>`, `(`, `)`) est traité comme un miss de liste d'autorisation sauf si le binaire shell lui-même est dans la liste d'autorisation. Pour les wrappers shell (`bash|sh|zsh ... -c/-lc`), les surcharges d'environnement limitées à la requête sont réduites à une petite liste d'autorisation explicite (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`). Pour les décisions d'autorisation permanente en mode liste d'autorisation, les wrappers de dispatch connus (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) persistent les chemins d'exécutables internes au lieu des chemins de wrappers. Les multiplexeurs shell (`busybox`, `toybox`) sont aussi déballés pour les applets shell (`sh`, `ash`, etc.) pour que les exécutables internes soient persistés au lieu des binaires multiplexeurs. Si un wrapper ou multiplexeur ne peut pas être déballé en toute sécurité, aucune entrée de liste d'autorisation n'est persistée automatiquement.

Safe bins par défaut : `jq`, `cut`, `uniq`, `head`, `tail`, `tr`, `wc`.

`grep` et `sort` ne sont pas dans la liste par défaut. Si vous les ajoutez, gardez des entrées de liste d'autorisation explicites pour leurs workflows non-stdin. Pour `grep` en mode safe-bin, fournissez le pattern avec `-e`/`--regexp` ; la forme positionnelle du pattern est rejetée pour que les opérandes de fichier ne puissent pas être introduits comme des positionnels ambigus.

### Safe bins versus liste d'autorisation

| Sujet             | `tools.exec.safeBins`                                    | Liste d'autorisation (`exec-approvals.json`)                   |
| ----------------- | -------------------------------------------------------- | -------------------------------------------------------------- |
| Objectif          | Auto-autoriser les filtrés stdin étroits                  | Faire explicitement confiance à des exécutables spécifiques     |
| Type de correspondance | Nom de l'exécutable + politique argv safe-bin          | Pattern glob de chemin d'exécutable résolu                      |
| Portée des arguments | Restreint par le profil safe-bin et les règles de token littéral | Correspondance de chemin uniquement ; les arguments sont de votre responsabilité |
| Exemples typiques | `jq`, `head`, `tail`, `wc`                               | `python3`, `node`, `ffmpeg`, CLI personnalisés                  |
| Meilleure utilisation | Transformations de texte à faible risque dans les pipelines | Tout outil avec un comportement plus large ou des effets de bord |

Emplacement de la configuration :

- `safeBins` vient de la configuration (`tools.exec.safeBins` ou par agent `agents.list[].tools.exec.safeBins`).
- `safeBinTrustedDirs` vient de la configuration (`tools.exec.safeBinTrustedDirs` ou par agent `agents.list[].tools.exec.safeBinTrustedDirs`).
- `safeBinProfiles` vient de la configuration (`tools.exec.safeBinProfiles` ou par agent `agents.list[].tools.exec.safeBinProfiles`). Les clés de profil par agent substituent les clés globales.
- Les entrées de liste d'autorisation résident dans le fichier local sur l'hôte `~/.openclaw/exec-approvals.json` sous `agents.<id>.allowlist` (ou via l'interface de contrôle / `openclaw approvals allowlist ...`).
- `openclaw security audit` avertit avec `tools.exec.safe_bins_interpreter_unprofiled` quand des binaires interpréteurs/runtime apparaissent dans `safeBins` sans profils explicites.
- `openclaw doctor --fix` peut créer les entrées `safeBinProfiles.<bin>` personnalisées manquantes comme `{}` (examinez et renforcez ensuite). Les binaires interpréteurs/runtime ne sont pas auto-créés.

Exemple de profil personnalisé :

```json5
{
  tools: {
    exec: {
      safeBins: ["jq", "myfilter"],
      safeBinProfiles: {
        myfilter: {
          minPositional: 0,
          maxPositional: 0,
          allowedValueFlags: ["-n", "--limit"],
          deniedFlags: ["-f", "--file", "-c", "--command"],
        },
      },
    },
  },
}
```

## Édition via l'interface de contrôle

Utilisez la carte **Interface de contrôle → Nodes → Approbations exec** pour modifier les défauts, les substitutions par agent et les listes d'autorisation. Choisissez une portée (Défauts ou un agent), ajustez la politique, ajoutez/supprimez des patterns de liste d'autorisation, puis **Enregistrez**. L'interface affiche les métadonnées de **dernière utilisation** par pattern pour que vous puissiez garder la liste propre.

Le sélecteur de cible choisit **Gateway** (approbations locales) ou un **Node**. Les nodes doivent annoncer `system.execApprovals.get/set` (application macOS ou node host headless). Si un node n'annonce pas encore les approbations exec, modifiez directement son fichier local `~/.openclaw/exec-approvals.json`.

CLI : `openclaw approvals` supporte l'édition gateway ou node (voir [CLI Approbations](/cli/approvals)).

## Flux d'approbation

Quand une invite est requise, le gateway diffuse `exec.approval.requested` aux clients opérateurs. L'interface de contrôle et l'application macOS le résolvent via `exec.approval.resolve`, puis le gateway transmet la requête approuvée au node host.

Quand des approbations sont requises, l'outil exec retourne immédiatement avec un id d'approbation. Utilisez cet id pour corréler les événements système ultérieurs (`Exec finished` / `Exec denied`). Si aucune décision n'arrive avant le timeout, la requête est traitée comme un timeout d'approbation et présentée comme raison de refus.

Le dialogue de confirmation inclut :

- commande + arguments
- cwd
- id d'agent
- chemin d'exécutable résolu
- métadonnées hôte + politique

Actions :

- **Allow once** → exécuter maintenant
- **Always allow** → ajouter à la liste d'autorisation + exécuter
- **Deny** → bloquer

## Transfert des approbations aux canaux de chat

Vous pouvez transférer les invites d'approbation exec à n'importe quel canal de chat (y compris les canaux de plugins) et les approuver avec `/approve`. Ceci utilise le pipeline de livraison sortante normal.

Configuration :

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // sous-chaîne ou regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

Répondez dans le chat :

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

### Flux IPC macOS

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Notes de sécurité :

- Socket Unix mode `0600`, token stocké dans `exec-approvals.json`.
- Vérification du pair même-UID.
- Challenge/réponse (nonce + HMAC token + hash de la requête) + TTL court.

## Événements système

Le cycle de vie exec est exposé comme messages système :

- `Exec running` (uniquement si la commande dépasse le seuil d'avis d'exécution)
- `Exec finished`
- `Exec denied`

Ceux-ci sont publiés dans la session de l'agent après que le node rapporte l'événement. Les approbations exec de l'hôte gateway émettent les mêmes événements de cycle de vie quand la commande se termine (et optionnellement quand elle dure plus longtemps que le seuil). Les execs avec approbation réutilisent l'id d'approbation comme `runId` dans ces messages pour faciliter la corrélation.

## Implications

- **full** est puissant ; préférez les listes d'autorisation quand possible.
- **ask** vous garde dans la boucle tout en permettant des approbations rapides.
- Les listes d'autorisation par agent empêchent les approbations d'un agent de fuiter vers d'autres.
- Les approbations ne s'appliquent qu'aux requêtes exec hôte des **expéditeurs autorisés**. Les expéditeurs non autorisés ne peuvent pas émettre `/exec`.
- `/exec security=full` est une commodité de niveau session pour les opérateurs autorisés et ignore les approbations par conception. Pour bloquer fermement l'exec hôte, définissez la sécurité des approbations à `deny` ou refusez l'outil `exec` via la politique d'outils.

Liens connexes :

- [Outil Exec](/tools/exec)
- [Mode Elevated](/tools/elevated)
- [Skills](/tools/skills)
