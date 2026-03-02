---
summary: "Contrat pour les plans `secrets apply` : chemins cibles autorisés, validation et comportement auth-profile en référence uniquement"
read_when:
  - Génération ou révision de fichiers de plan `openclaw secrets apply`
  - Débogage des erreurs `Invalid plan target path`
  - Compréhension de l'influence de `keyRef` et `tokenRef` sur la découverte implicite des fournisseurs
title: "Contrat de plan d'application des secrets"
---

# Contrat de plan d'application des secrets

Cette page définit le contrat strict appliqué par `openclaw secrets apply`.

Si une cible ne correspond pas à ces règles, l'application échoue avant de muter la configuration.

## Forme du fichier de plan

`openclaw secrets apply --from <plan.json>` attend un tableau `targets` de cibles de plan :

```json5
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```

## Types de cibles et chemins autorisés

| `target.type`                        | Forme `target.path` autorisée                             | Règle de correspondance d'id optionnelle            |
| ------------------------------------ | --------------------------------------------------------- | --------------------------------------------------- |
| `models.providers.apiKey`            | `models.providers.<providerId>.apiKey`                    | `providerId` doit correspondre à `<providerId>` s'il est présent |
| `skills.entries.apiKey`              | `skills.entries.<skillKey>.apiKey`                        | n/a                                                 |
| `channels.googlechat.serviceAccount` | `channels.googlechat.serviceAccount`                      | `accountId` doit être vide/omis                     |
| `channels.googlechat.serviceAccount` | `channels.googlechat.accounts.<accountId>.serviceAccount` | `accountId` doit correspondre à `<accountId>` s'il est présent |

## Règles de validation du chemin

Chaque cible est validée avec toutes les règles suivantes :

- `type` doit être l'un des types de cibles autorisés ci-dessus.
- `path` doit être un chemin à points non vide.
- `pathSegments` peut être omis. S'il est fourni, il doit se normaliser exactement au même chemin que `path`.
- Les segments interdits sont rejetés : `__proto__`, `prototype`, `constructor`.
- Le chemin normalisé doit correspondre à l'une des formes de chemin autorisées pour le type de cible.
- Si `providerId` / `accountId` est défini, il doit correspondre à l'id encodé dans le chemin.

## Comportement en cas d'échec

Si une cible échoue la validation, apply quitte avec une erreur comme :

```text
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

Aucune mutation partielle n'est validée pour ce chemin de cible invalide.

## Profils d'authentification en référence uniquement et fournisseurs implicites

La découverte implicite des fournisseurs considère également les profils d'authentification qui stockent des références au lieu d'identifiants en texte clair :

- Les profils `type: "api_key"` peuvent utiliser `keyRef` (par exemple références basées sur env).
- Les profils `type: "token"` peuvent utiliser `tokenRef`.

Comportement :

- Pour les fournisseurs de clé API (par exemple `volcengine`, `byteplus`), les profils en référence uniquement peuvent toujours activer les entrées de fournisseur implicites.
- Pour `github-copilot`, si le profil n'a pas de token en texte clair, la découverte essaiera la résolution env `tokenRef` avant l'échange de token.

## Vérifications opérateur

```bash
# Valider le plan sans écritures
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# Puis appliquer pour de vrai
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

Si apply échoue avec un message de chemin de cible invalide, régénérez le plan avec `openclaw secrets configure` ou corrigez le chemin de cible à l'une des formes autorisées ci-dessus.

## Documents connexes

- [Gestion des secrets](/gateway/secrets)
- [CLI `secrets`](/cli/secrets)
- [Référence de configuration](/gateway/configuration-reference)
