---
summary: "Persistance des permissions macOS (TCC) et exigences de signature"
read_when:
  - Debogage d'invites de permissions macOS manquantes ou bloquees
  - Empaquetage ou signature de l'application macOS
  - Modification des identifiants de bundle ou des chemins d'installation de l'application
title: "Permissions macOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/permissions.md
  workflow: manual
---

# Permissions macOS (TCC)

Les octrois de permissions macOS sont fragiles. TCC associé un octroi de permission a la
signature de code de l'application, a l'identifiant de bundle et au chemin sur le disque. Si l'un de ceux-ci change,
macOS traite l'application comme nouvelle et peut abandonner ou masquer les invites.

## Prerequis pour des permissions stables

- Même chemin : exécutez l'application depuis un emplacement fixe (pour OpenClaw, `dist/OpenClaw.app`).
- Même identifiant de bundle : changer le bundle ID créé une nouvelle identité de permission.
- Application signee : les builds non signes ou signes ad-hoc ne persistent pas les permissions.
- Signature coherente : utilisez un vrai certificat Apple Development ou Developer ID
  pour que la signature reste stable entre les recompilations.

Les signatures ad-hoc generent une nouvelle identité a chaque build. macOS oubliera les
octrois précédents, et les invites peuvent disparaitre complètement jusqu'a ce que les entrées obsoletes soient effacees.

## Checklist de récupération lorsque les invites disparaissent

1. Quittez l'application.
2. Supprimez l'entrée de l'application dans Préférences Système → Confidentialité et sécurité.
3. Relancez l'application depuis le même chemin et re-accordez les permissions.
4. Si l'invite n'apparaît toujours pas, reinitialisez les entrées TCC avec `tccutil` et réessayez.
5. Certaines permissions ne reapparaissent qu'après un redémarrage complet de macOS.

Exemples de réinitialisation (remplacez le bundle ID si nécessaire) :

```bash
sudo tccutil reset Accessibility ai.openclaw.mac
sudo tccutil reset ScreenCapture ai.openclaw.mac
sudo tccutil reset AppleEvents
```

## Permissions de fichiers et dossiers (Bureau/Documents/Telechargements)

macOS peut également restreindre le Bureau, Documents et Telechargements pour les processus terminal/arriere-plan. Si les lectures de fichiers ou les listings de répertoires se bloquent, accordez l'accès au même contexte de processus qui effectué les opérations sur les fichiers (par exemple Terminal/iTerm, application lancee par LaunchAgent, ou processus SSH).

Solution de contournement : deplacez les fichiers dans l'espace de travail OpenClaw (`~/.openclaw/workspace`) si vous voulez eviter les octrois par dossier.

Si vous testez les permissions, signez toujours avec un vrai certificat. Les builds
ad-hoc ne sont acceptables que pour des executions locales rapides ou les permissions n'ont pas d'importance.
