---
summary: "Installer OpenClaw — script d'installation, npm/pnpm, depuis les sources, Docker, et plus"
read_when:
  - Vous avez besoin d'une méthode d'installation autre que le démarrage rapide des Premiers pas
  - Vous souhaitez déployer sur une plateforme cloud
  - Vous devez mettre à jour, migrer ou désinstaller
title: "Installation"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/index.md
  workflow: manual
---

# Installation

Vous avez déjà suivi les [Premiers pas](/start/getting-started) ? Tout est prêt. Cette page concerne les méthodes d'installation alternatives, les instructions spécifiques aux plateformes et la maintenance.

## Configuration requise

- **[Node 22+](/install/node)** (le [script d'installation](#méthodes-dinstallation) l'installera s'il est absent)
- macOS, Linux ou Windows
- `pnpm` uniquement si vous compilez depuis les sources

<Note>
Sous Windows, nous recommandons fortement d'exécuter OpenClaw sous [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).
</Note>

## Méthodes d'installation

<Tip>
Le **script d'installation** est la méthode recommandée pour installer OpenClaw. Il gère la détection de Node, l'installation et la configuration initiale en une seule étape.
</Tip>

<Warning>
Pour les hébergeurs VPS/cloud, évitez les images de marketplace tierces « 1-clic » dans la mesure du possible. Préférez une image OS de base propre (par exemple Ubuntu LTS), puis installez OpenClaw vous-même avec le script d'installation.
</Warning>

<AccordionGroup>
  <Accordion title="Script d'installation" icon="rocket" defaultOpen>
    Télécharge le CLI, l'installe globalement via npm et lance l'assistant de configuration initiale.

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    C'est tout — le script gère la détection de Node, l'installation et la configuration initiale.

    Pour ignorer la configuration initiale et installer uniquement le binaire :

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
        ```
      </Tab>
    </Tabs>

    Pour tous les drapeaux, variables d'environnement et options CI/automatisation, consultez [Détails du script d'installation](/install/installer).

  </Accordion>

  <Accordion title="npm / pnpm" icon="package">
    Si vous avez déjà Node 22+ et préférez gérer l'installation vous-même :

    <Tabs>
      <Tab title="npm">
        ```bash
        npm install -g openclaw@latest
        openclaw onboard --install-daemon
        ```

        <Accordion title="Erreurs de compilation de sharp ?">
          Si libvips est installé globalement (courant sur macOS via Homebrew) et que `sharp` échoue, forcez les binaires précompilés :

          ```bash
          SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
          ```

          Si vous voyez `sharp: Please add node-gyp to your dependencies`, installez les outils de compilation (macOS : Xcode CLT + `npm install -g node-gyp`) ou utilisez la variable d'environnement ci-dessus.
        </Accordion>
      </Tab>
      <Tab title="pnpm">
        ```bash
        pnpm add -g openclaw@latest
        pnpm approve-builds -g        # approuver openclaw, node-llama-cpp, sharp, etc.
        openclaw onboard --install-daemon
        ```

        <Note>
        pnpm nécessite une approbation explicite pour les paquets avec des scripts de compilation. Après la première installation qui affiche l'avertissement « Ignored build scripts », exécutez `pnpm approve-builds -g` et sélectionnez les paquets listés.
        </Note>
      </Tab>
    </Tabs>

  </Accordion>

  <Accordion title="Depuis les sources" icon="github">
    Pour les contributeurs ou toute personne souhaitant exécuter depuis un checkout local.

    <Steps>
      <Step title="Cloner et compiler">
        Clonez le [dépôt OpenClaw](https://github.com/openclaw/openclaw) et compilez :

        ```bash
        git clone https://github.com/openclaw/openclaw.git
        cd openclaw
        pnpm install
        pnpm ui:build
        pnpm build
        ```
      </Step>
      <Step title="Lier le CLI">
        Rendez la commande `openclaw` disponible globalement :

        ```bash
        pnpm link --global
        ```

        Alternativement, ignorez le lien et exécutez les commandes via `pnpm openclaw ...` depuis le dépôt.
      </Step>
      <Step title="Lancer la configuration initiale">
        ```bash
        openclaw onboard --install-daemon
        ```
      </Step>
    </Steps>

    Pour des workflows de développement plus approfondis, consultez [Configuration](/start/setup).

  </Accordion>
</AccordionGroup>

## Autres méthodes d'installation

<CardGroup cols={2}>
  <Card title="Docker" href="/install/docker" icon="container">
    Déploiements conteneurisés ou headless.
  </Card>
  <Card title="Podman" href="/install/podman" icon="container">
    Conteneur rootless : exécutez `setup-podman.sh` une fois, puis le script de lancement.
  </Card>
  <Card title="Nix" href="/install/nix" icon="snowflake">
    Installation déclarative via Nix.
  </Card>
  <Card title="Ansible" href="/install/ansible" icon="server">
    Provisionnement automatisé de flottes.
  </Card>
  <Card title="Bun" href="/install/bun" icon="zap">
    Utilisation CLI uniquement via le runtime Bun.
  </Card>
</CardGroup>

## Après l'installation

Vérifiez que tout fonctionne :

```bash
openclaw doctor         # vérifier les problèmes de configuration
openclaw status         # statut du gateway
openclaw dashboard      # ouvrir l'interface navigateur
```

Si vous avez besoin de chemins d'exécution personnalisés, utilisez :

- `OPENCLAW_HOME` pour les chemins internes basés sur le répertoire personnel
- `OPENCLAW_STATE_DIR` pour l'emplacement de l'état mutable
- `OPENCLAW_CONFIG_PATH` pour l'emplacement du fichier de configuration

Consultez [Variables d'environnement](/help/environment) pour la priorité et tous les détails.

## Dépannage : `openclaw` introuvable

<Accordion title="Diagnostic et correction du PATH">
  Diagnostic rapide :

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) ou `$(npm prefix -g)` (Windows) **n'est pas** dans votre `$PATH`, votre shell ne peut pas trouver les binaires npm globaux (y compris `openclaw`).

Correction — ajoutez-le à votre fichier de démarrage du shell (`~/.zshrc` ou `~/.bashrc`) :

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Sous Windows, ajoutez la sortie de `npm prefix -g` à votre PATH.

Puis ouvrez un nouveau terminal (ou `rehash` dans zsh / `hash -r` dans bash).
</Accordion>

## Mise à jour / désinstallation

<CardGroup cols={3}>
  <Card title="Mise à jour" href="/install/updating" icon="refresh-cw">
    Gardez OpenClaw à jour.
  </Card>
  <Card title="Migration" href="/install/migrating" icon="arrow-right">
    Déplacez vers une nouvelle machine.
  </Card>
  <Card title="Désinstallation" href="/install/uninstall" icon="trash-2">
    Supprimez complètement OpenClaw.
  </Card>
</CardGroup>
