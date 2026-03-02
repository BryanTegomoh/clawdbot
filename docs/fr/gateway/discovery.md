---
summary: "Découverte des nœuds et transports (Bonjour, Tailscale, SSH) pour trouver le gateway"
read_when:
  - Implémentation ou modification de la découverte/annonce Bonjour
  - Ajustement des modes de connexion distante (direct vs SSH)
  - Conception de la découverte de nœuds + appairage pour les nœuds distants
title: "Découverte et transports"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/discovery.md
  workflow: manual
---

# Découverte et transports

OpenClaw a deux problèmes distincts qui se ressemblent en surface :

1. **Contrôle à distance de l'opérateur** : l'application barre de menu macOS contrôlant un gateway s'exécutant ailleurs.
2. **Appairage de nœuds** : iOS/Android (et futurs nœuds) trouvant un gateway et s'appairant de manière sécurisée.

L'objectif de conception est de garder toute la découverte/annonce réseau dans le **Node Gateway** (`openclaw gateway`) et de garder les clients (application mac, iOS) comme consommateurs.

## Termes

- **Gateway** : un processus gateway unique longue durée qui possède l'état (sessions, appairage, registre de nœuds) et exécute les canaux. La plupart des installations utilisent un seul gateway par hôte ; des installations multi-gateway isolées sont possibles.
- **Gateway WS (plan de contrôle)** : le point d'accès WebSocket sur `127.0.0.1:18789` par défaut ; peut être lié au LAN/tailnet via `gateway.bind`.
- **Transport WS direct** : un point d'accès Gateway WS exposé sur le LAN/tailnet (sans SSH).
- **Transport SSH (repli)** : contrôle à distance par transfert de `127.0.0.1:18789` via SSH.
- **Bridge TCP legacy (obsolète/supprimé)** : ancien transport de nœuds (voir [Protocole Bridge](/gateway/bridge-protocol)) ; n'est plus annoncé pour la découverte.

Détails du protocole :

- [Protocole Gateway](/gateway/protocol)
- [Protocole Bridge (legacy)](/gateway/bridge-protocol)

## Pourquoi nous conservons les deux : « direct » et SSH

- **WS direct** offre la meilleure UX sur le même réseau et au sein d'un tailnet :
  - découverte automatique sur le LAN via Bonjour
  - jetons d'appairage + ACLs gérés par le gateway
  - pas d'accès shell requis ; la surface du protocole peut rester étroite et auditable
- **SSH** reste le repli universel :
  - fonctionne partout où vous avez un accès SSH (même sur des réseaux non liés)
  - survit aux problèmes multicast/mDNS
  - ne nécessite aucun nouveau port entrant en dehors de SSH

## Entrées de découverte (comment les clients localisent le gateway)

### 1) Bonjour / mDNS (LAN uniquement)

Bonjour fonctionne au mieux et ne traverse pas les réseaux. Il n'est utilisé que pour la commodité sur le « même LAN ».

Direction cible :

- Le **gateway** annonce son point d'accès WS via Bonjour.
- Les clients parcourent et affichent une liste « choisir un gateway », puis stockent le point d'accès choisi.

Dépannage et détails des balises : [Bonjour](/gateway/bonjour).

#### Détails des balises de service

- Types de service :
  - `_openclaw-gw._tcp` (balise de transport gateway)
- Clés TXT (non secrètes) :
  - `rôle=gateway`
  - `lanHost=<hostname>.local`
  - `sshPort=22` (ou ce qui est annoncé)
  - `gatewayPort=18789` (Gateway WS + HTTP)
  - `gatewayTls=1` (uniquement lorsque TLS est activé)
  - `gatewayTlsSha256=<sha256>` (uniquement lorsque TLS est activé et l'empreinte disponible)
  - `canvasPort=<port>` (port de l'hôte canvas ; actuellement identique à `gatewayPort` lorsque l'hôte canvas est activé)
  - `cliPath=<path>` (optionnel ; chemin absolu vers un point d'entrée ou binaire `openclaw` exécutable)
  - `tailnetDns=<magicdns>` (indice optionnel ; auto-détecté lorsque Tailscale est disponible)

Notes de sécurité :

- Les enregistrements TXT Bonjour/mDNS sont **non authentifiés**. Les clients doivent traiter les valeurs TXT comme de simples indices d'interface.
- Le routage (hôte/port) doit préférer le **point d'accès de service résout** (SRV + A/AAAA) aux valeurs TXT `lanHost`, `tailnetDns` ou `gatewayPort`.
- L'épinglage TLS ne doit jamais permettre à un `gatewayTlsSha256` annoncé de remplacer un épinglage précédemment stocké.
- Les nœuds iOS/Android doivent traiter les connexions directes basées sur la découverte comme **TLS uniquement** et exiger une confirmation explicite de l'utilisateur « faire confiance à cette empreinte » avant de stocker un épinglage initial (vérification hors bande).

Désactivation/remplacement :

- `OPENCLAW_DISABLE_BONJOUR=1` désactive l'annonce.
- `gateway.bind` dans `~/.openclaw/openclaw.json` contrôle le mode de liaison du Gateway.
- `OPENCLAW_SSH_PORT` remplace le port SSH annoncé dans les TXT (par défaut 22).
- `OPENCLAW_TAILNET_DNS` publie un indice `tailnetDns` (MagicDNS).
- `OPENCLAW_CLI_PATH` remplace le chemin CLI annoncé.

### 2) Tailnet (inter-réseau)

Pour les configurations de type Londres/Vienne, Bonjour ne fonctionnera pas. La cible « directe » recommandée est :

- Le nom MagicDNS Tailscale (préféré) ou une IP tailnet stable.

Si le gateway peut détecter qu'il s'exécute sous Tailscale, il publie `tailnetDns` comme indice optionnel pour les clients (y compris les balises étendues).

### 3) Manuel / cible SSH

Lorsqu'il n'y a pas de route directe (ou que le direct est désactivé), les clients peuvent toujours se connecter via SSH en transférant le port gateway en loopback.

Voir [Accès distant](/gateway/remote).

## Sélection de transport (politique client)

Comportement client recommandé :

1. Si un point d'accès direct appairé est configuré et accessible, l'utiliser.
2. Sinon, si Bonjour trouve un gateway sur le LAN, proposer un choix « Utiliser ce gateway » en un tap et le sauvegarder comme point d'accès direct.
3. Sinon, si un DNS/IP tailnet est configuré, essayer le direct.
4. Sinon, revenir au SSH.

## Appairage + authentification (transport direct)

Le gateway est la source de vérité pour l'admission des nœuds/clients.

- Les requêtes d'appairage sont créées/approuvées/rejetées dans le gateway (voir [Appairage Gateway](/gateway/pairing)).
- Le gateway applique :
  - l'authentification (jeton / paire de clés)
  - les scopes/ACLs (le gateway n'est pas un proxy brut vers chaque méthode)
  - les limités de débit

## Responsabilités par composant

- **Gateway** : annonce les balises de découverte, gère les décisions d'appairage et héberge le point d'accès WS.
- **Application macOS** : aide à choisir un gateway, affiche les invites d'appairage et utilise SSH uniquement comme repli.
- **Nœuds iOS/Android** : parcourent Bonjour par commodité et se connectent au Gateway WS appairé.
