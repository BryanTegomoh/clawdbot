---
summary: "Comment OpenClaw construit le contexte de prompt et rapporte l'utilisation des jetons + coûts"
read_when:
  - Explication de l'utilisation des jetons, des coûts ou des fenêtres de contexte
  - Débogage de la croissance du contexte ou du comportement de compaction
title: "Utilisation des jetons et coûts"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/token-use.md
  workflow: manual
---

# Utilisation des jetons et coûts

OpenClaw suit les **jetons**, pas les caractères. Les jetons sont spécifiques au modèle, mais la plupart
des modèles de style OpenAI font en moyenne ~4 caractères par jeton pour le texte anglais.

## Comment le prompt système est construit

OpenClaw assemble son propre prompt système à chaque exécution. Il inclut :

- Liste des outils + descriptions courtes
- Liste des skills (uniquement les métadonnées ; les instructions sont chargées à la demande avec `read`)
- Instructions de mise à jour automatique
- Fichiers d'espace de travail + de démarrage (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` quand nouveau, plus `MEMORY.md` et/ou `memory.md` quand présents). Les fichiers volumineux sont tronqués par `agents.defaults.bootstrapMaxChars` (défaut : 20000), et l'injection bootstrap totale est plafonnée par `agents.defaults.bootstrapTotalMaxChars` (défaut : 150000). Les fichiers `memory/*.md` sont disponibles à la demande via les outils mémoire et ne sont pas auto-injectés.
- Heure (UTC + fuseau horaire utilisateur)
- Balises de réponse + comportement heartbeat
- Métadonnées d'exécution (hôte/OS/modèle/réflexion)

Voir le détail complet dans [System Prompt](/concepts/system-prompt).

## Ce qui compte dans la fenêtre de contexte

Tout ce que le modèle reçoit compte dans la limité de contexte :

- Prompt système (toutes les sections listées ci-dessus)
- Historique de conversation (messages utilisateur + assistant)
- Appels d'outils et résultats d'outils
- Pièces jointes/transcriptions (images, audio, fichiers)
- Résumés de compaction et artefacts d'élagage
- Wrappers fournisseur ou en-têtes de sécurité (non visibles, mais comptés)

Pour les images, OpenClaw réduit les charges d'images des transcriptions/outils avant les appels fournisseur.
Utilisez `agents.defaults.imageMaxDimensionPx` (défaut : `1200`) pour ajuster ceci :

- Des valeurs plus basses réduisent généralement l'utilisation de jetons de vision et la taille des charges.
- Des valeurs plus élevées préservent plus de détails visuels pour l'OCR/captures d'écran d'interface.

Pour un détail pratique (par fichier injecté, outils, skills, et taille du prompt système), utilisez `/context list` ou `/context detail`. Voir [Context](/concepts/context).

## Comment voir l'utilisation actuelle des jetons

Utilisez ceux-ci dans le chat :

- `/status` → **carte de statut riche en emojis** avec le modèle de session, l'utilisation du contexte,
  les jetons entrée/sortie de la dernière réponse, et le **coût estimé** (clé API uniquement).
- `/usage off|tokens|full` → ajouté un **pied de page d'utilisation par réponse** à chaque réponse.
  - Persiste par session (stocké comme `responseUsage`).
  - L'authentification OAuth **masque le coût** (jetons uniquement).
- `/usage cost` → affiche un résumé des coûts local à partir des journaux de session OpenClaw.

Autres surfaces :

- **TUI/Web TUI :** `/status` + `/usage` sont pris en charge.
- **CLI :** `openclaw status --usage` et `openclaw channels list` affichent
  les fenêtres de quota fournisseur (pas les coûts par réponse).

## Estimation des coûts (quand affichée)

Les coûts sont estimés à partir de votre configuration tarifaire de modèle :

```
models.providers.<provider>.models[].cost
```

Ce sont des **USD par 1M de jetons** pour `input`, `output`, `cacheRead` et
`cacheWrite`. Si la tarification est manquante, OpenClaw affiche uniquement les jetons. Les jetons
OAuth n'affichent jamais le coût en dollars.

## Impact du TTL de cache et de l'élagage

La mise en cache des prompts fournisseur ne s'applique que dans la fenêtre de TTL du cache. OpenClaw peut
optionnellement exécuter un **élagage cache-ttl** : il élague la session une fois le TTL du cache
expiré, puis réinitialise la fenêtre de cache pour que les requêtes suivantes puissent réutiliser le
contexte fraîchement mis en cache au lieu de remettre en cache tout l'historique. Cela maintient les coûts
d'écriture de cache plus bas quand une session reste inactive au-delà du TTL.

Configurez-le dans [Gateway configuration](/gateway/configuration) et consultez les
détails de comportement dans [Session pruning](/concepts/session-pruning).

Le heartbeat peut maintenir le cache **actif** pendant les périodes d'inactivité. Si le TTL de cache de votre modèle
est de `1h`, définir l'intervalle de heartbeat juste en dessous (ex. `55m`) peut éviter
de remettre en cache le prompt complet, réduisant les coûts d'écriture de cache.

Dans les configurations multi-agents, vous pouvez garder une configuration de modèle partagée et ajuster le comportement
du cache par agent avec `agents.list[].params.cacheRetention`.

Pour un guide paramètre par paramètre, voir [Prompt Caching](/reference/prompt-caching).

Pour la tarification API Anthropic, les lectures de cache sont nettement moins chères que les jetons
d'entrée, tandis que les écritures de cache sont facturées avec un multiplicateur plus élevé. Voir la
tarification de la mise en cache des prompts Anthropic pour les derniers tarifs et multiplicateurs de TTL :
[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### Exemple : maintenir le cache 1h actif avec le heartbeat

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### Exemple : trafic mixte avec stratégie de cache par agent

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # base par défaut pour la plupart des agents
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # maintenir le cache long actif pour les sessions approfondies
    - id: "alerts"
      params:
        cacheRetention: "none" # éviter les écritures de cache pour les notifications en rafale
```

`agents.list[].params` se fusionne par-dessus les `params` du modèle sélectionné, vous pouvez donc
surcharger uniquement `cacheRetention` et hériter des autres valeurs par défaut du modèle.

### Exemple : activer l'en-tête bêta de contexte 1M Anthropic

La fenêtre de contexte 1M d'Anthropic est actuellement en accès bêta restreint. OpenClaw peut injecter la
valeur `anthropic-beta` requise quand vous activez `context1m` sur les modèles Opus
ou Sonnet pris en charge.

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

Cela correspond à l'en-tête bêta `context-1m-2025-08-07` d'Anthropic.

Si vous authentifiez Anthropic avec des jetons OAuth/abonnement (`sk-ant-oat-*`),
OpenClaw ignore l'en-tête bêta `context-1m-*` car Anthropic rejette actuellement
cette combinaison avec HTTP 401.

## Conseils pour réduire la pression sur les jetons

- Utilisez `/compact` pour résumer les longues sessions.
- Réduisez les sorties d'outils volumineuses dans vos workflows.
- Diminuez `agents.defaults.imageMaxDimensionPx` pour les sessions riches en captures d'écran.
- Gardez les descriptions de skills courtes (la liste des skills est injectée dans le prompt).
- Préférez des modèles plus petits pour le travail exploratoire et verbeux.

Voir [Skills](/tools/skills) pour la formule exacte de surcoût de la liste des skills.
