---
summary: "Découverte Bonjour/mDNS + débogage (balises Gateway, clients et modes de défaillance courants)"
read_when:
  - Débogage des problèmes de découverte Bonjour sur macOS/iOS
  - Modification des types de service mDNS, enregistrements TXT ou UX de découverte
title: "Découverte Bonjour"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: gateway/bonjour.md
  workflow: manual
---

# Découverte Bonjour / mDNS

OpenClaw utilise Bonjour (mDNS / DNS-SD) comme **commodité réseau local uniquement** pour découvrir un Gateway actif (point d'accès WebSocket). C'est un fonctionnement au mieux qui **ne remplace pas** la connectivité SSH ou Tailnet.

## Bonjour étendu (Unicast DNS-SD) via Tailscale

Si le nœud et le gateway sont sur des réseaux différents, le mDNS multicast ne traversera pas la frontière. Vous pouvez conserver la même UX de découverte en passant au **DNS-SD unicast** (« Wide-Area Bonjour ») via Tailscale.

Étapes générales :

1. Exécuter un serveur DNS sur l'hôte du gateway (accessible via Tailnet).
2. Publier des enregistrements DNS-SD pour `_openclaw-gw._tcp` sous une zone dédiée (exemple : `openclaw.internal.`).
3. Configurer le **split DNS** Tailscale pour que votre domaine choisi soit résolu via ce serveur DNS pour les clients (y compris iOS).

OpenClaw prend en charge tout domaine de découverte ; `openclaw.internal.` n'est qu'un exemple. Les nœuds iOS/Android parcourent à la fois `local.` et votre domaine étendu configuré.

### Configuration Gateway (recommandée)

```json5
{
  gateway: { bind: "tailnet" }, // tailnet uniquement (recommandé)
  discovery: { wideArea: { enabled: true } }, // active la publication DNS-SD étendue
}
```

### Configuration initiale du serveur DNS (hôte du gateway)

```bash
openclaw dns setup --apply
```

Cela installe CoreDNS et le configuré pour :

- écouter sur le port 53 uniquement sur les interfaces Tailscale du gateway
- servir votre domaine choisi (exemple : `openclaw.internal.`) depuis `~/.openclaw/dns/<domain>.db`

Validez depuis une machine connectée au tailnet :

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Paramètres DNS Tailscale

Dans la console d'administration Tailscale :

- Ajoutez un serveur de noms pointant vers l'IP tailnet du gateway (UDP/TCP 53).
- Ajoutez un split DNS pour que votre domaine de découverte utilisé ce serveur de noms.

Une fois que les clients acceptent le DNS tailnet, les nœuds iOS peuvent parcourir `_openclaw-gw._tcp` dans votre domaine de découverte sans multicast.

### Sécurité du listener Gateway (recommandée)

Le port WS du Gateway (par défaut `18789`) se lie au loopback par défaut. Pour l'accès LAN/tailnet, liez explicitement et gardez l'authentification activée.

Pour les configurations tailnet uniquement :

- Définissez `gateway.bind: "tailnet"` dans `~/.openclaw/openclaw.json`.
- Redémarrez le Gateway (ou redémarrez l'application barre de menu macOS).

## Ce qui est annoncé

Seul le Gateway annonce `_openclaw-gw._tcp`.

## Types de service

- `_openclaw-gw._tcp` : balise de transport gateway (utilisée par les nœuds macOS/iOS/Android).

## Clés TXT (indices non secrets)

Le Gateway annonce de petits indices non secrets pour faciliter les flux d'interface :

- `rôle=gateway`
- `displayName=<nom convivial>`
- `lanHost=<hostname>.local`
- `gatewayPort=<port>` (Gateway WS + HTTP)
- `gatewayTls=1` (uniquement lorsque TLS est activé)
- `gatewayTlsSha256=<sha256>` (uniquement lorsque TLS est activé et l'empreinte disponible)
- `canvasPort=<port>` (uniquement lorsque l'hôte canvas est activé ; actuellement identique à `gatewayPort`)
- `sshPort=<port>` (par défaut 22 quand non remplace)
- `transport=gateway`
- `cliPath=<chemin>` (optionnel ; chemin absolu vers un point d'entrée `openclaw` exécutable)
- `tailnetDns=<magicdns>` (indice optionnel lorsque Tailnet est disponible)

Notes de sécurité :

- Les enregistrements TXT Bonjour/mDNS sont **non authentifiés**. Les clients ne doivent pas traiter les TXT comme un routage faisant autorité.
- Les clients doivent router en utilisant le point d'accès de service résout (SRV + A/AAAA). Traitez `lanHost`, `tailnetDns`, `gatewayPort` et `gatewayTlsSha256` comme de simples indices.
- L'épinglage TLS ne doit jamais permettre à un `gatewayTlsSha256` annoncé de remplacer un épinglage précédemment stocké.
- Les nœuds iOS/Android doivent traiter les connexions directes basées sur la découverte comme **TLS uniquement** et exiger une confirmation explicite de l'utilisateur avant de faire confiance à une empreinte pour la première fois.

## Débogage sur macOS

Outils intégrés utiles :

- Parcourir les instances :

  ```bash
  dns-sd -B _openclaw-gw._tcp local.
  ```

- Résoudre une instance (remplacez `<instance>`) :

  ```bash
  dns-sd -L "<instance>" _openclaw-gw._tcp local.
  ```

Si le parcours fonctionne mais la résolution échoue, vous rencontrez généralement un problème de politique LAN ou de résolveur mDNS.

## Débogage dans les logs du Gateway

Le Gateway écrit un fichier de log rotatif (affiché au démarrage comme `gateway log file: ...`). Recherchez les lignes `bonjour:`, en particulier :

- `bonjour: advertise failed ...`
- `bonjour: ... name conflict resolved` / `hostname conflict resolved`
- `bonjour: watchdog detected non-announced service ...`

## Débogage sur le nœud iOS

Le nœud iOS utilisé `NWBrowser` pour découvrir `_openclaw-gw._tcp`.

Pour capturer les logs :

- Réglages → Gateway → Avancé → **Discovery Debug Logs**
- Réglages → Gateway → Avancé → **Discovery Logs** → reproduire → **Copier**

Le log inclut les transitions d'état du navigateur et les changements de l'ensemble de résultats.

## Modes de défaillance courants

- **Bonjour ne traverse pas les réseaux** : utilisez Tailnet ou SSH.
- **Multicast bloqué** : certains réseaux Wi-Fi désactivent le mDNS.
- **Veille / rotation d'interface** : macOS peut temporairement perdre les résultats mDNS ; réessayez.
- **Le parcours fonctionne mais la résolution échoue** : gardez des noms de machine simples (évitez les emojis ou la ponctuation), puis redémarrez le Gateway. Le nom d'instance de service dérive du nom d'hôte, donc des noms trop complexes peuvent perturber certains résolveurs.

## Noms d'instance échappés (`\032`)

Bonjour/DNS-SD échappe souvent les octets dans les noms d'instance de service en séquences décimales `\DDD` (par exemple, les espaces deviennent `\032`).

- C'est normal au niveau du protocole.
- Les interfaces doivent décoder pour l'affichage (iOS utilisé `BonjourEscapes.decode`).

## Désactivation / configuration

- `OPENCLAW_DISABLE_BONJOUR=1` désactive l'annonce (legacy : `OPENCLAW_DISABLE_BONJOUR`).
- `gateway.bind` dans `~/.openclaw/openclaw.json` contrôle le mode de liaison du Gateway.
- `OPENCLAW_SSH_PORT` remplace le port SSH annoncé dans les TXT (legacy : `OPENCLAW_SSH_PORT`).
- `OPENCLAW_TAILNET_DNS` publie un indice MagicDNS dans les TXT (legacy : `OPENCLAW_TAILNET_DNS`).
- `OPENCLAW_CLI_PATH` remplace le chemin CLI annoncé (legacy : `OPENCLAW_CLI_PATH`).

## Documentation connexe

- Politique de découverte et sélection de transport : [Découverte](/gateway/discovery)
- Appairage de nœuds + approbations : [Appairage Gateway](/gateway/pairing)
