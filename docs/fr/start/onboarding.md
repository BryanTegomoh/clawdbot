---
read_when:
  - Conception de l'assistant de configuration initiale macOS
  - Mise en place de l'authentification ou de l'identité
summary: Flux de configuration initiale au premier lancement d'OpenClaw (application macOS)
title: "Configuration initiale (application macOS)"
sidebarTitle: "Configuration initiale : application macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: start/onboarding.md
  workflow: manual
---

# Configuration initiale (application macOS)

Ce document décrit le flux de configuration initiale **actuel** au premier lancement. L'objectif est une
expérience « jour 0 » fluide : choisir où le Gateway s'exécute, connecter l'authentification, lancer
l'assistant et laisser l'agent s'initialiser.
Pour une vue d'ensemble des chemins de configuration initiale, consultez [Vue d'ensemble de la configuration initiale](/start/onboarding-overview).

<Steps>
<Step title="Approuver l'avertissement macOS">
<Frame>
<img src="/assets/macos-onboarding/01-macos-warning.jpeg" alt="" />
</Frame>
</Step>
<Step title="Approuver la recherche de réseaux locaux">
<Frame>
<img src="/assets/macos-onboarding/02-local-networks.jpeg" alt="" />
</Frame>
</Step>
<Step title="Message de bienvenue et avis de sécurité">
<Frame caption="Lisez l'avis de sécurité affiché et décidez en conséquence">
<img src="/assets/macos-onboarding/03-security-notice.png" alt="" />
</Frame>

Modèle de confiance de sécurité :

- Par défaut, OpenClaw est un agent personnel : une seule frontière opérateur de confiance.
- Les configurations partagées/multi-utilisateurs nécessitent un verrouillage (séparer les frontières de confiance, limiter l'accès aux outils au minimum et suivre [Sécurité](/gateway/security)).

</Step>
<Step title="Local vs Distant">
<Frame>
<img src="/assets/macos-onboarding/04-choose-gateway.png" alt="" />
</Frame>

Où le **Gateway** s'exécute-t-il ?

- **Ce Mac (local uniquement) :** la configuration initiale peut configurer l'authentification et écrire les identifiants
  localement.
- **Distant (via SSH/Tailnet) :** la configuration initiale **ne configure pas** l'authentification localement ;
  les identifiants doivent exister sur l'hôte du gateway.
- **Configurer plus tard :** ignorer la configuration et laisser l'application non configurée.

<Tip>
**Conseil sur l'authentification du Gateway :**
- L'assistant génère maintenant un **jeton** même pour le loopback, donc les clients WS locaux doivent s'authentifier.
- Si vous désactivez l'authentification, tout processus local peut se connecter ; utilisez cela uniquement sur des machines entièrement de confiance.
- Utilisez un **jeton** pour l'accès multi-machines ou les liaisons non-loopback.
</Tip>
</Step>
<Step title="Permissions">
<Frame caption="Choisissez les permissions que vous souhaitez accorder à OpenClaw">
<img src="/assets/macos-onboarding/05-permissions.png" alt="" />
</Frame>

La configuration initiale demande les permissions TCC nécessaires pour :

- Automatisation (AppleScript)
- Notifications
- Accèssibilité
- Enregistrement d'écran
- Microphone
- Reconnaissance vocale
- Caméra
- Localisation

</Step>
<Step title="CLI">
  <Info>Cette étape est optionnelle</Info>
  L'application peut installer le CLI global `openclaw` via npm/pnpm afin que les
  flux de travail en terminal et les tâches launchd fonctionnent immédiatement.
</Step>
<Step title="Chat de configuration initiale (session dédiée)">
  Après la configuration, l'application ouvre une session de chat dédiée pour que l'agent puisse
  se présenter et guider les étapes suivantes. Cela permet de séparer les conseils de première exécution
  de votre conversation normale. Consultez [Initialisation](/start/bootstrapping) pour
  ce qui se passe sur l'hôte du gateway lors du premier lancement de l'agent.
</Step>
</Steps>
