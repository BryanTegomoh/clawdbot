---
summary: "Référence CLI pour `openclaw secrets` (reload, audit, configure, apply)"
read_when:
  - Re-résolution des références de secrets au moment de l'exécution
  - Audit des résidus en texte clair et des références non résolues
  - Configuration des SecretRefs et application des modifications de nettoyage unidirectionnelles
title: "secrets"
---

# `openclaw secrets`

Utilisez `openclaw secrets` pour migrer les identifiants du texte clair vers les SecretRefs et maintenir le runtime des secrets actifs en bonne santé.

Rôles des commandes :

- `reload` : RPC de passerelle (`secrets.reload`) qui re-résout les références et échange l'instantané du runtime uniquement en cas de succès complet (aucune écriture de configuration).
- `audit` : analyse en lecture seule de la configuration + des magasins d'authentification + des résidus hérités (`.env`, `auth.json`) pour le texte clair, les références non résolues et la dérive de priorité.
- `configure` : planificateur interactif pour la configuration du fournisseur + le mapping de cible + la vérification préalable (TTY requis).
- `apply` : exécute un plan sauvegardé (`--dry-run` pour validation uniquement), puis nettoie les résidus de texte clair migrés.

Boucle opératoire recommandée :

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

Note sur le code de sortie pour CI/gates :

- `audit --check` retourne `1` en cas de découvertes, `2` lorsque les références ne sont pas résolues.

Connexe :

- Guide des secrets : [Gestion des secrets](/gateway/secrets)
- Guide de sécurité : [Sécurité](/gateway/security)

## Recharger l'instantané du runtime

Re-résout les références de secrets et échange atomiquement l'instantané du runtime.

```bash
openclaw secrets reload
openclaw secrets reload --json
```

Remarques :

- Utilise la méthode RPC de passerelle `secrets.reload`.
- Si la résolution échoue, la passerelle conserve le dernier instantané connu et retourne une erreur (aucune activation partielle).
- La réponse JSON inclut `warningCount`.

## Audit

Analyse l'état d'OpenClaw pour :

- stockage de secrets en texte clair
- références non résolues
- dérive de priorité (`auth-profiles` masquant les références de configuration)
- résidus hérités (`auth.json`, notes OAuth hors périmètre)

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

Comportement de sortie :

- `--check` quitte avec un code non-zéro en cas de découvertes.
- les références non résolues quittent avec un code non-zéro de priorité plus élevée.

Points clés de la forme du rapport :

- `status` : `clean | findings | unresolved`
- `summary` : `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
- codes de découverte :
  - `PLAINTEXT_FOUND`
  - `REF_UNRESOLVED`
  - `REF_SHADOWED`
  - `LEGACY_RESIDUE`

## Configure (assistant interactif)

Construit les modifications de fournisseur + SecretRef de manière interactive, exécute la vérification préalable et applique optionnellement :

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --json
```

Flux :

- Configuration du fournisseur d'abord (`add/edit/remove` pour les alias `secrets.providers`).
- Mapping des identifiants ensuite (sélectionner les champs et assigner les références `{source, provider, id}`).
- Vérification préalable et application optionnelle en dernier.

Drapeaux :

- `--providers-only` : configure `secrets.providers` uniquement, ignore le mapping des identifiants.
- `--skip-provider-setup` : ignore la configuration du fournisseur et mappe les identifiants aux fournisseurs existants.

Remarques :

- Nécessite un TTY interactif.
- Vous ne pouvez pas combiner `--providers-only` avec `--skip-provider-setup`.
- `configure` cible les champs porteurs de secrets dans `openclaw.json`.
- Incluez tous les champs porteurs de secrets que vous avez l'intention de migrer (par exemple à la fois `models.providers.*.apiKey` et `skills.entries.*.apiKey`) afin que l'audit puisse atteindre un état propre.
- Il effectue une résolution de vérification préalable avant l'application.
- Les plans générés utilisent par défaut les options de nettoyage (`scrubEnv`, `scrubAuthProfilesForProviderTargets`, `scrubLegacyAuthJson` tous activés).
- Le chemin d'application est unidirectionnel pour les valeurs de texte clair migrées.
- Sans `--apply`, le CLI demande toujours `Apply this plan now?` après la vérification préalable.
- Avec `--apply` (et sans `--yes`), le CLI demande une confirmation supplémentaire de migration irréversible.

Note de sécurité du fournisseur exec :

- Les installations Homebrew exposent souvent des binaires liés par lien symbolique sous `/opt/homebrew/bin/*`.
- Définissez `allowSymlinkCommand: true` uniquement lorsque nécessaire pour les chemins de gestionnaire de paquets de confiance, et associez-le à `trustedDirs` (par exemple `["/opt/homebrew"]`).

## Appliquer un plan sauvegardé

Applique ou vérifie au préalable un plan généré précédemment :

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

Détails du contrat du plan (chemins cibles autorisés, règles de validation et sémantique d'échec) :

- [Contrat de plan d'application des secrets](/gateway/secrets-plan-contract)

Ce que `apply` peut mettre à jour :

- `openclaw.json` (cibles SecretRef + upserts/suppressions de fournisseur)
- `auth-profiles.json` (nettoyage des cibles de fournisseur)
- résidus hérités `auth.json`
- clés de secrets connues de `~/.openclaw/.env` dont les valeurs ont été migrées

## Pourquoi pas de sauvegardes de rollback

`secrets apply` n'écrit intentionnellement pas de sauvegardes de rollback contenant les anciennes valeurs en texte clair.

La sécurité provient d'une vérification préalable stricte + une application quasi-atomique avec restauration en mémoire au mieux de l'effort en cas d'échec.

## Exemple

```bash
# Auditez d'abord, puis configurez, puis confirmez l'état propre :
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

Si `audit --check` signale toujours des découvertes de texte clair après une migration partielle, vérifiez que vous avez également migré les clés de compétences (`skills.entries.*.apiKey`) et tout autre chemin cible signalé.
