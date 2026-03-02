---
title: "AGENTS.md par défaut"
summary: "Instructions par défaut de l'agent OpenClaw et liste des skills pour la configuration d'assistant personnel"
read_when:
  - Démarrage d'une nouvelle session d'agent OpenClaw
  - Activation ou audit des skills par défaut
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: référence/AGENTS.default.md
  workflow: manual
---

# AGENTS.md — Assistant personnel OpenClaw (défaut)

## Première exécution (recommandé)

OpenClaw utilise un répertoire d'espace de travail dédié pour l'agent. Par défaut : `~/.openclaw/workspace` (configurable via `agents.defaults.workspace`).

1. Créez l'espace de travail (s'il n'existe pas déjà) :

```bash
mkdir -p ~/.openclaw/workspace
```

2. Copiez les modèles d'espace de travail par défaut dans l'espace de travail :

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3. Optionnel : si vous souhaitez la liste de skills d'assistant personnel, remplacez AGENTS.md par ce fichier :

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4. Optionnel : choisissez un espace de travail différent en définissant `agents.defaults.workspace` (supporte `~`) :

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

## Paramètres de sécurité par défaut

- Ne pas déverser les répertoires ou secrets dans le chat.
- Ne pas exécuter de commandes destructrices sauf demande explicite.
- Ne pas envoyer de réponses partielles/en streaming vers les surfaces de messagerie externes (uniquement les réponses finales).

## Démarrage de session (requis)

- Lire `SOUL.md`, `USER.md`, `memory.md`, et aujourd'hui+hier dans `memory/`.
- Le faire avant de répondre.

## Soul (requis)

- `SOUL.md` définit l'identité, le ton et les limités. Le maintenir à jour.
- Si vous modifiez `SOUL.md`, informez l'utilisateur.
- Vous êtes une instance fraîche à chaque session ; la continuité vit dans ces fichiers.

## Espaces partagés (recommandé)

- Vous n'êtes pas la voix de l'utilisateur ; soyez prudent dans les chats de groupe ou les canaux publics.
- Ne partagez pas de données privées, coordonnées ou notes internes.

## Système de mémoire (recommandé)

- Journal quotidien : `memory/YYYY-MM-DD.md` (créez `memory/` si nécessaire).
- Mémoire à long terme : `memory.md` pour les faits durables, préférences et décisions.
- Au démarrage de session, lire aujourd'hui + hier + `memory.md` si présent.
- Capturer : décisions, préférences, contraintes, points en suspens.
- Éviter les secrets sauf demande explicite.

## Outils et skills

- Les outils vivent dans les skills ; suivez le `SKILL.md` de chaque skill quand vous en avez besoin.
- Gardez les notes spécifiques à l'environnement dans `TOOLS.md` (Notes pour les Skills).

## Astuce de sauvegarde (recommandé)

Si vous traitez cet espace de travail comme la « mémoire » de Clawd, faites-en un dépôt git (idéalement privé) pour que `AGENTS.md` et vos fichiers mémoire soient sauvegardés.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Add Clawd workspace"
# Optionnel : ajouter un remote privé + push
```

## Ce que fait OpenClaw

- Exécute le Gateway WhatsApp + l'agent de codage Pi pour que l'assistant puisse lire/écrire les chats, récupérer du contexte et exécuter des skills via le Mac hôte.
- L'application macOS gère les permissions (enregistrement d'écran, notifications, microphone) et expose le CLI `openclaw` via son binaire intégré.
- Les chats directs se regroupent dans la session `main` de l'agent par défaut ; les groupes restent isolés comme `agent:<agentId>:<channel>:group:<id>` (salons/canaux : `agent:<agentId>:<channel>:channel:<id>`) ; les heartbeats maintiennent les tâches d'arrière-plan activés.

## Skills principaux (activer dans Paramètres → Skills)

- **mcporter** — Runtime/CLI de serveur d'outils pour gérer les backends de skills externes.
- **Peekaboo** — Captures d'écran macOS rapides avec analyse optionnelle de vision IA.
- **camsnap** — Capturer des images, clips ou alertes de mouvement depuis des caméras de sécurité RTSP/ONVIF.
- **oracle** — CLI d'agent prêt pour OpenAI avec relecture de session et contrôle de navigateur.
- **eightctl** — Contrôlez votre sommeil, depuis le terminal.
- **imsg** — Envoyer, lire, diffuser en continu iMessage et SMS.
- **wacli** — CLI WhatsApp : synchroniser, chercher, envoyer.
- **discord** — Actions Discord : réagir, stickers, sondages. Utilisez les cibles `user:<id>` ou `channel:<id>` (les identifiants numériques seuls sont ambigus).
- **gog** — CLI Google Suite : Gmail, Calendar, Drive, Contacts.
- **spotify-player** — Client Spotify en terminal pour chercher/mettre en file/contrôler la lecture.
- **sag** — Parole ElevenLabs avec UX style say de mac ; diffuse vers les haut-parleurs par défaut.
- **Sonos CLI** — Contrôler les enceintes Sonos (découverte/statut/lecture/volume/groupement) depuis des scripts.
- **blucli** — Jouer, grouper et automatiser les lecteurs BluOS depuis des scripts.
- **OpenHue CLI** — Contrôle d'éclairage Philips Hue pour les scènes et automatisations.
- **OpenAI Whisper** — Reconnaissance vocale locale pour la dictée rapide et les transcriptions de messages vocaux.
- **Gemini CLI** — Modèles Google Gemini depuis le terminal pour du Q&R rapide.
- **agent-tools** — Boîte à outils utilitaire pour les automatisations et scripts d'aide.

## Notes d'utilisation

- Préférez le CLI `openclaw` pour le scripting ; l'application Mac gère les permissions.
- Lancez les installations depuis l'onglet Skills ; il masque le bouton si un binaire est déjà présent.
- Gardez les heartbeats activés pour que l'assistant puisse planifier des rappels, surveiller les boîtes de réception et déclencher des captures de caméra.
- Le Canvas UI s'exécute en plein écran avec des overlays natifs. Évitez de placer des contrôles critiques dans les coins haut-gauche/haut-droit/bords inférieurs ; ajoutez des marges explicites dans la mise en page et ne vous fiez pas aux insets de zone sûre.
- Pour la vérification par navigateur, utilisez `openclaw browser` (tabs/status/screenshot) avec le profil Chrome géré par OpenClaw.
- Pour l'inspection DOM, utilisez `openclaw browser eval|query|dom|snapshot` (et `--json`/`--out` quand vous avez besoin d'une sortie machine).
- Pour les interactions, utilisez `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type nécessitent des refs de snapshot ; utilisez `evaluate` pour les sélecteurs CSS).
