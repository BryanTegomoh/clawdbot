---
summary: "Application compagnon OpenClaw macOS (barre de menus + broker gateway)"
read_when:
  - Implementation des fonctionnalités de l'application macOS
  - Modification du cycle de vie du gateway ou du pontage de nœuds sur macOS
title: "Application macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/macos.md
  workflow: manual
---

# Application compagnon OpenClaw macOS (barre de menus + broker gateway)

L'application macOS est le **compagnon barre de menus** pour OpenClaw. Elle gère les permissions,
administre/se connecte au Gateway localement (launchd ou manuel) et expose les
capacités macOS a l'agent en tant que nœud.

## Ce qu'elle fait

- Affiche les notifications natives et le statut dans la barre de menus.
- Géré les invites TCC (Notifications, Accessibilité, Enregistrement d'ecran, Microphone,
  Reconnaissance vocale, Automatisation/AppleScript).
- Exécuté ou se connecte au Gateway (local ou distant).
- Expose les outils spécifiques a macOS (Canvas, Camera, Enregistrement d'ecran, `system.run`).
- Démarre le service hôte de nœud local en mode **distant** (launchd), et l'arrêté en mode **local**.
- Heberge optionnellement **PeekabooBridge** pour l'automatisation de l'interface utilisateur.
- Installe le CLI global (`openclaw`) via npm/pnpm sur demande (bun non recommandé pour le runtime du Gateway).

## Mode local vs distant

- **Local** (par défaut) : l'application se connecte a un Gateway local en cours d'exécution s'il est présent ;
  sinon elle active le service launchd via `openclaw gateway install`.
- **Distant** : l'application se connecte a un Gateway via SSH/Tailscale et ne démarre jamais
  un processus local.
  L'application démarre le **service hôte de nœud** local pour que le Gateway distant puisse atteindre ce Mac.
  L'application ne lance pas le Gateway en tant que processus enfant.

## Contrôle launchd

L'application gère un LaunchAgent par utilisateur labellise `ai.openclaw.gateway`
(ou `ai.openclaw.<profile>` lors de l'utilisation de `--profile`/`OPENCLAW_PROFILE` ; l'ancien `com.openclaw.*` est toujours decharge).

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Remplacez le label par `ai.openclaw.<profile>` lors de l'exécution d'un profil nomme.

Si le LaunchAgent n'est pas installe, activez-le depuis l'application ou exécutez
`openclaw gateway install`.

## Capacités du nœud (mac)

L'application macOS se présenté comme un nœud. Commandes courantes :

- Canvas : `canvas.présent`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
- Camera : `camera.snap`, `camera.clip`
- Ecran : `screen.record`
- Système : `system.run`, `system.notify`

Le nœud rapporte une carte `permissions` pour que les agents puissent decider de ce qui est autorisé.

Service de nœud + IPC de l'application :

- Lorsque le service hôte de nœud headless est en cours d'exécution (mode distant), il se connecte au WebSocket du Gateway en tant que nœud.
- `system.run` s'exécute dans l'application macOS (contexte UI/TCC) via un socket Unix local ; les invites + la sortie restent dans l'application.

Diagramme (SCI) :

```
Gateway -> Service de noeud (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Application Mac (UI + TCC + system.run)
```

## Approbations d'exécution (system.run)

`system.run` est contrôle par les **Approbations d'exécution** dans l'application macOS (Paramètres → Approbations d'exécution).
La sécurité + demande + liste blanche sont stockees localement sur le Mac dans :

```
~/.openclaw/exec-approvals.json
```

Exemple :

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

Notes :

- Les entrées `allowlist` sont des motifs glob pour les chemins de binaires resolus.
- Le texte de commande shell brut qui contient de la syntaxe de contrôle ou d'expansion shell (`&&`, `||`, `;`, `|`, `` ` ``, `$`, `<`, `>`, `(`, `)`) est traite comme un manquement a la liste blanche et nécessite une approbation explicite (ou l'ajout du binaire shell a la liste blanche).
- Choisir « Toujours autoriser » dans l'invite ajouté cette commande a la liste blanche.
- Les surcharges d'environnement de `system.run` sont filtrées (supprimé `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`) puis fusionnees avec l'environnement de l'application.
- Pour les wrappers shell (`bash|sh|zsh ... -c/-lc`), les surcharges d'environnement limitées a la requête sont réduites a une petite liste blanche explicite (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
- Pour les decisions « toujours autoriser » en mode liste blanche, les wrappers de dispatch connus (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) persistent les chemins de l'executable interne au lieu des chemins du wrapper. Si le deroulement n'est pas sur, aucune entrée de liste blanche n'est persistée automatiquement.

## Liens profonds

L'application enregistre le schema d'URL `openclaw://` pour les actions locales.

### `openclaw://agent`

Déclenché une requête Gateway `agent`.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Paramètres de requête :

- `message` (requis)
- `sessionKey` (optionnel)
- `thinking` (optionnel)
- `deliver` / `to` / `channel` (optionnel)
- `timeoutSeconds` (optionnel)
- `key` (clé optionnelle en mode sans surveillance)

Sécurité :

- Sans `key`, l'application demande confirmation.
- Sans `key`, l'application impose une limite de longueur de message courte pour l'invite de confirmation et ignore `deliver` / `to` / `channel`.
- Avec une `key` valide, l'exécution est sans surveillance (destinee aux automatisations personnelles).

## Flux d'intégration (typique)

1. Installez et lancez **OpenClaw.app**.
2. Completez la checklist des permissions (invites TCC).
3. Assurez-vous que le mode **Local** est actif et que le Gateway est en cours d'exécution.
4. Installez le CLI si vous souhaitez un accès terminal.

## Workflow de build et dev (natif)

- `cd apps/macos && swift build`
- `swift run OpenClaw` (ou Xcode)
- Empaquetage de l'application : `scripts/package-mac-app.sh`

## Debogage de la connectivité Gateway (CLI macOS)

Utilisez le CLI de debogage pour exercer la même poignee de main WebSocket du Gateway et la même logique de
decouverte que l'application macOS utilise, sans lancer l'application.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Options de connexion :

- `--url <ws://host:port>` : surcharger la configuration
- `--mode <local|remote>` : résoudre depuis la configuration (par défaut : config ou local)
- `--probe` : forcer une sonde de sante
- `--timeout <ms>` : délai de la requête (par défaut : `15000`)
- `--json` : sortie structuree pour le diff

Options de decouverte :

- `--include-local` : inclure les gateways qui seraient filtrés comme « locaux »
- `--timeout <ms>` : fenêtre globale de decouverte (par défaut : `2000`)
- `--json` : sortie structuree pour le diff

Astuce : comparez avec `openclaw gateway discover --json` pour voir si le
pipeline de decouverte de l'application macOS (NWBrowser + fallback DNS-SD tailnet) differe de
la decouverte `dns-sd` du CLI Node.

## Mecanisme de connexion distante (tunnels SSH)

Lorsque l'application macOS s'exécute en mode **Distant**, elle ouvre un tunnel SSH pour que les composants UI locaux
puissent communiquer avec un Gateway distant comme s'il etait sur localhost.

### Tunnel de contrôle (port WebSocket du Gateway)

- **Objectif :** vérifications de sante, statut, Web Chat, configuration et autres appels du plan de contrôle.
- **Port local :** le port du Gateway (par défaut `18789`), toujours stable.
- **Port distant :** le même port du Gateway sur l'hôte distant.
- **Comportement :** pas de port local aleatoire ; l'application reutilise un tunnel sain existant
  ou le redémarre si nécessaire.
- **Forme SSH :** `ssh -N -L <local>:127.0.0.1:<remote>` avec BatchMode +
  ExitOnForwardFailure + options keepalive.
- **Rapport d'IP :** le tunnel SSH utilisé le loopback, donc le gateway verra l'IP du nœud
  comme `127.0.0.1`. Utilisez le transport **Direct (ws/wss)** si vous voulez que l'IP reelle du client
  apparaisse (voir [Accès distant macOS](/platforms/mac/remote)).

Pour les étapes de configuration, voir [Accès distant macOS](/platforms/mac/remote). Pour les details
du protocole, voir [Protocole Gateway](/gateway/protocol).

## Documentation associée

- [Runbook Gateway](/gateway)
- [Gateway (macOS)](/platforms/mac/bundled-gateway)
- [Permissions macOS](/platforms/mac/permissions)
- [Canvas](/platforms/mac/canvas)
