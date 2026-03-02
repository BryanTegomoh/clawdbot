---
title: "Node.js"
summary: "Installer et configurer Node.js pour OpenClaw : exigences de version, options d'installation et dépannage du PATH"
read_when:
  - "Vous devez installer Node.js avant d'installer OpenClaw"
  - "Vous avez installe OpenClaw mais `openclaw` donne command not found"
  - "npm install -g échoue avec des problèmes de permissions ou de PATH"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/node.md
  workflow: manual
---

# Node.js

OpenClaw nécessite **Node 22 ou plus recent**. Le [script d'installation](/install#install-methods) détectera et installera Node automatiquement : cette page est pour quand vous souhaitez configurer Node vous-même et vous assurer que tout est correctement branche (versions, PATH, installations globales).

## Vérifier votre version

```bash
node -v
```

Si cela affiche `v22.x.x` ou supérieur, c'est bon. Si Node n'est pas installe ou si la version est trop ancienne, choisissez une méthode d'installation ci-dessous.

## Installer Node

<Tabs>
  <Tab title="macOS">
    **Homebrew** (recommandé) :

    ```bash
    brew install node
    ```

    Ou télécharger l'installeur macOS depuis [nodejs.org](https://nodejs.org/).

  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian :**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL :**

    ```bash
    sudo dnf install nodejs
    ```

    Ou utiliser un gestionnaire de versions (voir ci-dessous).

  </Tab>
  <Tab title="Windows">
    **winget** (recommandé) :

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey :**

    ```powershell
    choco install nodejs-lts
    ```

    Ou télécharger l'installeur Windows depuis [nodejs.org](https://nodejs.org/).

  </Tab>
</Tabs>

<Accordion title="Utiliser un gestionnaire de versions (nvm, fnm, mise, asdf)">
  Les gestionnaires de versions vous permettent de passer facilement d'une version de Node a l'autre. Options populaires :

- [**fnm**](https://github.com/Schniz/fnm) : rapide, multi-plateforme
- [**nvm**](https://github.com/nvm-sh/nvm) : largement utilisé sur macOS/Linux
- [**mise**](https://mise.jdx.dev/) : polyglotte (Node, Python, Ruby, etc.)

Exemple avec fnm :

```bash
fnm install 22
fnm use 22
```

  <Warning>
  Assurez-vous que votre gestionnaire de versions est initialisé dans votre fichier de démarrage shell (`~/.zshrc` ou `~/.bashrc`). S'il ne l'est pas, `openclaw` pourrait ne pas être trouve dans les nouvelles sessions terminal car le PATH n'inclurait pas le répertoire bin de Node.
  </Warning>
</Accordion>

## Dépannage

### `openclaw: command not found`

Cela signifie presque toujours que le répertoire global bin de npm n'est pas dans votre PATH.

<Steps>
  <Step title="Trouver votre préfixe npm global">
    ```bash
    npm prefix -g
    ```
  </Step>
  <Step title="Vérifier s'il est dans votre PATH">
    ```bash
    echo "$PATH"
    ```

    Cherchez `<npm-prefix>/bin` (macOS/Linux) ou `<npm-prefix>` (Windows) dans la sortie.

  </Step>
  <Step title="L'ajouter a votre fichier de démarrage shell">
    <Tabs>
      <Tab title="macOS / Linux">
        Ajoutez a `~/.zshrc` ou `~/.bashrc` :

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        Puis ouvrez un nouveau terminal (ou exécutez `rehash` dans zsh / `hash -r` dans bash).
      </Tab>
      <Tab title="Windows">
        Ajoutez la sortie de `npm prefix -g` a votre PATH système via Paramètres, Système, Variables d'environnement.
      </Tab>
    </Tabs>

  </Step>
</Steps>

### Erreurs de permission sur `npm install -g` (Linux)

Si vous voyez des erreurs `EACCES`, changez le préfixe global de npm vers un répertoire accessible en écriture par l'utilisateur :

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Ajoutez la ligne `export PATH=...` a votre `~/.bashrc` ou `~/.zshrc` pour la rendre permanente.
