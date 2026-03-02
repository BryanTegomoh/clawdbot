---
summary: "Exécuter OpenClaw dans un conteneur Podman sans privilèges root"
read_when:
  - Vous souhaitez un Gateway conteneurisé avec Podman au lieu de Docker
title: "Podman"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/install/podman.md
  workflow: manual
---

# Podman

Exécutez le Gateway OpenClaw dans un conteneur Podman **sans privilèges root** (rootless). Utilisé la même image que Docker (construite à partir du [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) du dépôt).

## Prérequis

- Podman (rootless)
- Sudo pour la configuration initiale (création utilisateur, construction de l'image)

## Démarrage rapide

**1. Configuration initiale** (depuis la racine du dépôt ; crée l'utilisateur, construit l'image, installe le script de lancement) :

```bash
./setup-podman.sh
```

Cela crée également un fichier minimal `~openclaw/.openclaw/openclaw.json` (définit `gateway.mode="local"`) pour que le Gateway puisse démarrer sans exécuter l'assistant de configuration.

Par défaut, le conteneur n'est **pas** installé en tant que service systemd ; vous le démarrez manuellement (voir ci-dessous). Pour une configuration de type production avec démarrage automatique et redémarrages, installez-le plutôt en tant que service utilisateur systemd Quadlet :

```bash
./setup-podman.sh --quadlet
```

(Ou définissez `OPENCLAW_PODMAN_QUADLET=1` ; utilisez `--container` pour installer uniquement le conteneur et le script de lancement.)

**2. Démarrer le Gateway** (manuellement, pour un test rapide) :

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Assistant de configuration** (par ex. pour ajouter des canaux ou des fournisseurs) :

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Puis ouvrez `http://127.0.0.1:18789/` et utilisez le jeton depuis `~openclaw/.openclaw/.env` (ou la valeur affichée par le script de configuration).

## Systemd (Quadlet, optionnel)

Si vous avez exécuté `./setup-podman.sh --quadlet` (ou `OPENCLAW_PODMAN_QUADLET=1`), une unité [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) est installée pour que le Gateway s'exécute en tant que service utilisateur systemd pour l'utilisateur openclaw. Le service est activé et démarré à la fin de la configuration.

- **Démarrer :** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Arrêter :** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Statut :** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Journaux :** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Le fichier quadlet se trouve dans `~openclaw/.config/containers/systemd/openclaw.container`. Pour modifier les ports ou les variables d'environnement, éditez ce fichier (ou le `.env` qu'il source), puis exécutez `sudo systemctl --machine openclaw@ --user daemon-reload` et redémarrez le service. Au démarrage du système, le service démarre automatiquement si le lingering est activé pour openclaw (la configuration le fait quand loginctl est disponible).

Pour ajouter le quadlet **après** une configuration initiale qui ne l'utilisait pas, relancez : `./setup-podman.sh --quadlet`.

## L'utilisateur openclaw (sans connexion interactive)

`setup-podman.sh` crée un utilisateur système dédié `openclaw` :

- **Shell :** `nologin` : pas de connexion interactive ; réduit la surface d'attaque.
- **Répertoire personnel :** par ex. `/home/openclaw` : contient `~/.openclaw` (configuration, espace de travail) et le script de lancement `run-openclaw-podman.sh`.
- **Podman rootless :** L'utilisateur doit avoir une plage **subuid** et **subgid**. De nombreuses distributions les assignent automatiquement lors de la création de l'utilisateur. Si la configuration affiche un avertissement, ajoutez des lignes dans `/etc/subuid` et `/etc/subgid` :

  ```text
  openclaw:100000:65536
  ```

  Puis démarrez le Gateway avec cet utilisateur (par ex. depuis cron ou systemd) :

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Configuration :** Seuls `openclaw` et root peuvent accéder à `/home/openclaw/.openclaw`. Pour modifier la configuration : utilisez l'interface de contrôle une fois le Gateway en cours d'exécution, ou `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Environnement et configuration

- **Jeton :** Stocké dans `~openclaw/.openclaw/.env` sous `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` et `run-openclaw-podman.sh` le génèrent s'il est absent (utilisé `openssl`, `python3`, ou `od`).
- **Optionnel :** Dans ce `.env`, vous pouvez définir les clés des fournisseurs (par ex. `GROQ_API_KEY`, `OLLAMA_API_KEY`) et d'autres variables d'environnement OpenClaw.
- **Ports hôte :** Par défaut, le script mappe `18789` (Gateway) et `18790` (bridge). Modifiez le mappage des ports **hôte** avec `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` et `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` lors du lancement.
- **Liaison du Gateway :** Par défaut, `run-openclaw-podman.sh` démarre le Gateway avec `--bind loopback` pour un accès local sécurisé. Pour exposer sur le réseau local, définissez `OPENCLAW_GATEWAY_BIND=lan` et configurez `gateway.controlUi.allowedOrigins` (ou activez explicitement le fallback host-header) dans `openclaw.json`.
- **Chemins :** La configuration et l'espace de travail hôte se trouvent par défaut dans `~openclaw/.openclaw` et `~openclaw/.openclaw/workspace`. Modifiez les chemins hôte utilisés par le script de lancement avec `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR`.

## Commandes utiles

- **Journaux :** Avec quadlet : `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Avec le script : `sudo -u openclaw podman logs -f openclaw`
- **Arrêter :** Avec quadlet : `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Avec le script : `sudo -u openclaw podman stop openclaw`
- **Redémarrer :** Avec quadlet : `sudo systemctl --machine openclaw@ --user start openclaw.service`. Avec le script : relancez le script de lancement ou `podman start openclaw`
- **Supprimer le conteneur :** `sudo -u openclaw podman rm -f openclaw` : la configuration et l'espace de travail sur l'hôte sont conservés

## Dépannage

- **Permission refusée (EACCES) sur la configuration ou auth-profiles :** Le conteneur utilisé par défaut `--userns=keep-id` et s'exécute avec le même uid/gid que l'utilisateur hôte exécutant le script. Assurez-vous que vos `OPENCLAW_CONFIG_DIR` et `OPENCLAW_WORKSPACE_DIR` hôte appartiennent à cet utilisateur.
- **Démarrage du Gateway bloqué (absence de `gateway.mode=local`) :** Assurez-vous que `~openclaw/.openclaw/openclaw.json` existe et définit `gateway.mode="local"`. `setup-podman.sh` crée ce fichier s'il est absent.
- **Podman rootless échoue pour l'utilisateur openclaw :** Vérifiez que `/etc/subuid` et `/etc/subgid` contiennent une ligne pour `openclaw` (par ex. `openclaw:100000:65536`). Ajoutez-la si elle manque et redémarrez.
- **Nom de conteneur déjà utilisé :** Le script de lancement utilisé `podman run --replace`, donc le conteneur existant est remplacé quand vous redémarrez. Pour nettoyer manuellement : `podman rm -f openclaw`.
- **Script introuvable lors de l'exécution en tant qu'openclaw :** Assurez-vous que `setup-podman.sh` a été exécuté pour que `run-openclaw-podman.sh` soit copié dans le répertoire personnel d'openclaw (par ex. `/home/openclaw/run-openclaw-podman.sh`).
- **Service Quadlet introuvable ou échec au démarrage :** Exécutez `sudo systemctl --machine openclaw@ --user daemon-reload` après avoir modifié le fichier `.container`. Quadlet nécessite cgroups v2 : `podman info --format '{{.Host.CgroupsVersion}}'` devrait afficher `2`.

## Optionnel : exécuter en tant que votre propre utilisateur

Pour exécuter le Gateway en tant que votre utilisateur normal (sans utilisateur openclaw dédié) : construisez l'image, créez `~/.openclaw/.env` avec `OPENCLAW_GATEWAY_TOKEN`, et lancez le conteneur avec `--userns=keep-id` et des montages vers votre `~/.openclaw`. Le script de lancement est conçu pour le flux utilisateur openclaw ; pour une configuration mono-utilisateur, vous pouvez exécuter manuellement la commande `podman run` du script en pointant la configuration et l'espace de travail vers votre répertoire personnel. Recommandé pour la plupart des utilisateurs : utilisez `setup-podman.sh` et exécutez en tant qu'utilisateur openclaw afin que la configuration et le processus soient isolés.
