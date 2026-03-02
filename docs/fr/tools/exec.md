---
summary: "Utilisation de l'outil exec, modes stdin et support TTY"
read_when:
  - Utilisation ou modification de l'outil exec
  - Débogage du comportement stdin ou TTY
title: "Outil Exec"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: tools/exec.md
  workflow: manual
---

# Outil exec

Exécutez des commandes shell dans l'espace de travail. Supporte l'exécution en avant-plan + arrière-plan via `process`.
Si `process` est interdit, `exec` s'exécute de manière synchrone et ignore `yieldMs`/`background`.
Les sessions en arrière-plan sont limitées par agent ; `process` ne voit que les sessions du même agent.

## Paramètres

- `command` (obligatoire)
- `workdir` (défaut : cwd)
- `env` (substitutions clé/valeur)
- `yieldMs` (défaut 10000) : passage en arrière-plan automatique après ce délai
- `background` (bool) : passage en arrière-plan immédiat
- `timeout` (secondes, défaut 1800) : arrêt à expiration
- `pty` (bool) : exécuter dans un pseudo-terminal quand disponible (CLI TTY uniquement, agents de codage, interfaces terminal)
- `host` (`sandbox | gateway | node`) : lieu d'exécution
- `security` (`deny | allowlist | full`) : mode d'application pour `gateway`/`node`
- `ask` (`off | on-miss | always`) : invites d'approbation pour `gateway`/`node`
- `node` (chaîne) : id/nom du node pour `host=node`
- `elevated` (bool) : demander le mode élevé (hôte gateway) ; `security=full` n'est forcé que quand elevated résout à `full`

Notes :

- `host` est par défaut `sandbox`.
- `elevated` est ignoré quand le sandboxing est désactivé (exec s'exécute déjà sur l'hôte).
- Les approbations `gateway`/`node` sont contrôlées par `~/.openclaw/exec-approvals.json`.
- `node` nécessite un node apparié (application compagnon ou node host headless).
- Si plusieurs nodes sont disponibles, définissez `exec.node` ou `tools.exec.node` pour en sélectionner un.
- Sur les hôtes non-Windows, exec utilisé `SHELL` quand défini ; si `SHELL` est `fish`, il préfère `bash` (ou `sh`)
  depuis `PATH` pour éviter les scripts incompatibles avec fish, puis revient à `SHELL` si aucun n'existe.
- Sur les hôtes Windows, exec préfère la découverte PowerShell 7 (`pwsh`) (Program Files, ProgramW6432, puis PATH),
  puis revient à Windows PowerShell 5.1.
- L'exécution hôte (`gateway`/`node`) rejette `env.PATH` et les substitutions de chargeur (`LD_*`/`DYLD_*`) pour
  empêcher le détournement de binaires ou l'injection de code.
- Important : le sandboxing est **désactivé par défaut**. Si le sandboxing est désactivé et que `host=sandbox` est explicitement
  configuré/demandé, exec échoue de manière sûre au lieu de s'exécuter silencieusement sur l'hôte gateway.
  Activez le sandboxing ou utilisez `host=gateway` avec des approbations.
- Les vérifications préalables de scripts (pour les erreurs courantes de syntaxe shell Python/Node) n'inspectent que les fichiers à l'intérieur de
  la limité `workdir` effective. Si un chemin de script résout en dehors de `workdir`, la vérification préalable est ignorée pour
  ce fichier.

## Configuration

- `tools.exec.notifyOnExit` (défaut : true) : quand activé, les sessions exec en arrière-plan enfilent un événement système et demandent un heartbeat à la sortie.
- `tools.exec.approvalRunningNoticeMs` (défaut : 10000) : émet un seul avis « en cours » quand une exec à approbation dure plus longtemps (0 désactive).
- `tools.exec.host` (défaut : `sandbox`)
- `tools.exec.security` (défaut : `deny` pour sandbox, `allowlist` pour gateway + node quand non défini)
- `tools.exec.ask` (défaut : `on-miss`)
- `tools.exec.node` (défaut : non défini)
- `tools.exec.pathPrepend` : liste de répertoires à ajouter en tête de `PATH` pour les exécutions exec (gateway + sandbox uniquement).
- `tools.exec.safeBins` : binaires sûrs stdin uniquement pouvant s'exécuter sans entrées explicites de liste d'autorisation. Pour les détails de comportement, voir [Safe bins](/tools/exec-approvals#safe-bins-stdin-only).
- `tools.exec.safeBinTrustedDirs` : répertoires supplémentaires explicitement approuvés pour les vérifications de chemin `safeBins`. Les entrées `PATH` ne sont jamais auto-approuvées. Les défauts intégrés sont `/bin` et `/usr/bin`.
- `tools.exec.safeBinProfiles` : politique argv personnalisée optionnelle par safe bin (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

Exemple :

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### Gestion du PATH

- `host=gateway` : fusionne le `PATH` de votre shell de connexion dans l'environnement exec. Les substitutions `env.PATH` sont
  rejetées pour l'exécution hôte. Le démon lui-même fonctionne avec un `PATH` minimal :
  - macOS : `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  - Linux : `/usr/local/bin`, `/usr/bin`, `/bin`
- `host=sandbox` : exécute `sh -lc` (shell de connexion) à l'intérieur du conteneur, donc `/etc/profile` peut réinitialiser `PATH`.
  OpenClaw ajoute `env.PATH` en tête après le sourcing du profil via une variable d'environnement interne (pas d'interpolation shell) ;
  `tools.exec.pathPrepend` s'applique ici aussi.
- `host=node` : seules les substitutions d'env non bloquées que vous passez sont envoyées au node. Les substitutions `env.PATH` sont
  rejetées pour l'exécution hôte et ignorées par les node hosts. Si vous avez besoin d'entrées PATH supplémentaires sur un node,
  configurez l'environnement du service node host (systemd/launchd) ou installez les outils dans des emplacements standards.

Liaison de node par agent (utilisez l'index de la liste des agents dans la configuration) :

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Interface de contrôle : l'onglet Nodes inclut un petit panneau « Liaison exec node » pour les mêmes paramètres.

## Substitutions de session (`/exec`)

Utilisez `/exec` pour définir des défauts **par session** pour `host`, `security`, `ask` et `node`.
Envoyez `/exec` sans arguments pour afficher les valeurs actuelles.

Exemple :

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## Modèle d'autorisation

`/exec` n'est honoré que pour les **expéditeurs autorisés** (listes d'autorisation de canaux/appariement plus `commands.useAccessGroups`).
Il met à jour **uniquement l'état de session** et n'écrit pas la configuration. Pour désactiver complètement exec, refusez-le via la politique d'outils (`tools.deny: ["exec"]` ou par agent). Les approbations hôte s'appliquent toujours sauf si vous définissez explicitement `security=full` et `ask=off`.

## Approbations exec (application compagnon / node host)

Les agents sandboxés peuvent exiger une approbation par requête avant que `exec` ne s'exécute sur l'hôte gateway ou node.
Voir [Approbations exec](/tools/exec-approvals) pour la politique, la liste d'autorisation et le flux d'interface.

Quand des approbations sont requises, l'outil exec retourne immédiatement avec
`status: "approval-pending"` et un id d'approbation. Une fois approuvé (ou refusé / expiré),
le Gateway émet des événements système (`Exec finished` / `Exec denied`). Si la commande est toujours
en cours après `tools.exec.approvalRunningNoticeMs`, un seul avis `Exec running` est émis.

## Liste d'autorisation + safe bins

L'application de la liste d'autorisation manuelle correspond **uniquement aux chemins de binaires résolus** (pas de correspondance par nom de base). Quand
`security=allowlist`, les commandes shell sont auto-autorisées uniquement si chaque segment de pipeline est
dans la liste d'autorisation ou est un safe bin. Le chaînage (`;`, `&&`, `||`) et les redirections sont rejetés en
mode liste d'autorisation sauf si chaque segment de niveau supérieur satisfait la liste d'autorisation (y compris les safe bins).
Les redirections restent non supportées.

`autoAllowSkills` est un chemin de commodité séparé dans les approbations exec. Ce n'est pas la même chose que
les entrées manuelles de la liste d'autorisation de chemins. Pour une confiance explicite stricte, gardez `autoAllowSkills` désactivé.

Utilisez les deux contrôles pour des tâches différentes :

- `tools.exec.safeBins` : petits filtrés de flux stdin uniquement.
- `tools.exec.safeBinTrustedDirs` : répertoires supplémentaires explicitement approuvés pour les chemins d'exécutables safe-bin.
- `tools.exec.safeBinProfiles` : politique argv explicite pour les safe bins personnalisés.
- liste d'autorisation : confiance explicite pour les chemins d'exécutables.

Ne traitez pas `safeBins` comme une liste d'autorisation générique, et n'ajoutez pas de binaires interpréteurs/runtime (par exemple `python3`, `node`, `ruby`, `bash`). Si vous en avez besoin, utilisez des entrées explicites de liste d'autorisation et gardez les invites d'approbation activées.
`openclaw security audit` avertit quand des entrées `safeBins` d'interpréteurs/runtime n'ont pas de profils explicites, et `openclaw doctor --fix` peut créer les entrées `safeBinProfiles` personnalisées manquantes.

Pour tous les détails de politique et exemples, voir [Approbations exec](/tools/exec-approvals#safe-bins-stdin-only) et [Safe bins versus liste d'autorisation](/tools/exec-approvals#safe-bins-versus-allowlist).

## Exemples

Avant-plan :

```json
{ "tool": "exec", "command": "ls -la" }
```

Arrière-plan + poll :

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Envoi de touches (style tmux) :

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Soumettre (envoyer CR uniquement) :

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

Coller (avec délimiteurs par défaut) :

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply_patch (expérimental)

`apply_patch` est un sous-outil de `exec` pour les éditions multi-fichiers structurées.
Activez-le explicitement :

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

Notes :

- Disponible uniquement pour les modèles OpenAI/OpenAI Codex.
- La politique d'outils s'applique toujours ; `allow: ["exec"]` autorisé implicitement `apply_patch`.
- La configuration se trouve sous `tools.exec.applyPatch`.
- `tools.exec.applyPatch.workspaceOnly` est par défaut `true` (contenu dans l'espace de travail). Définissez-le à `false` uniquement si vous souhaitez intentionnellement que `apply_patch` écrive/supprimé en dehors du répertoire de l'espace de travail.
