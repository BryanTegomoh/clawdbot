---
summary: "Gestion des secrets : contrat SecretRef, comportement de l'instantané du runtime et nettoyage unidirectionnel sécurisé"
read_when:
  - Configuration des SecretRefs pour les fournisseurs, profils d'authentification, compétences ou Google Chat
  - Utilisation de secrets reload/audit/configure/apply en toute sécurité en production
  - Compréhension du comportement fail-fast et last-known-good
title: "Gestion des secrets"
---

# Gestion des secrets

OpenClaw prend en charge les références de secrets additives afin que les identifiants n'aient pas besoin d'être stockés en texte clair dans les fichiers de configuration.

Le texte clair fonctionne toujours. Les références de secrets sont facultatives.

## Objectifs et modèle runtime

Les secrets sont résolus dans un instantané runtime en mémoire.

- La résolution est immédiate lors de l'activation, pas paresseuse sur les chemins de requête.
- Le démarrage échoue rapidement si un identifiant référencé ne peut pas être résolu.
- Le rechargement utilise un échange atomique : succès complet ou conservation du dernier instantané connu.
- Les requêtes runtime lisent à partir de l'instantané actif en mémoire.

Cela maintient les pannes des fournisseurs de secrets hors du chemin de requête critique.

## Vérification préalable de référence lors de l'intégration

Lorsque l'intégration s'exécute en mode interactif et que vous choisissez le stockage de référence de secret, OpenClaw effectue une vérification préalable rapide avant d'enregistrer :

- Références env : valide le nom de la variable d'environnement et confirme qu'une valeur non vide est visible lors de l'intégration.
- Références de fournisseur (`file` ou `exec`) : valide le fournisseur sélectionné, résout l'`id` fourni et vérifie le type de valeur.

Si la validation échoue, l'intégration affiche l'erreur et vous permet de réessayer.

## Contrat SecretRef

Utilisez une seule forme d'objet partout :

```json5
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### `source: "env"`

```json5
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

Validation :

- `provider` doit correspondre à `^[a-z][a-z0-9_-]{0,63}$`
- `id` doit correspondre à `^[A-Z][A-Z0-9_]{0,127}$`

### `source: "file"`

```json5
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

Validation :

- `provider` doit correspondre à `^[a-z][a-z0-9_-]{0,63}$`
- `id` doit être un pointeur JSON absolu (`/...`)
- Échappement RFC6901 dans les segments : `~` => `~0`, `/` => `~1`

### `source: "exec"`

```json5
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

Validation :

- `provider` doit correspondre à `^[a-z][a-z0-9_-]{0,63}$`
- `id` doit correspondre à `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$`

## Configuration du fournisseur

Définissez les fournisseurs sous `secrets.providers` :

```json5
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // ou "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### Fournisseur Env

- Liste d'autorisation optionnelle via `allowlist`.
- Les valeurs env manquantes/vides font échouer la résolution.

### Fournisseur File

- Lit le fichier local à partir de `path`.
- `mode: "json"` attend une charge JSON objet et résout `id` comme pointeur.
- `mode: "singleValue"` attend l'identifiant de référence `"value"` et retourne le contenu du fichier.
- Le chemin doit passer les vérifications de propriété/permission.

### Fournisseur Exec

- Exécute le chemin binaire absolu configuré, sans shell.
- Par défaut, `command` doit pointer vers un fichier régulier (pas un lien symbolique).
- Définissez `allowSymlinkCommand: true` pour autoriser les chemins de commande par lien symbolique (par exemple les shims Homebrew). OpenClaw valide le chemin cible résolu.
- Activez `allowSymlinkCommand` uniquement lorsque requis pour les chemins de gestionnaire de paquets de confiance, et associez-le à `trustedDirs` (par exemple `["/opt/homebrew"]`).
- Lorsque `trustedDirs` est défini, les vérifications s'appliquent au chemin cible résolu.
- Prend en charge le timeout, le timeout sans sortie, les limites d'octets de sortie, la liste d'autorisation env et les répertoires de confiance.
- Charge de requête (stdin) :

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

- Charge de réponse (stdout) :

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "sk-..." } }
```

Erreurs optionnelles par id :

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```

## Exemples d'intégration Exec

### 1Password CLI

```json5
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // requis pour les binaires liés par Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### HashiCorp Vault CLI

```json5
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // requis pour les binaires liés par Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```

### `sops`

```json5
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // requis pour les binaires liés par Homebrew
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```

## Champs dans le périmètre (v1)

### `~/.openclaw/openclaw.json`

- `models.providers.<provider>.apiKey`
- `skills.entries.<skillKey>.apiKey`
- `channels.googlechat.serviceAccount`
- `channels.googlechat.serviceAccountRef`
- `channels.googlechat.accounts.<accountId>.serviceAccount`
- `channels.googlechat.accounts.<accountId>.serviceAccountRef`

### `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

- `profiles.<profileId>.keyRef` pour `type: "api_key"`
- `profiles.<profileId>.tokenRef` pour `type: "token"`

Les modifications du stockage des identifiants OAuth sont hors périmètre.

## Comportement requis et priorité

- Champ sans référence : inchangé.
- Champ avec référence : requis au moment de l'activation.
- Si le texte clair et la référence existent tous les deux, la référence l'emporte au runtime et le texte clair est ignoré.

Code d'avertissement :

- `SECRETS_REF_OVERRIDES_PLAINTEXT`

## Déclencheurs d'activation

L'activation des secrets est tentée lors :

- Du démarrage (vérification préalable plus activation finale)
- Du chemin d'application à chaud du rechargement de configuration
- Du chemin de vérification de redémarrage du rechargement de configuration
- Du rechargement manuel via `secrets.reload`

Contrat d'activation :

- Le succès échange l'instantané atomiquement.
- L'échec au démarrage abandonne le démarrage de la passerelle.
- L'échec au rechargement du runtime conserve le dernier instantané connu.

## Signaux opérateur dégradé et récupéré

Lorsque l'activation au moment du rechargement échoue après un état sain, OpenClaw entre dans un état de secrets dégradé.

Codes d'événement système et de log uniques :

- `SECRETS_RELOADER_DEGRADED`
- `SECRETS_RELOADER_RECOVERED`

Comportement :

- Dégradé : le runtime conserve le dernier instantané connu.
- Récupéré : émis une fois après une activation réussie.
- Les échecs répétés en état déjà dégradé enregistrent des avertissements mais n'inondent pas les événements.
- L'échec rapide au démarrage n'émet pas d'événements dégradés car aucun instantané runtime n'existe encore.

## Flux de travail d'audit et de configuration

Utilisez ce flux opérateur par défaut :

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

Complétude de la migration :

- Incluez les cibles `skills.entries.<skillKey>.apiKey` lorsque ces compétences utilisent des clés API.
- Si `audit --check` signale toujours des découvertes de texte clair après une migration partielle, migrez les chemins signalés restants et réexécutez l'audit.

### `secrets audit`

Les découvertes incluent :

- valeurs en texte clair au repos (`openclaw.json`, `auth-profiles.json`, `.env`)
- références non résolues
- masquage de priorité (`auth-profiles` ayant la priorité sur les références de configuration)
- résidus hérités (`auth.json`, rappels OAuth hors périmètre)

### `secrets configure`

Assistant interactif qui :

- configure `secrets.providers` d'abord (`env`/`file`/`exec`, add/edit/remove)
- vous permet de sélectionner les champs porteurs de secrets dans `openclaw.json`
- capture les détails SecretRef (`source`, `provider`, `id`)
- exécute la résolution de vérification préalable
- peut appliquer immédiatement

Modes utiles :

- `openclaw secrets configure --providers-only`
- `openclaw secrets configure --skip-provider-setup`

`configure` applique par défaut :

- nettoyage des identifiants statiques correspondants de `auth-profiles.json` pour les fournisseurs ciblés
- nettoyage des entrées `api_key` statiques héritées de `auth.json`
- nettoyage des lignes de secrets connues correspondantes de `<config-dir>/.env`

### `secrets apply`

Appliquer un plan sauvegardé :

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

Pour les détails du contrat cible/chemin strict et les règles de rejet exactes, voir :

- [Contrat de plan d'application des secrets](/gateway/secrets-plan-contract)

## Politique de sécurité unidirectionnelle

OpenClaw n'écrit intentionnellement **pas** de sauvegardes de rollback contenant les valeurs de secret en texte clair pré-migration.

Modèle de sécurité :

- la vérification préalable doit réussir avant le mode écriture
- l'activation du runtime est validée avant la validation
- apply met à jour les fichiers en utilisant le remplacement atomique de fichiers et la restauration en mémoire au mieux de l'effort en cas d'échec

## Notes de compatibilité `auth.json`

Pour les identifiants statiques, le runtime OpenClaw ne dépend plus du texte clair `auth.json`.

- La source d'identifiants runtime est l'instantané résolu en mémoire.
- Les entrées `api_key` statiques héritées de `auth.json` sont nettoyées lorsqu'elles sont découvertes.
- Le comportement de compatibilité hérité lié à OAuth reste séparé.

## Documents connexes

- Commandes CLI : [secrets](/cli/secrets)
- Détails du contrat de plan : [Contrat de plan d'application des secrets](/gateway/secrets-plan-contract)
- Configuration d'authentification : [Authentification](/gateway/authentication)
- Posture de sécurité : [Sécurité](/gateway/security)
- Priorité des variables d'environnement : [Variables d'environnement](/help/environment)
