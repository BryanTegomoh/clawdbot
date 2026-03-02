---
read_when:
  - Première installation depuis zéro
  - Vous cherchez le chemin le plus rapide vers un chat fonctionnel
summary: Installez OpenClaw et lancez votre premier chat en quelques minutes.
title: Premiers pas
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/getting-started.md
  workflow: manual
---

# Premiers pas

Objectif : passer de zéro à un premier chat fonctionnel avec un minimum de configuration.

<Info>
Chat le plus rapide : ouvrez l'interface de contrôle (aucune configuration de canal nécessaire). Exécutez `openclaw dashboard`
et discutez dans le navigateur, ou ouvrez `http://127.0.0.1:18789/` sur
l'<Tooltip headline="Hôte du Gateway" tip="La machine exécutant le service gateway OpenClaw.">hôte du Gateway</Tooltip>.
Documentation : [Tableau de bord](/web/dashboard) et [Interface de contrôle](/web/control-ui).
</Info>

## Prérequis

- Node 22 ou plus récent

<Tip>
Vérifiez votre version de Node avec `node --version` en cas de doute.
</Tip>

## Configuration rapide (CLI)

<Steps>
  <Step title="Installer OpenClaw (recommandé)">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
        <img
  src="/assets/install-script.svg"
  alt="Processus du script d'installation"
  className="rounded-lg"
/>
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    <Note>
    Autres méthodes d'installation et prérequis : [Installation](/install).
    </Note>

  </Step>
  <Step title="Lancer l'assistant de configuration initiale">
    ```bash
    openclaw onboard --install-daemon
    ```

    L'assistant configure l'authentification, les paramètres du gateway et les canaux optionnels.
    Voir [Assistant de configuration initiale](/start/wizard) pour les détails.

  </Step>
  <Step title="Vérifier le Gateway">
    Si vous avez installé le service, il devrait déjà fonctionner :

    ```bash
    openclaw gateway status
    ```

  </Step>
  <Step title="Ouvrir l'interface de contrôle">
    ```bash
    openclaw dashboard
    ```
  </Step>
</Steps>

<Check>
Si l'interface de contrôle se charge, votre Gateway est prêt à être utilisé.
</Check>

## Vérifications et extras optionnels

<AccordionGroup>
  <Accordion title="Exécuter le Gateway au premier plan">
    Utile pour des tests rapides ou le dépannage.

    ```bash
    openclaw gateway --port 18789
    ```

  </Accordion>
  <Accordion title="Envoyer un message de test">
    Nécessite un canal configuré.

    ```bash
    openclaw message send --target +15555550123 --message "Bonjour depuis OpenClaw"
    ```

  </Accordion>
</AccordionGroup>

## Variables d'environnement utiles

Si vous exécutez OpenClaw en tant que compte de service ou souhaitez des emplacements personnalisés pour la configuration et l'état :

- `OPENCLAW_HOME` définit le répertoire personnel utilisé pour la résolution des chemins internes.
- `OPENCLAW_STATE_DIR` remplace le répertoire d'état.
- `OPENCLAW_CONFIG_PATH` remplace le chemin du fichier de configuration.

Référence complète des variables d'environnement : [Variables d'environnement](/help/environment).

## Aller plus loin

<Columns>
  <Card title="Assistant de configuration initiale (détails)" href="/start/wizard">
    Référence complète de l'assistant CLI et options avancées.
  </Card>
  <Card title="Configuration initiale macOS" href="/start/onboarding">
    Flux de première exécution pour l'application macOS.
  </Card>
</Columns>

## Ce que vous obtiendrez

- Un Gateway fonctionnel
- L'authentification configurée
- L'accès à l'interface de contrôle ou un canal connecté

## Étapes suivantes

- Sécurité des messages privés et approbations : [Appairage](/channels/pairing)
- Connecter d'autres canaux : [Canaux](/channels)
- Flux avancés et installation depuis les sources : [Configuration](/start/setup)
