---
summary: "Référence CLI pour `openclaw security` (auditer et corriger les failles de sécurité courantes)"
read_when:
  - Vous souhaitez exécuter un audit de sécurité rapide sur la configuration/l'état
  - Vous souhaitez appliquer les suggestions de « correction » sûres (chmod, renforcement des valeurs par défaut)
title: "security"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: docs/cli/security.md
  workflow: manual
---

# `openclaw security`

Outils de sécurité (audit + corrections optionnelles).

Liens connexes :

- Guide de sécurité : [Sécurité](/gateway/security)

## Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

L'audit avertit quand plusieurs expéditeurs DM partagent la session principale et recommandé le **mode DM sécurisé** : `session.dmScope="per-channel-peer"` (ou `per-account-channel-peer` pour les canaux multi-comptes) pour les boîtes de réception partagées.
Ceci concerne le renforcement des boîtes de réception coopératives/partagées. Un seul Gateway partagé par des opérateurs mutuellement non fiables/adversaires n'est pas une configuration recommandée ; séparez les limités de confiance avec des Gateways distincts (ou des utilisateurs/hôtes OS séparés).
Il émet également `security.trust_model.multi_user_heuristic` quand la configuration suggère une ingestion multi-utilisateurs probable (par exemple, politique DM/groupe ouverte, cibles de groupe configurées, ou règles d'expéditeur génériques), et rappelle qu'OpenClaw est un modèle de confiance d'assistant personnel par défaut.
Pour les configurations multi-utilisateurs intentionnelles, la recommandation de l'audit est de sandboxer toutes les sessions, de maintenir l'accès au système de fichiers limité à l'espace de travail, et de garder les identités ou identifiants personnels/privés hors de cette exécution.
Il avertit également quand de petits modèles (`<=300B`) sont utilisés sans sandboxing et avec les outils web/navigateur activés.
Pour l'ingestion webhook, il avertit quand `hooks.defaultSessionKey` n'est pas défini, quand les substitutions `sessionKey` de requête sont activées, et quand les substitutions sont activées sans `hooks.allowedSessionKeyPrefixes`.
Il avertit aussi quand les paramètres Docker du bac à sable sont configurés alors que le mode bac à sable est désactivé, quand `gateway.nodes.denyCommands` utilisé des entrées de type pattern inefficaces/inconnues, quand `gateway.nodes.allowCommands` activé explicitement des commandes de nœud dangereuses, quand le `tools.profile="minimal"` global est surchargé par les profils d'outils des agents, quand les groupes ouverts exposent des outils d'exécution/système de fichiers sans protection bac à sable/espace de travail, et quand les outils de plugins d'extension installés peuvent être accessibles sous une politique d'outils permissive.
Il signale également `gateway.allowRealIpFallback=true` (risque d'usurpation d'en-tête si les proxies sont mal configurés) et `discovery.mdns.mode="full"` (fuite de métadonnées via les enregistrements TXT mDNS).
Il avertit aussi quand le navigateur du bac à sable utilisé le réseau Docker `bridge` sans `sandbox.browser.cdpSourceRange`.
Il signale les modes réseau Docker dangereux du bac à sable (incluant `host` et les jonctions d'espace de noms `container:*`).
Il avertit quand les conteneurs Docker existants du navigateur du bac à sable ont des labels de hash manquants/périmés (par exemple des conteneurs pré-migration manquant `openclaw.browserConfigEpoch`) et recommandé `openclaw sandbox recreate --browser --all`.
Il avertit quand les enregistrements d'installation npm des plugins/hooks sont non épinglés, manquent de métadonnées d'intégrité, ou divergent des versions de packages actuellement installés.
Il avertit quand les listes d'autorisation de canaux reposent sur des noms/emails/tags mutables au lieu d'identifiants stables (Discord, Slack, Google Chat, MS Teams, Mattermost, portées IRC si applicable).
Il avertit quand `gateway.auth.mode="none"` laisse les API HTTP du Gateway accessibles sans secret partagé (`/tools/invoke` plus tout point de terminaison `/v1/*` activé).
Les paramètres préfixés par `dangerous`/`dangerously` sont des surcharges explicites d'opérateur de type « bris de glace » ; en activer un n'est pas, en soi, un rapport de vulnérabilité de sécurité.
Pour l'inventaire complet des paramètres dangereux, voir la section « Résumé des indicateurs non sécurisés ou dangereux » dans [Sécurité](/gateway/security).

## Sortie JSON

Utilisez `--json` pour les vérifications CI/politique :

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

Si `--fix` et `--json` sont combinés, la sortie inclut à la fois les actions de correction et le rapport final :

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## Ce que `--fix` modifie

`--fix` applique des remédiations sûres et déterministes :

- bascule le `groupPolicy="open"` courant vers `groupPolicy="allowlist"` (incluant les variantes de compte dans les canaux pris en charge)
- définit `logging.redactSensitive` de `"off"` à `"tools"`
- renforce les permissions pour l'état/la configuration et les fichiers sensibles courants (`credentials/*.json`, `auth-profiles.json`, `sessions.json`, `*.jsonl` de session)

`--fix` ne fait **pas** :

- la rotation des jetons/mots de passe/clés API
- la désactivation des outils (`gateway`, `cron`, `exec`, etc.)
- la modification des choix de liaison/authentification/exposition réseau du Gateway
- la suppression ou la réécriture des plugins/Skills
