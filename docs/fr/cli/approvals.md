---
summary: "Référence CLI pour `openclaw approvals` (approbations d'exécution pour le Gateway ou les node hosts)"
read_when:
  - Vous souhaitez modifier les approbations d'exécution depuis le CLI
  - Vous devez gérer les listes d'autorisation sur le Gateway ou les node hosts
title: "approvals"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: cli/approvals.md
  workflow: manual
---

# `openclaw approvals`

Gérer les approbations d'exécution pour l'**hôte local**, l'**hôte Gateway** ou un **node host**.
Par défaut, les commandes ciblent le fichier d'approbations local sur le disque. Utilisez `--gateway` pour cibler le Gateway, ou `--node` pour cibler un node spécifique.

Liens :

- Approbations d'exécution : [Exec approvals](/tools/exec-approvals)
- Nodes : [Nodes](/nodes)

## Commandes courantes

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## Remplacer les approbations depuis un fichier

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## Utilitaires de liste d'autorisation

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## Notes

- `--node` utilisé le même résolveur que `openclaw nodes` (id, nom, ip ou préfixe d'id).
- `--agent` est par défaut `"*"`, ce qui s'applique à tous les agents.
- Le node host doit annoncer `system.execApprovals.get/set` (application macOS ou node host headless).
- Les fichiers d'approbations sont stockes par hôte dans `~/.openclaw/exec-approvals.json`.
