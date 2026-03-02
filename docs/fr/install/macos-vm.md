---
summary: "Exécuter OpenClaw dans une VM macOS isolée (locale ou hébergée) pour l'isolation ou iMessage"
read_when:
  - Vous voulez OpenClaw isolé de votre environnement macOS principal
  - Vous voulez l'intégration iMessage (BlueBubbles) dans un bac à sable
  - Vous voulez un environnement macOS réinitialisable que vous pouvez cloner
  - Vous voulez comparer les options de VM macOS locales vs hébergées
title: "VM macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: install/macos-vm.md
  workflow: manual
---

# OpenClaw sur VM macOS (bac à sable)

## Recommandation par défaut (la plupart des utilisateurs)

- **Petit VPS Linux** pour un Gateway toujours actif à faible coût. Voir [Hébergement VPS](/vps).
- **Matériel dédié** (Mac mini ou machine Linux) si vous voulez un contrôle total et une **IP résidentielle** pour l'automatisation du navigateur. De nombreux sites bloquent les IP de centres de données, donc la navigation locale fonctionne souvent mieux.
- **Hybride :** gardez le Gateway sur un VPS bon marché et connectez votre Mac comme **nœud** quand vous avez besoin d'automatisation navigateur/UI. Voir [Nœuds](/nodes) et [Gateway distant](/gateway/remote).

Utilisez une VM macOS quand vous avez spécifiquement besoin de fonctionnalités exclusives à macOS (iMessage/BlueBubbles) ou voulez une isolation stricte de votre Mac quotidien.

## Options de VM macOS

### VM locale sur votre Mac Apple Silicon (Lume)

Exécutez OpenClaw dans une VM macOS isolée sur votre Mac Apple Silicon existant avec [Lume](https://cua.ai/docs/lume).

Cela vous donne :

- Environnement macOS complet en isolation (votre hôte reste propre)
- Support iMessage via BlueBubbles (impossible sur Linux/Windows)
- Réinitialisation instantanée par clonage de VM
- Pas de matériel supplémentaire ni de coûts cloud

### Fournisseurs Mac hébergés (cloud)

Si vous voulez macOS dans le cloud, les fournisseurs Mac hébergés fonctionnent aussi :

- [MacStadium](https://www.macstadium.com/) (Macs hébergés)
- D'autres fournisseurs Mac hébergés fonctionnent aussi ; suivez leur documentation VM + SSH

Une fois que vous avez l'accès SSH à une VM macOS, continuez à l'étape 6 ci-dessous.

---

## Chemin rapide (Lume, utilisateurs expérimentés)

1. Installer Lume
2. `lume create openclaw --os macos --ipsw latest`
3. Compléter l'Assistant de configuration, activer la Connexion à distance (SSH)
4. `lume run openclaw --no-display`
5. Se connecter en SSH, installer OpenClaw, configurer les canaux
6. Terminé

---

## Ce dont vous avez besoin (Lume)

- Mac Apple Silicon (M1/M2/M3/M4)
- macOS Sequoia ou ultérieur sur l'hôte
- ~60 Go d'espace disque libre par VM
- ~20 minutes

---

## 1) Installer Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Si `~/.local/bin` n'est pas dans votre PATH :

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Vérifiez :

```bash
lume --version
```

Documentation : [Installation de Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

---

## 2) Créer la VM macOS

```bash
lume create openclaw --os macos --ipsw latest
```

Cela télécharge macOS et crée la VM. Une fenêtre VNC s'ouvre automatiquement.

Note : le téléchargement peut prendre du temps selon votre connexion.

---

## 3) Compléter l'Assistant de configuration

Dans la fenêtre VNC :

1. Sélectionnez la langue et la région
2. Ignorez l'identifiant Apple (ou connectez-vous si vous voulez iMessage plus tard)
3. Créez un compte utilisateur (retenez le nom d'utilisateur et le mot de passe)
4. Ignorez toutes les fonctionnalités optionnelles

Après la fin de la configuration, activez SSH :

1. Ouvrez Réglages Système, puis Général, puis Partage
2. Activez « Connexion à distance »

---

## 4) Obtenir l'adresse IP de la VM

```bash
lume get openclaw
```

Cherchez l'adresse IP (généralement `192.168.64.x`).

---

## 5) Se connecter en SSH à la VM

```bash
ssh youruser@192.168.64.X
```

Remplacez `youruser` par le compte que vous avez créé, et l'IP par celle de votre VM.

---

## 6) Installer OpenClaw

Dans la VM :

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Suivez les invites de configuration initiale pour configurer votre fournisseur de modèle (Anthropic, OpenAI, etc.).

---

## 7) Configurer les canaux

Éditez le fichier de configuration :

```bash
nano ~/.openclaw/openclaw.json
```

Ajoutez vos canaux :

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Puis connectez-vous à WhatsApp (scanner le QR) :

```bash
openclaw channels login
```

---

## 8) Exécuter la VM en mode headless

Arrêtez la VM et redémarrez sans affichage :

```bash
lume stop openclaw
lume run openclaw --no-display
```

La VM s'exécute en arrière-plan. Le daemon d'OpenClaw maintient le gateway en fonctionnement.

Pour vérifier le statut :

```bash
ssh youruser@192.168.64.X "openclaw status"
```

---

## Bonus : intégration iMessage

C'est la fonctionnalité phare de l'exécution sur macOS. Utilisez [BlueBubbles](https://bluebubbles.app) pour ajouter iMessage à OpenClaw.

Dans la VM :

1. Téléchargez BlueBubbles depuis bluebubbles.app
2. Connectez-vous avec votre identifiant Apple
3. Activez l'API Web et définissez un mot de passe
4. Pointez les webhooks BlueBubbles vers votre gateway (exemple : `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

Ajoutez à votre configuration OpenClaw :

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Redémarrez le gateway. Votre agent peut maintenant envoyer et recevoir des iMessages.

Détails complets de la configuration : [Canal BlueBubbles](/channels/bluebubbles)

---

## Sauvegarder une image de référence

Avant de personnaliser davantage, prenez un instantané de votre état propre :

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Réinitialisez à tout moment :

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

---

## Fonctionnement 24/7

Gardez la VM en fonctionnement en :

- Gardant votre Mac branché
- Désactivant la mise en veille dans Réglages Système, puis Économie d'énergie
- Utilisant `caffeinate` si nécessaire

Pour un fonctionnement véritablement permanent, envisagez un Mac mini dédié ou un petit VPS. Voir [Hébergement VPS](/vps).

---

## Dépannage

| Problème                           | Solution                                                                           |
| ---------------------------------- | ---------------------------------------------------------------------------------- |
| Impossible de se connecter en SSH  | Vérifiez que « Connexion à distance » est activée dans les Réglages Système de la VM |
| L'IP de la VM n'apparaît pas       | Attendez le démarrage complet de la VM, exécutez `lume get openclaw` à nouveau      |
| Commande Lume introuvable          | Ajoutez `~/.local/bin` à votre PATH                                                 |
| Le QR WhatsApp ne se scanne pas    | Assurez-vous d'être connecté à la VM (pas l'hôte) lors de l'exécution de `openclaw channels login` |

---

## Documentation connexe

- [Hébergement VPS](/vps)
- [Nœuds](/nodes)
- [Gateway distant](/gateway/remote)
- [Canal BlueBubbles](/channels/bluebubbles)
- [Démarrage rapide Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
- [Référence CLI Lume](https://cua.ai/docs/lume/reference/cli-reference)
- [Configuration VM sans surveillance](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (avancé)
- [Bac à sable Docker](/install/docker) (approche