---
summary: "Authentification des modèles : OAuth, clés API et setup-token"
read_when:
  - Débogage de l'authentification des modèles ou de l'expiration OAuth
  - Documentation de l'authentification ou du stockage des identifiants
title: "Authentification"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/authentication.md
  workflow: manual
---

# Authentification

OpenClaw prend en charge OAuth et les clés API pour les fournisseurs de modèles. Pour les comptes Anthropic, nous recommandons l'utilisation d'une **clé API**. Pour l'accès par abonnement Claude, utilisez le jeton longue durée créé par `claude setup-token`.

Voir [/concepts/oauth](/concepts/oauth) pour le flux OAuth complet et la structure de stockage.
Pour l'authentification basee sur SecretRef (fournisseurs `env`/`file`/`exec`), voir [Gestion des secrets](/gateway/secrets).

## Configuration Anthropic recommandée (clé API)

Si vous utilisez Anthropic directement, utilisez une clé API.

1. Créez une clé API dans la console Anthropic.
2. Placez-la sur l'**hôte du gateway** (la machine exécutant `openclaw gateway`).

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. Si le Gateway fonctionne sous systemd/launchd, préférez placer la clé dans
   `~/.openclaw/.env` pour que le démon puisse la lire :

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

Ensuite, redémarrez le démon (ou redémarrez votre processus Gateway) et vérifiez à nouveau :

```bash
openclaw models status
openclaw doctor
```

Si vous préférez ne pas gérer les variables d'environnement vous-même, l'assistant d'intégration peut stocker les clés API pour l'utilisation par le démon : `openclaw onboard`.

Voir [Help](/help) pour plus de détails sur l'héritage des variables d'environnement (`env.shellEnv`, `~/.openclaw/.env`, systemd/launchd).

## Anthropic : setup-token (authentification par abonnement)

Pour Anthropic, le chemin recommandé est une **clé API**. Si vous utilisez un abonnement Claude, le flux setup-token est également pris en charge. Exécutez-le sur l'**hôte du gateway** :

```bash
claude setup-token
```

Puis collez-le dans OpenClaw :

```bash
openclaw models auth setup-token --provider anthropic
```

Si le jeton a été créé sur une autre machine, collez-le manuellement :

```bash
openclaw models auth paste-token --provider anthropic
```

Si vous voyez une erreur Anthropic telle que :

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

...utilisez plutôt une clé API Anthropic.

Saisie manuelle du jeton (tout fournisseur ; écrit dans `auth-profiles.json` + met à jour la configuration) :

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

Les refs de profil d'authentification sont également prises en charge pour les identifiants statiques :

- Les identifiants `api_key` peuvent utiliser `keyRef: { source, provider, id }`
- Les identifiants `token` peuvent utiliser `tokenRef: { source, provider, id }`

Vérification compatible avec l'automatisation (code de sortie `1` si expiré/absent, `2` si en cours d'expiration) :

```bash
openclaw models status --check
```

Les scripts d'opérations optionnels (systemd/Termux) sont documentés ici :
[/automation/auth-monitoring](/automation/auth-monitoring)

> `claude setup-token` nécessite un TTY interactif.

## Vérification du statut d'authentification des modèles

```bash
openclaw models status
openclaw doctor
```

## Comportement de rotation des clés API (gateway)

Certains fournisseurs prennent en charge la relance d'une requête avec des clés alternatives lorsqu'un appel API atteint une limite de débit du fournisseur.

- Ordre de priorité :
  - `OPENCLAW_LIVE_<PROVIDER>_KEY` (remplacement unique)
  - `<PROVIDER>_API_KEYS`
  - `<PROVIDER>_API_KEY`
  - `<PROVIDER>_API_KEY_*`
- Les fournisseurs Google incluent également `GOOGLE_API_KEY` comme repli supplémentaire.
- La même liste de clés est dédupliquée avant utilisation.
- OpenClaw relance avec la clé suivante uniquement pour les erreurs de limité de débit (par exemple `429`, `rate_limit`, `quota`, `resource exhausted`).
- Les erreurs non liées aux limités de débit ne sont pas relancées avec des clés alternatives.
- Si toutes les clés échouent, l'erreur finale de la dernière tentative est renvoyée.

## Contrôle de l'identifiant utilisé

### Par session (commande chat)

Utilisez `/model <alias-ou-id>@<profileId>` pour fixer un identifiant de fournisseur spécifique pour la session en cours (exemples d'identifiants de profil : `anthropic:default`, `anthropic:work`).

Utilisez `/model` (ou `/model list`) pour un sélecteur compact ; utilisez `/model status` pour la vue complète (candidats + prochain profil d'authentification, plus détails du point d'accès du fournisseur lorsqu'ils sont configurés).

### Par agent (remplacement CLI)

Définissez un ordre de profil d'authentification explicite pour un agent (stocké dans le `auth-profiles.json` de cet agent) :

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

Utilisez `--agent <id>` pour cibler un agent spécifique ; omettez-le pour utiliser l'agent par défaut configuré.

## Dépannage

### « No credentials found »

Si le profil de jeton Anthropic est manquant, exécutez `claude setup-token` sur l'**hôte du gateway**, puis vérifiez à nouveau :

```bash
openclaw models status
```

### Jeton en cours d'expiration/expiré

Exécutez `openclaw models status` pour confirmer quel profil expire. Si le profil est manquant, relancez `claude setup-token` et collez à nouveau le jeton.

## Prérequis

- Abonnement Claude Max ou Pro (pour `claude setup-token`)
- CLI Claude Code installé (commande `claude` disponible)
