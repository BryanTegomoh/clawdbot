---
summary: "Application nœud iOS : connexion au Gateway, appairage, canvas et dépannage"
read_when:
  - Appairage ou reconnexion du nœud iOS
  - Exécution de l'application iOS depuis les sources
  - Debogage de la decouverte du gateway ou des commandes canvas
title: "Application iOS"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/ios.md
  workflow: manual
---

# Application iOS (Nœud)

Disponibilité : aperçu interne. L'application iOS n'est pas encore distribuee publiquement.

## Ce qu'elle fait

- Se connecté a un Gateway via WebSocket (LAN ou tailnet).
- Expose les capacités du nœud : Canvas, Capture d'ecran, Capture camera, Localisation, Mode conversation, Voice wake.
- Reçoit les commandes `node.invoke` et rapporte les événements de statut du nœud.

## Prerequis

- Gateway en cours d'exécution sur un autre appareil (macOS, Linux ou Windows via WSL2).
- Chemin réseau :
  - Même LAN via Bonjour, **ou**
  - Tailnet via unicast DNS-SD (domaine exemple : `openclaw.internal.`), **ou**
  - Hôte/port manuel (solution de repli).

## Démarrage rapide (appairage + connexion)

1. Démarrez le Gateway :

```bash
openclaw gateway --port 18789
```

2. Dans l'application iOS, ouvrez Paramètres et choisissez un gateway decouvert (ou activez Hôte manuel et entrez l'hôte/port).

3. Approuvez la demande d'appairage sur l'hôte du gateway :

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

4. Verifiez la connexion :

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## Chemins de decouverte

### Bonjour (LAN)

Le Gateway annonce `_openclaw-gw._tcp` sur `local.`. L'application iOS les liste automatiquement.

### Tailnet (inter-réseaux)

Si mDNS est bloque, utilisez une zone unicast DNS-SD (choisissez un domaine ; exemple : `openclaw.internal.`) et le split DNS de Tailscale.
Voir [Bonjour](/gateway/bonjour) pour l'exemple CoreDNS.

### Hôte/port manuel

Dans Paramètres, activez **Hôte manuel** et entrez l'hôte du gateway + le port (par défaut `18789`).

## Canvas + A2UI

Le nœud iOS affiche un canvas WKWebView. Utilisez `node.invoke` pour le piloter :

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

Notes :

- L'hôte Canvas du Gateway sert `/__openclaw__/canvas/` et `/__openclaw__/a2ui/`.
- Il est servi depuis le serveur HTTP du Gateway (même port que `gateway.port`, par défaut `18789`).
- Le nœud iOS navigue automatiquement vers A2UI a la connexion lorsqu'une URL d'hôte Canvas est annoncee.
- Revenez au scaffold intégré avec `canvas.navigate` et `{"url":""}`.

### Canvas eval / snapshot

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## Voice wake + mode conversation

- Voice wake et le mode conversation sont disponibles dans Paramètres.
- iOS peut suspendre l'audio en arriere-plan ; considerez les fonctionnalités vocales comme fonctionnant au mieux lorsque l'application n'est pas activé.

## Erreurs courantes

- `NODE_BACKGROUND_UNAVAILABLE` : mettez l'application iOS au premier plan (les commandes canvas/camera/ecran le nécessitent).
- `A2UI_HOST_NOT_CONFIGURED` : le Gateway n'a pas annonce d'URL d'hôte Canvas ; verifiez `canvasHost` dans la [Configuration du Gateway](/gateway/configuration).
- La demande d'appairage n'apparaît jamais : exécutez `openclaw nodes pending` et approuvez manuellement.
- La reconnexion échoue après reinstallation : le jeton d'appairage du Keychain a ete efface ; reappairez le nœud.

## Documentation associée

- [Appairage](/gateway/pairing)
- [Decouverte](/gateway/discovery)
- [Bonjour](/gateway/bonjour)
