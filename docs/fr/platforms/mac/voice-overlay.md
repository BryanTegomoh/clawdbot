---
summary: "Cycle de vie de la superposition vocale lorsque le mot declencheur et le push-to-talk se chevauchent"
read_when:
  - Ajustement du comportement de la superposition vocale
title: "Superposition vocale"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/voice-overlay.md
  workflow: manual
---

# Cycle de vie de la superposition vocale (macOS)

Public : contributeurs de l'application macOS. Objectif : maintenir la superposition vocale previsible lorsque le mot declencheur et le push-to-talk se chevauchent.

## Intention actuelle

- Si la superposition est déjà visible depuis le mot declencheur et que l'utilisateur appuie sur le raccourci, la session raccourci _adopte_ le texte existant au lieu de le réinitialiser. La superposition reste affichee tant que le raccourci est maintenu. Lorsque l'utilisateur relache : envoie s'il y a du texte après trim, sinon ferme.
- Le mot declencheur seul envoie toujours automatiquement au silence ; le push-to-talk envoie immédiatement au relachement.

## Implemente (9 dec. 2025)

- Les sessions de superposition portent maintenant un jeton par capture (mot declencheur ou push-to-talk). Les mises a jour partielles/finales/envoi/fermeture/niveau sont ignorées lorsque le jeton ne correspond pas, evitant les callbacks obsoletes.
- Le push-to-talk adopte tout texte de superposition visible comme préfixe (appuyer sur le raccourci alors que la superposition wake est affichee conserve le texte et ajoute la nouvelle parole). Il attend jusqu'a 1,5s pour une transcription finale avant de se rabattre sur le texte actuel.
- Les logs de carillon/superposition sont émis au niveau `info` dans les categories `voicewake.overlay`, `voicewake.ptt`, et `voicewake.chime` (debut de session, partiel, final, envoi, fermeture, raison du carillon).

## Prochaines étapes

1. **VoiceSessionCoordinator (acteur)**
   - Possède exactement une `VoiceSession` a la fois.
   - API (basee sur les jetons) : `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Ignoré les callbacks portant des jetons obsoletes (empeche les anciens reconnaisseurs de rouvrir la superposition).
2. **VoiceSession (modèle)**
   - Champs : `token`, `source` (wakeWord|pushToTalk), texte committe/volatile, drapeaux de carillon, minuteries (envoi automatique, inactivite), `overlayMode` (display|editing|sending), délai de refroidissement.
3. **Liaison de la superposition**
   - `VoiceSessionPublisher` (`ObservableObject`) reflete la session active dans SwiftUI.
   - `VoiceWakeOverlayView` affiche uniquement via le publisher ; elle ne modifie jamais directement les singletons globaux.
   - Les actions utilisateur de la superposition (`sendNow`, `dismiss`, `edit`) rappellent le coordinateur avec le jeton de session.
4. **Chemin d'envoi unifie**
   - Sur `endCapture` : si le texte trim est vide → fermer ; sinon `performSend(session:)` (joue le carillon d'envoi une fois, transmet, ferme).
   - Push-to-talk : pas de délai ; mot declencheur : délai optionnel pour l'envoi automatique.
   - Appliquer un court refroidissement au runtime wake après la fin du push-to-talk pour que le mot declencheur ne se redeclenche pas immédiatement.
5. **Journalisation**
   - Le coordinateur émet des logs `.info` dans le sous-système `ai.openclaw`, categories `voicewake.overlay` et `voicewake.chime`.
   - Événements clés : `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

## Checklist de debogage

- Diffusez les logs tout en reproduisant une superposition bloquée :

  ```bash
  sudo log stream --predicate 'subsystem == "ai.openclaw" AND category CONTAINS "voicewake"' --level info --style compact
  ```

- Verifiez qu'il n'y a qu'un seul jeton de session actif ; les callbacks obsoletes devraient être ignorés par le coordinateur.
- Assurez-vous que le relachement du push-to-talk appelle toujours `endCapture` avec le jeton actif ; si le texte est vide, attendez-vous a `dismiss` sans carillon ni envoi.

## Étapes de migration (suggerees)

1. Ajoutez `VoiceSessionCoordinator`, `VoiceSession` et `VoiceSessionPublisher`.
2. Refactorisez `VoiceWakeRuntime` pour créer/mettre a jour/terminer des sessions au lieu de toucher directement `VoiceWakeOverlayController`.
3. Refactorisez `VoicePushToTalk` pour adopter les sessions existantes et appeler `endCapture` au relachement ; appliquez le refroidissement du runtime.
4. Connectez `VoiceWakeOverlayController` au publisher ; supprimez les appels directs depuis le runtime/PTT.
5. Ajoutez des tests d'intégration pour l'adoption de session, le refroidissement et la fermeture avec texte vide.
