---
summary: "Flux de l'application macOS pour contrôler un gateway OpenClaw distant via SSH"
read_when:
  - Configuration ou debogage du contrôle mac distant
title: "Contrôle distant"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/remote.md
  workflow: manual
---

# OpenClaw distant (macOS ⇄ hôte distant)

Ce flux permet a l'application macOS d'agir comme une telecommande complète pour un gateway OpenClaw s'executant sur un autre hôte (bureau/serveur). C'est la fonctionnalité **Remote over SSH** (exécution distante) de l'application. Toutes les fonctionnalités (vérifications de sante, transfert de Voice Wake et Web Chat) reutilisent la même configuration SSH distante depuis _Parametres → General_.

## Modes

- **Local (ce Mac)** : tout s'exécute sur l'ordinateur portable. Pas de SSH implique.
- **Remote over SSH (par défaut)** : les commandes OpenClaw sont executees sur l'hôte distant. L'application mac ouvre une connexion SSH avec `-o BatchMode` plus votre identité/clé choisie et un transfert de port local.
- **Remote direct (ws/wss)** : pas de tunnel SSH. L'application mac se connecte directement a l'URL du gateway (par exemple, via Tailscale Serve ou un reverse proxy HTTPS public).

## Transports distants

Le mode distant supporte deux transports :

- **Tunnel SSH** (par défaut) : utilisé `ssh -N -L ...` pour rediriger le port du gateway vers localhost. Le gateway verra l'IP du nœud comme `127.0.0.1` car le tunnel est en loopback.
- **Direct (ws/wss)** : se connecte directement a l'URL du gateway. Le gateway voit l'IP reelle du client.

## Prerequis sur l'hôte distant

1. Installez Node + pnpm et compilez/installez le CLI OpenClaw (`pnpm install && pnpm build && pnpm link --global`).
2. Assurez-vous qu'`openclaw` est dans le PATH pour les shells non interactifs (lien symbolique dans `/usr/local/bin` ou `/opt/homebrew/bin` si nécessaire).
3. Ouvrez SSH avec authentification par clé. Nous recommandons les IPs **Tailscale** pour une accessibilité stable hors LAN.

## Configuration de l'application macOS

1. Ouvrez _Parametres → General_.
2. Sous **OpenClaw runs**, choisissez **Remote over SSH** et définissez :
   - **Transport** : **SSH tunnel** ou **Direct (ws/wss)**.
   - **Cible SSH** : `user@host` (optionnel `:port`).
     - Si le gateway est sur le même LAN et annonce Bonjour, selectionnez-le dans la liste decouverte pour remplir automatiquement ce champ.
   - **URL du Gateway** (Direct uniquement) : `wss://gateway.example.ts.net` (ou `ws://...` pour local/LAN).
   - **Fichier d'identité** (avance) : chemin vers votre clé.
   - **Racine du projet** (avance) : chemin du checkout distant utilisé pour les commandes.
   - **Chemin du CLI** (avance) : chemin optionnel vers un point d'entrée/binaire `openclaw` executable (rempli automatiquement lorsqu'il est annonce).
3. Appuyez sur **Test remote**. Le succes indique que `openclaw status --json` distant s'exécute correctement. Les échecs signifient généralement des problèmes de PATH/CLI ; le code de sortie 127 signifie que le CLI n'est pas trouve a distance.
4. Les vérifications de sante et Web Chat s'executeront désormais automatiquement via ce tunnel SSH.

## Web Chat

- **Tunnel SSH** : Web Chat se connecte au gateway via le port WebSocket de contrôle redirige (par défaut 18789).
- **Direct (ws/wss)** : Web Chat se connecte directement a l'URL du gateway configurée.
- Il n'y a plus de serveur HTTP WebChat séparé.

## Permissions

- L'hôte distant a besoin des mêmes approbations TCC que le local (Automatisation, Accessibilité, Enregistrement d'ecran, Microphone, Reconnaissance vocale, Notifications). Lancez l'intégration sur cette machine pour les accorder une fois.
- Les nœuds annoncent leur état de permissions via `node.list` / `node.describe` pour que les agents sachent ce qui est disponible.

## Notes de sécurité

- Préférez les liaisons loopback sur l'hôte distant et connectez-vous via SSH ou Tailscale.
- Le tunnel SSH utilisé la vérification stricte de la clé d'hôte ; faites confiance a la clé d'hôte d'abord pour qu'elle existe dans `~/.ssh/known_hosts`.
- Si vous liez le Gateway a une interface non-loopback, exigez l'authentification par jeton/mot de passe.
- Voir [Sécurité](/gateway/security) et [Tailscale](/gateway/tailscale).

## Flux de connexion WhatsApp (distant)

- Exécutez `openclaw channels login --verbose` **sur l'hôte distant**. Scannez le QR avec WhatsApp sur votre téléphone.
- Relancez la connexion sur cet hôte si l'authentification expire. La vérification de sante signalera les problèmes de liaison.

## Dépannage

- **exit 127 / not found** : `openclaw` n'est pas dans le PATH pour les shells non-login. Ajoutez-le a `/etc/paths`, votre shell rc, ou creez un lien symbolique dans `/usr/local/bin`/`/opt/homebrew/bin`.
- **Échec de la sonde de sante** : verifiez l'accessibilité SSH, le PATH, et que Baileys est connecté (`openclaw status --json`).
- **Web Chat bloque** : confirmez que le gateway est en cours d'exécution sur l'hôte distant et que le port redirige correspond au port WS du gateway ; l'UI nécessite une connexion WS saine.
- **L'IP du nœud affiche 127.0.0.1** : attendu avec le tunnel SSH. Passez le **Transport** a **Direct (ws/wss)** si vous voulez que le gateway voie l'IP reelle du client.
- **Voice Wake** : les phrases de déclenchement sont transmises automatiquement en mode distant ; aucun transfert séparé n'est nécessaire.

## Sons de notification

Choisissez des sons par notification depuis les scripts avec `openclaw` et `node.invoke`, par exemple :

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

Il n'y a plus de bascule « son par défaut » global dans l'application ; les appelants choisissent un son (ou aucun) par requête.
