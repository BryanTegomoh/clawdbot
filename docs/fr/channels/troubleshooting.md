---
summary: "Dépannage rapide au niveau des canaux avec signatures de pannes et correctifs par canal"
read_when:
  - Le transport du canal indique connecté mais les réponses échouent
  - Vous avez besoin de vérifications spécifiques au canal avant la documentation approfondie des fournisseurs
title: "Dépannage des canaux"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: channels/troubleshooting.md
  workflow: manual
---

# Dépannage des canaux

Utilisez cette page lorsqu'un canal se connecte mais que le comportement est incorrect.

## Echelle de commandes

Exécutez ces commandes dans l'ordre en premier :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

État de base sain :

- `Runtime: running`
- `RPC probe: ok`
- La sonde du canal affiche connecté/prêt

## WhatsApp

### Signatures de pannes WhatsApp

| Symptôme                          | Vérification rapide                                 | Correctif                                                       |
| --------------------------------- | --------------------------------------------------- | --------------------------------------------------------------- |
| Connecté mais pas de réponses en message privé | `openclaw pairing list whatsapp`                    | Approuver l'expéditeur ou changer la politique/liste autorisée. |
| Messages de groupe ignorés        | Vérifier `requireMention` + modèles de mention dans la config | Mentionner le bot ou assouplir la politique de mention pour ce groupe. |
| Déconnexion aleatoire/boucles de reconnexion | `openclaw channels status --probe` + logs           | Se reconnecter et vérifier que le répertoire des identifiants est sain. |

Dépannage complet : [/channels/whatsapp#troubleshooting-quick](/channels/whatsapp#troubleshooting-quick)

## Telegram

### Signatures de pannes Telegram

| Symptôme                            | Vérification rapide                             | Correctif                                                                       |
| ----------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| `/start` mais pas de flux de réponse utilisable | `openclaw pairing list telegram`                | Approuver l'appairage ou changer la politique de messages privés.               |
| Bot en ligne mais le groupe reste silencieux | Vérifier l'exigence de mention et le mode de confidentialité du bot | Désactiver le mode de confidentialité pour la visibilité du groupe ou mentionner le bot. |
| Échecs d'envoi avec erreurs réseau  | Inspecter les logs pour les échecs d'appel API Telegram | Corriger le routage DNS/IPv6/proxy vers `api.telegram.org`.                    |
| Mise a jour et la liste autorisée vous bloque | `openclaw security audit` et listes autorisées de config | Exécuter `openclaw doctor --fix` ou remplacer `@username` par des ID numériques d'expéditeur. |

Dépannage complet : [/channels/telegram#troubleshooting](/channels/telegram#troubleshooting)

## Discord

### Signatures de pannes Discord

| Symptôme                          | Vérification rapide                 | Correctif                                                         |
| --------------------------------- | ----------------------------------- | ----------------------------------------------------------------- |
| Bot en ligne mais pas de réponses du serveur | `openclaw channels status --probe`  | Autoriser le serveur/canal et vérifier l'intent de contenu de message. |
| Messages de groupe ignorés        | Vérifier les logs pour les rejets de mention | Mentionner le bot ou définir `requireMention: false` pour le serveur/canal. |
| Réponses aux messages privés manquantes | `openclaw pairing list discord`     | Approuver l'appairage ou ajuster la politique de messages privés. |

Dépannage complet : [/channels/discord#troubleshooting](/channels/discord#troubleshooting)

## Slack

### Signatures de pannes Slack

| Symptôme                                 | Vérification rapide                       | Correctif                                             |
| ---------------------------------------- | ----------------------------------------- | ----------------------------------------------------- |
| Mode socket connecte mais pas de réponses | `openclaw channels status --probe`        | Vérifier le jeton d'app + jeton de bot et les scopes requis. |
| Messages privés bloqués                  | `openclaw pairing list slack`             | Approuver l'appairage ou assouplir la politique de messages privés. |
| Message de canal ignoré                  | Vérifier `groupPolicy` et la liste autorisée du canal | Autoriser le canal ou passer la politique a `open`.   |

Dépannage complet : [/channels/slack#troubleshooting](/channels/slack#troubleshooting)

## iMessage et BlueBubbles

### Signatures de pannes iMessage et BlueBubbles

| Symptôme                           | Vérification rapide                                                     | Correctif                                                 |
| ---------------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------- |
| Aucun événement entrant            | Vérifier l'accessibilité du webhook/serveur et les permissions de l'app | Corriger l'URL du webhook ou l'état du serveur BlueBubbles. |
| Peut envoyer mais pas recevoir sur macOS | Vérifier les permissions de confidentialité macOS pour l'automatisation Messages | Re-accorder les permissions TCC et redémarrer le processus du canal. |
| Expéditeur de message privé bloque | `openclaw pairing list imessage` ou `openclaw pairing list bluebubbles` | Approuver l'appairage ou mettre a jour la liste autorisée. |

Dépannage complet :

- [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](/channels/imessage#troubleshooting-macos-privacy-and-security-tcc)
- [/channels/bluebubbles#troubleshooting](/channels/bluebubbles#troubleshooting)

## Signal

### Signatures de pannes Signal

| Symptôme                          | Vérification rapide                        | Correctif                                                        |
| --------------------------------- | ------------------------------------------ | ---------------------------------------------------------------- |
| Daemon accessible mais bot silencieux | `openclaw channels status --probe`         | Vérifier l'URL/le compte du daemon `signal-cli` et le mode de reception. |
| Message privé bloque              | `openclaw pairing list signal`             | Approuver l'expéditeur ou ajuster la politique de messages privés. |
| Les réponses de groupe ne se declenchent pas | Vérifier la liste autorisée du groupe et les modèles de mention | Ajouter l'expéditeur/le groupe ou assouplir le filtrage. |

Dépannage complet : [/channels/signal#troubleshooting](/channels/signal#troubleshooting)

## Matrix

### Signatures de pannes Matrix

| Symptôme                              | Vérification rapide                          | Correctif                                           |
| ------------------------------------- | -------------------------------------------- | --------------------------------------------------- |
| Connecté mais ignoré les messages du salon | `openclaw channels status --probe`           | Vérifier `groupPolicy` et la liste autorisée du salon. |
| Les messages privés ne sont pas traites | `openclaw pairing list matrix`               | Approuver l'expéditeur ou ajuster la politique de messages privés. |
| Les salons chiffrés échouent          | Vérifier le module crypto et les paramètres de chiffrement | Activer le support du chiffrement et rejoindre/synchroniser le salon. |

Dépannage complet : [/channels/matrix#troubleshooting](/channels/matrix#troubleshooting)
