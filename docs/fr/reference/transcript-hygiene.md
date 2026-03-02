---
summary: "Référence : règles d'assainissement et de réparation des transcriptions spécifiques aux fournisseurs"
read_when:
  - Vous déboguez des rejets de requêtes fournisseur liés à la forme des transcriptions
  - Vous modifiez la logique d'assainissement des transcriptions ou de réparation des appels d'outils
  - Vous enquêtez sur des incohérences d'identifiants d'appels d'outils entre fournisseurs
title: "Hygiène des transcriptions"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/transcript-hygiene.md
  workflow: manual
---

# Hygiène des transcriptions (Corrections fournisseur)

Ce document décrit les **corrections spécifiques aux fournisseurs** appliquées aux transcriptions avant une exécution
(construction du contexte modèle). Ce sont des ajustements **en mémoire** utilisés pour satisfaire les exigences
strictes des fournisseurs. Ces étapes d'hygiène ne **réécrivent pas** la transcription JSONL stockée
sur le disque ; cependant, une passe de réparation séparée du fichier de session peut réécrire les fichiers JSONL
mal formés en supprimant les lignes invalides avant le chargement de la session. Quand une réparation survient, le fichier
original est sauvegardé à côté du fichier de session.

Le périmètre inclut :

- Assainissement des identifiants d'appels d'outils
- Validation des entrées d'appels d'outils
- Réparation de l'appariement des résultats d'outils
- Validation / ordonnancement des tours
- Nettoyage des signatures de réflexion
- Assainissement des charges d'images
- Marquage de provenance des entrées utilisateur (pour les prompts routés inter-sessions)

Si vous avez besoin des détails de stockage des transcriptions, voir :

- [/reference/session-management-compaction](/reference/session-management-compaction)

---

## Où cela s'exécute

Toute l'hygiène des transcriptions est centralisée dans le runner intégré :

- Sélection de politique : `src/agents/transcript-policy.ts`
- Application assainissement/réparation : `sanitizeSessionHistory` dans `src/agents/pi-embedded-runner/google.ts`

La politique utilisé `provider`, `modelApi` et `modelId` pour décider quoi appliquer.

Séparément de l'hygiène des transcriptions, les fichiers de session sont réparés (si nécessaire) avant le chargement :

- `repairSessionFileIfNeeded` dans `src/agents/session-file-repair.ts`
- Appelé depuis `run/attempt.ts` et `compact.ts` (runner intégré)

---

## Règle globale : assainissement des images

Les charges d'images sont toujours assainies pour empêcher les rejets côté fournisseur dus aux limités
de taille (réduction/recompression des images base64 surdimensionnées).

Cela aide également à contrôler la pression de jetons liée aux images pour les modèles capables de vision.
Des dimensions maximales plus basses réduisent généralement l'utilisation de jetons ; des dimensions plus élevées préservent les détails.

Implémentation :

- `sanitizeSessionMessagesImages` dans `src/agents/pi-embedded-helpers/images.ts`
- `sanitizeContentBlocksImages` dans `src/agents/tool-images.ts`
- La dimension maximale d'image est configurable via `agents.defaults.imageMaxDimensionPx` (défaut : `1200`).

---

## Règle globale : appels d'outils mal formés

Les blocs d'appels d'outils assistant auxquels il manque à la fois `input` et `arguments` sont supprimés
avant la construction du contexte modèle. Cela empêche les rejets fournisseur dus aux appels d'outils
partiellement persistés (par exemple, après un échec de limitation de débit).

Implémentation :

- `sanitizeToolCallInputs` dans `src/agents/session-transcript-repair.ts`
- Appliqué dans `sanitizeSessionHistory` dans `src/agents/pi-embedded-runner/google.ts`

---

## Règle globale : provenance des entrées inter-sessions

Quand un agent envoie un prompt dans une autre session via `sessions_send` (incluant
les étapes de réponse/annonce agent-à-agent), OpenClaw persiste le tour utilisateur créé avec :

- `message.provenance.kind = "inter_session"`

Ces métadonnées sont écrites au moment de l'ajout à la transcription et ne changent pas le rôle
(`rôle: "user"` reste pour la compatibilité fournisseur). Les lecteurs de transcription peuvent utiliser
ceci pour éviter de traiter les prompts internes routés comme des instructions d'auteur utilisateur final.

Lors de la reconstruction du contexte, OpenClaw ajoute également un court marqueur `[Inter-session message]`
en mémoire à ces tours utilisateur afin que le modèle puisse les distinguer des
instructions d'utilisateur final externe.

---

## Matrice fournisseur (comportement actuel)

**OpenAI / OpenAI Codex**

- Assainissement des images uniquement.
- Suppression des signatures de raisonnement orphelines (éléments de raisonnement isolés sans bloc de contenu suivant) pour les transcriptions OpenAI Responses/Codex.
- Pas d'assainissement des identifiants d'appels d'outils.
- Pas de réparation d'appariement des résultats d'outils.
- Pas de validation ou réordonnancement des tours.
- Pas de résultats d'outils synthétiques.
- Pas de suppression des signatures de réflexion.

**Google (Generative AI / Gemini CLI / Antigravity)**

- Assainissement des identifiants d'appels d'outils : alphanumérique strict.
- Réparation d'appariement des résultats d'outils et résultats d'outils synthétiques.
- Validation des tours (alternance de tours style Gemini).
- Correction d'ordonnancement des tours Google (ajout d'un petit bootstrap utilisateur si l'historique commence par un assistant).
- Antigravity Claude : normalisation des signatures de réflexion ; suppression des blocs de réflexion non signés.

**Anthropic / Minimax (compatible Anthropic)**

- Réparation d'appariement des résultats d'outils et résultats d'outils synthétiques.
- Validation des tours (fusion des tours utilisateur consécutifs pour satisfaire l'alternance stricte).

**Mistral (incluant la détection basée sur l'identifiant du modèle)**

- Assainissement des identifiants d'appels d'outils : strict9 (alphanumérique de longueur 9).

**OpenRouter Gemini**

- Nettoyage des signatures de réflexion : suppression des valeurs `thought_signature` non-base64 (conservation du base64).

**Tout le reste**

- Assainissement des images uniquement.

---

## Comportement historique (pré-2026.1.22)

Avant la version 2026.1.22, OpenClaw appliquait plusieurs couches d'hygiène de transcription :

- Une **extension de nettoyage de transcription** s'exécutait à chaque construction de contexte et pouvait :
  - Réparer l'appariement utilisation/résultat d'outils.
  - Assainir les identifiants d'appels d'outils (incluant un mode non-strict qui préservait `_`/`-`).
- Le runner effectuait également un assainissement spécifique au fournisseur, ce qui dupliquait le travail.
- Des mutations supplémentaires survenaient en dehors de la politique fournisseur, incluant :
  - Suppression des balises `<final>` du texte assistant avant la persistance.
  - Suppression des tours d'erreur assistant vides.
  - Troncature du contenu assistant après les appels d'outils.

Cette complexité causait des régressions inter-fournisseurs (notamment l'appariement `openai-responses`
`call_id|fc_id`). Le nettoyage de la version 2026.1.22 a supprimé l'extension, centralisé
la logique dans le runner, et rendu OpenAI **sans modification** au-delà de l'assainissement des images.
