---
summary: "Guide de configuration pour les developpeurs travaillant sur l'application macOS OpenClaw"
read_when:
  - Mise en place de l'environnement de développement macOS
title: "Configuration dev macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/dev-setup.md
  workflow: manual
---

# Configuration developpeur macOS

Ce guide couvre les étapes nécessaires pour compiler et exécuter l'application macOS OpenClaw depuis les sources.

## Prerequis

Avant de compiler l'application, assurez-vous que les éléments suivants sont installes :

1. **Xcode 26.2+** : requis pour le développement Swift.
2. **Node.js 22+ et pnpm** : requis pour le gateway, le CLI et les scripts d'empaquetage.

## 1. Installer les dependances

Installez les dependances du projet :

```bash
pnpm install
```

## 2. Compiler et empaqueter l'application

Pour compiler l'application macOS et l'empaqueter dans `dist/OpenClaw.app`, exécutez :

```bash
./scripts/package-mac-app.sh
```

Si vous n'avez pas de certificat Apple Developer ID, le script utilisera automatiquement la **signature ad-hoc** (`-`).

Pour les modes d'exécution de développement, les drapeaux de signature et le dépannage de Team ID, consultez le README de l'application macOS :
[https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **Note** : les applications signees ad-hoc peuvent déclencher des invites de sécurité. Si l'application plante immédiatement avec « Abort trap 6 », consultez la section [Dépannage](#dépannage).

## 3. Installer le CLI

L'application macOS attend une installation globale du CLI `openclaw` pour gérer les taches en arriere-plan.

**Pour l'installer (recommandé) :**

1. Ouvrez l'application OpenClaw.
2. Allez dans l'onglet **Général** des paramètres.
3. Cliquez sur **« Install CLI »**.

Alternativement, installez-le manuellement :

```bash
npm install -g openclaw@<version>
```

## Dépannage

### Échec de compilation : incompatibilite de toolchain ou SDK

La compilation de l'application macOS attend le dernier SDK macOS et la toolchain Swift 6.2.

**Dependances système (requises) :**

- **Dernière version de macOS disponible dans Mise a jour logicielle** (requise par les SDK Xcode 26.2)
- **Xcode 26.2** (toolchain Swift 6.2)

**Vérifications :**

```bash
xcodebuild -version
xcrun swift --version
```

Si les versions ne correspondent pas, mettez a jour macOS/Xcode et relancez la compilation.

### L'application plante a l'octroi de permissions

Si l'application plante lorsque vous essayez d'autoriser l'accès a la **Reconnaissance vocale** ou au **Microphone**, cela peut être du a un cache TCC corrompu ou une incompatibilite de signature.

**Correction :**

1. Reinitialisez les permissions TCC :

   ```bash
   tccutil reset All ai.openclaw.mac.debug
   ```

2. Si cela échoue, changez temporairement le `BUNDLE_ID` dans [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) pour forcer un « nouvel état » depuis macOS.

### Le Gateway reste a « Starting... » indefiniment

Si le statut du gateway reste sur « Starting... », verifiez si un processus zombie occupe le port :

```bash
openclaw gateway status
openclaw gateway stop

# Si vous n'utilisez pas de LaunchAgent (mode dev / executions manuelles), trouvez l'ecouteur :
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Si une exécution manuelle occupe le port, arretez ce processus (Ctrl+C). En dernier recours, tuez le PID trouve ci-dessus.
