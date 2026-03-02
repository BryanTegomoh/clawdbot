---
summary: "Plugin Zalo Personal : connexion QR et messagerie via zca-cli (installation du plugin, configuration du canal, CLI et outil)"
read_when:
  - Vous souhaitez le support Zalo Personal (non officiel) dans OpenClaw
  - Vous configurez ou developpez le plugin zalouser
title: "Plugin Zalo Personal"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/plugins/zalouser.md
  workflow: manual
---

# Zalo Personal (plugin)

Support de Zalo Personal pour OpenClaw via un plugin, utilisant `zca-cli` pour automatiser un compte utilisateur Zalo normal.

> **Warning:** L'automatisation non officielle peut entrainer la suspension ou le bannissement du compte. Utilisez a vos propres risques.

## Denomination

L'identifiant du canal est `zalouser` pour indiquer explicitement qu'il automatise un **compte utilisateur Zalo personnel** (non officiel). Nous reservons `zalo` pour une potentielle future intégration officielle de l'API Zalo.

## Ou s'exécute-t-il

Ce plugin s'exécute **dans le processus du Gateway**.

Si vous utilisez un Gateway distant, installez/configurez-le sur la **machine exécutant le Gateway**, puis redémarrez le Gateway.

## Installation

### Option A : installation depuis npm

```bash
openclaw plugins install @openclaw/zalouser
```

Redémarrez le Gateway ensuite.

### Option B : installation depuis un dossier local (dev)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Redémarrez le Gateway ensuite.

## Prerequis : zca-cli

La machine du Gateway doit avoir `zca` dans le `PATH` :

```bash
zca --version
```

## Configuration

La configuration du canal se trouve sous `channels.zalouser` (pas sous `plugins.entries.*`) :

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## Outil d'agent

Nom de l'outil : `zalouser`

Actions : `send`, `image`, `link`, `friends`, `groups`, `me`, `status`
