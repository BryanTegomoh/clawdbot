---
summary: "Mettre à jour OpenClaw en toute sécurité (installation globale ou source), plus stratégie de rollback"
read_when:
  - Mise à jour d'OpenClaw
  - Quelque chose ne fonctionne plus après une mise à jour
title: "Mise à jour"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/install/updating.md
  workflow: manual
---

# Mise à jour

OpenClaw évolue rapidement (pré « 1.0 »). Traitez les mises à jour comme de l'infrastructure de production : mise à jour -> vérifications -> redémarrage (ou utilisez `openclaw update`, qui redémarre) -> validation.

## Recommandé : relancer l'installateur du site web (mise à jour en place)

La méthode de mise à jour **préférée** est de relancer l'installateur depuis le site web. Il
détecte les installations existantes, met à jour en place et exécute `openclaw doctor` si
nécessaire.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Notes :

- Ajoutez `--no-onboard` si vous ne voulez pas que l'assistant d'intégration se relance.
- Pour les **installations depuis les sources**, utilisez :

  ```bash
  curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
  ```

  L'installateur exécutera `git pull --rebase` **uniquement** si le dépôt est propre.

- Pour les **installations globales**, le script utilise `npm install -g openclaw@latest` en interne.
- Note historique : `clawdbot` reste disponible en tant que shim de compatibilité.

## Avant de mettre à jour

- Sachez comment vous avez installé : **global** (npm/pnpm) vs **depuis les sources** (git clone).
- Sachez comment votre Gateway fonctionne : **terminal au premier plan** vs **service supervisé** (launchd/systemd).
- Sauvegardez votre personnalisation :
  - Configuration : `~/.openclaw/openclaw.json`
  - Identifiants : `~/.openclaw/credentials/`
  - Espace de travail : `~/.openclaw/workspace`

## Mise à jour (installation globale)

Installation globale (choisissez une option) :

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

Nous ne recommandons **pas** Bun pour l'exécution du Gateway (bogues WhatsApp/Telegram).

Pour changer de canal de mise à jour (installations git + npm) :

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

Utilisez `--tag <dist-tag|version>` pour un tag/version d'installation ponctuelle.

Voir [Canaux de développement](/install/development-channels) pour la sémantique des canaux et les notes de version.

Note : sur les installations npm, le Gateway affiche un indice de mise à jour au démarrage (vérifie le tag du canal actuel). Désactivez via `update.checkOnStart: false`.

### Mise à jour automatique du cœur (optionnel)

La mise à jour automatique est **désactivée par défaut** et constitue une fonctionnalité du cœur du Gateway (pas un plugin).

```json
{
  "update": {
    "channel": "stable",
    "auto": {
      "enabled": true,
      "stableDelayHours": 6,
      "stableJitterHours": 12,
      "betaCheckIntervalHours": 1
    }
  }
}
```

Comportement :

- `stable` : quand une nouvelle version est détectée, OpenClaw attend `stableDelayHours` puis applique un jitter déterministe par installation dans `stableJitterHours` (déploiement progressif).
- `beta` : vérifie selon la cadence `betaCheckIntervalHours` (par défaut : toutes les heures) et applique quand une mise à jour est disponible.
- `dev` : pas d'application automatique ; utilisez `openclaw update` manuellement.

Utilisez `openclaw update --dry-run` pour prévisualiser les actions de mise à jour avant d'activer l'automatisation.

Puis :

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

Notes :

- Si votre Gateway fonctionne en tant que service, `openclaw gateway restart` est préférable à la suppression des PID.
- Si vous êtes épinglé à une version spécifique, voir « Rollback / épinglage » ci-dessous.

## Mise à jour (`openclaw update`)

Pour les **installations depuis les sources** (git checkout), préférez :

```bash
openclaw update
```

Il exécute un flux de mise à jour raisonnablement sûr :

- Nécessite un répertoire de travail propre.
- Bascule vers le canal sélectionné (tag ou branche).
- Fetch + rebase sur l'upstream configuré (canal dev).
- Installe les dépendances, compile, construit l'interface de contrôle et exécute `openclaw doctor`.
- Redémarre le Gateway par défaut (utilisez `--no-restart` pour ignorer).

Si vous avez installé via **npm/pnpm** (pas de métadonnées git), `openclaw update` essaiera de mettre à jour via votre gestionnaire de paquets. S'il ne peut pas détecter l'installation, utilisez plutôt « Mise à jour (installation globale) ».

## Mise à jour (interface de contrôle / RPC)

L'interface de contrôle propose **Mettre à jour et redémarrer** (RPC : `update.run`). Elle :

1. Exécute le même flux de mise à jour source que `openclaw update` (git checkout uniquement).
2. Écrit une sentinelle de redémarrage avec un rapport structuré (fin de stdout/stderr).
3. Redémarre le Gateway et notifie la dernière session active avec le rapport.

Si le rebase échoue, le Gateway interrompt et redémarre sans appliquer la mise à jour.

## Mise à jour (depuis les sources)

Depuis le checkout du dépôt :

Méthode préférée :

```bash
openclaw update
```

Méthode manuelle (équivalente) :

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # installe automatiquement les dépendances UI au premier lancement
openclaw doctor
openclaw health
```

Notes :

- `pnpm build` est important quand vous exécutez le binaire packagé `openclaw` ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) ou utilisez Node pour exécuter `dist/`.
- Si vous exécutez depuis un checkout de dépôt sans installation globale, utilisez `pnpm openclaw ...` pour les commandes CLI.
- Si vous exécutez directement depuis TypeScript (`pnpm openclaw ...`), une recompilation est généralement inutile, mais **les migrations de configuration s'appliquent toujours** -> exécutez doctor.
- Basculer entre les installations globales et git est facile : installez l'autre variante, puis exécutez `openclaw doctor` pour que le point d'entrée du service Gateway soit réécrit vers l'installation courante.

## Toujours exécuter : `openclaw doctor`

Doctor est la commande de « mise à jour sûre ». Elle est volontairement simple : réparation + migration + avertissements.

Note : si vous êtes sur une **installation depuis les sources** (git checkout), `openclaw doctor` proposera d'exécuter `openclaw update` d'abord.

Actions typiques :

- Migration des clés de configuration obsolètes / emplacements de fichiers de configuration hérités.
- Audit des politiques DM et avertissement sur les paramètres « ouverts » risqués.
- Vérification de la santé du Gateway avec possibilité de proposer un redémarrage.
- Détection et migration des anciens services Gateway (launchd/systemd ; schtasks hérités) vers les services OpenClaw actuels.
- Sous Linux, s'assuré du lingering utilisateur systemd (pour que le Gateway survive à la déconnexion).

Détails : [Doctor](/gateway/doctor)

## Démarrer / arrêter / redémarrer le Gateway

CLI (fonctionne quel que soit le système d'exploitation) :

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

Si vous êtes supervisé :

- macOS launchd (LaunchAgent intégré à l'application) : `launchctl kickstart -k gui/$UID/ai.openclaw.gateway` (utilisez `ai.openclaw.<profile>` ; l'ancien `com.openclaw.*` fonctionne toujours)
- Linux service utilisateur systemd : `systemctl --user restart openclaw-gateway[-<profile>].service`
- Windows (WSL2) : `systemctl --user restart openclaw-gateway[-<profile>].service`
  - `launchctl`/`systemctl` ne fonctionnent que si le service est installé ; sinon exécutez `openclaw gateway install`.

Runbook + libellés exacts des services : [Runbook du Gateway](/gateway)

## Rollback / épinglage (quand quelque chose casse)

### Épinglage (installation globale)

Installez une version connue comme fonctionnelle (remplacez `<version>` par la dernière version stable) :

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

Astuce : pour voir la version actuellement publiée, exécutez `npm view openclaw version`.

Puis redémarrez + relancez doctor :

```bash
openclaw doctor
openclaw gateway restart
```

### Épinglage (source) par date

Choisissez un commit à partir d'une date (exemple : « état de main au 2026-01-01 ») :

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

Puis réinstallez les dépendances + redémarrez :

```bash
pnpm install
pnpm build
openclaw gateway restart
```

Si vous voulez revenir à la dernière version plus tard :

```bash
git checkout main
git pull
```

## Si vous êtes bloqué

- Exécutez `openclaw doctor` à nouveau et lisez attentivement la sortie (elle indique souvent la solution).
- Consultez : [Dépannage](/gateway/troubleshooting)
- Demandez sur Discord : [https://discord.gg/clawd](https://discord.gg/clawd)
