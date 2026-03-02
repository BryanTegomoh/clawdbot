---
summary: "Modes voice wake et push-to-talk plus details de routage dans l'application mac"
read_when:
  - Travail sur les chemins voice wake ou PTT
title: "Voice Wake"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/voicewake.md
  workflow: manual
---

# Voice Wake et Push-to-Talk

## Modes

- **Mode mot declencheur** (par défaut) : le reconnaisseur vocal toujours actif attend les jetons declencheurs (`swabbleTriggerWords`). A la détection, il démarre la capture, affiche la superposition avec le texte partiel, et envoie automatiquement après le silence.
- **Push-to-talk (maintien Option droite)** : maintenez la touche Option droite pour capturer immédiatement, sans declencheur nécessaire. La superposition apparaît pendant le maintien ; le relachement finalise et transmet après un court délai pour permettre l'edition du texte.

## Comportement du runtime (mot declencheur)

- Le reconnaisseur vocal reside dans `VoiceWakeRuntime`.
- Le declencheur ne se déclenche que lorsqu'il y a une **pause significative** entre le mot declencheur et le mot suivant (ecart d'environ 0,55s). La superposition/carillon peut démarrer a la pause même avant que la commande ne commence.
- Fenêtres de silence : 2,0s lorsque la parole est en cours, 5,0s si seul le declencheur a ete entendu.
- Arrêt force : 120s pour eviter les sessions emballees.
- Anti-rebond entre les sessions : 350ms.
- La superposition est pilotee via `VoiceWakeOverlayController` avec coloration texte committe/volatile.
- Après l'envoi, le reconnaisseur redémarre proprement pour écouter le prochain declencheur.

## Invariants du cycle de vie

- Si Voice Wake est activé et les permissions sont accordees, le reconnaisseur de mot declencheur devrait écouter (sauf pendant une capture push-to-talk explicite).
- La visibilité de la superposition (y compris la fermeture manuelle via le bouton X) ne doit jamais empecher le reconnaisseur de reprendre.

## Mode d'échec de superposition bloquée (précédent)

Auparavant, si la superposition restait bloquée visible et que vous la fermiez manuellement, Voice Wake pouvait sembler « mort » car la tentative de redémarrage du runtime pouvait être bloquée par la visibilité de la superposition et aucun redémarrage subsequent n'etait planifie.

Durcissement :

- Le redémarrage du runtime wake n'est plus bloque par la visibilité de la superposition.
- La fin de la fermeture de la superposition déclenche un `VoiceWakeRuntime.refresh(...)` via `VoiceSessionCoordinator`, donc la fermeture manuelle via X reprend toujours l'écoute.

## Specificites du push-to-talk

- La détection du raccourci utilisé un moniteur global `.flagsChanged` pour **Option droite** (`keyCode 61` + `.option`). Nous observons uniquement les événements (pas d'interception).
- Le pipeline de capture reside dans `VoicePushToTalk` : démarre la reconnaissance vocale immédiatement, diffuse les partiels vers la superposition, et appelle `VoiceWakeForwarder` au relachement.
- Lorsque le push-to-talk démarre, nous mettons en pause le runtime de mot declencheur pour eviter les captures audio concurrentes ; il redémarre automatiquement après le relachement.
- Permissions : nécessite Microphone + Reconnaissance vocale ; la détection des événements nécessite l'approbation Accessibilité/Surveillance des entrées.
- Claviers externes : certains peuvent ne pas exposer l'Option droite comme attendu, offrez un raccourci de repli si les utilisateurs signalent des manquements.

## Paramètres utilisateur

- Bascule **Voice Wake** : activé le runtime de mot declencheur.
- **Maintenir Cmd+Fn pour parler** : activé le moniteur push-to-talk. Désactive sur macOS < 26.
- Sélecteurs de langue et micro, compteur de niveau en direct, tableau de mots declencheurs, testeur (local uniquement ; ne transmet pas).
- Le sélecteur de micro preserve la dernière sélection si un appareil se déconnecté, affiche un indice de déconnexion, et se rabat temporairement sur le défaut système jusqu'a son retour.
- **Sons** : carillons a la détection du declencheur et a l'envoi ; par défaut le son système macOS « Glass ». Vous pouvez choisir n'importe quel fichier chargeable par `NSSound` (par exemple MP3/WAV/AIFF) pour chaque événement ou choisir **No Sound**.

## Comportement de transmission

- Lorsque Voice Wake est activé, les transcriptions sont transmises au gateway/agent actif (le même mode local vs distant utilisé par le reste de l'application mac).
- Les réponses sont livrees au **dernier fournisseur principal utilisé** (WhatsApp/Telegram/Discord/WebChat). Si la livraison échoue, l'erreur est journalisee et l'exécution est toujours visible via WebChat/logs de session.

## Charge utile de transmission

- `VoiceWakeForwarder.prefixedTranscript(_:)` ajouté l'indication machine avant l'envoi. Partage entre les chemins mot declencheur et push-to-talk.

## Vérification rapide

- Activez le push-to-talk, maintenez Cmd+Fn, parlez, relachez : la superposition devrait afficher les partiels puis envoyer.
- Pendant le maintien, les oreilles de la barre de menus devraient rester agrandies (utilisé `triggerVoiceEars(ttl:nil)`) ; elles retombent après le relachement.
