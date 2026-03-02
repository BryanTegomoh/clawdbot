---
summary: "Étapes de vérification de santé pour la connectivité des canaux"
read_when:
  - Diagnostic de la santé du canal WhatsApp
title: "Vérifications de santé"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/health.md
  workflow: manual
---

# Vérifications de santé (CLI)

Guide rapide pour vérifier la connectivité des canaux sans deviner.

## Vérifications rapides

- `openclaw status` : résumé local : accessibilité/mode du gateway, indice de mise à jour, âge de l'authentification du canal lié, sessions + activité récente.
- `openclaw status --all` : diagnostic local complet (lecture seule, couleur, sûr à coller pour le débogage).
- `openclaw status --deep` : sonde aussi le Gateway en cours d'exécution (sondes par canal lorsque supporté).
- `openclaw health --json` : demande au Gateway en cours d'exécution un instantané de santé complet (WS uniquement ; pas de socket Baileys direct).
- Envoyez `/status` comme message autonome dans WhatsApp/WebChat pour obtenir une réponse de statut sans invoquer l'agent.
- Logs : suivez `/tmp/openclaw/openclaw-*.log` et filtrez pour `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

## Diagnostics approfondis

- Identifiants sur disque : `ls -l ~/.openclaw/credentials/whatsapp/<accountId>/creds.json` (mtime doit être récent).
- Stockage de session : `ls -l ~/.openclaw/agents/<agentId>/sessions/sessions.json` (le chemin peut être remplacé dans la configuration). Le nombre et les destinataires récents sont exposés via `status`.
- Flux de reconnexion : `openclaw channels logout && openclaw channels login --verbose` lorsque les codes de statut 409-515 ou `loggedOut` apparaissent dans les logs. (Note : le flux de connexion QR redémarre automatiquement une fois pour le statut 515 après l'appairage.)

## Quand quelque chose échoue

- `logged out` ou statut 409-515 → reconnectez avec `openclaw channels logout` puis `openclaw channels login`.
- Gateway inaccessible → démarrez-le : `openclaw gateway --port 18789` (utilisez `--force` si le port est occupé).
- Pas de messages entrants → confirmez que le téléphone lie est en ligne et que l'expéditeur est autorisé (`channels.whatsapp.allowFrom`) ; pour les discussions de groupe, assurez-vous que la liste autorisée + les règles de mention correspondent (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## Commande « health » dédiée

`openclaw health --json` demande au Gateway en cours d'exécution son instantané de santé (pas de sockets de canal directs depuis le CLI). Il signale les identifiants liés/l'âge de l'authentification lorsque disponible, les résumés de sonde par canal, le résumé du stockage de session et une durée de sonde. Il sort avec un code non nul si le Gateway est inaccessible ou si la sonde échoue/timeout. Utilisez `--timeout <ms>` pour remplacer le délai par défaut de 10s.
