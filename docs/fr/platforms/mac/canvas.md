---
summary: "Panneau Canvas contrôle par l'agent, intégré via WKWebView + schema d'URL personnalisé"
read_when:
  - Implementation du panneau Canvas macOS
  - Ajout de contrôles d'agent pour l'espace de travail visuel
  - Debogage du chargement du canvas WKWebView
title: "Canvas"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: platforms/mac/canvas.md
  workflow: manual
---

# Canvas (application macOS)

L'application macOS intègre un **panneau Canvas** contrôle par l'agent utilisant `WKWebView`. C'est
un espace de travail visuel leger pour HTML/CSS/JS, A2UI et de petites surfaces
interactives d'interface utilisateur.

## Ou le Canvas reside

L'état du Canvas est stocke sous Application Support :

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Le panneau Canvas sert ces fichiers via un **schema d'URL personnalisé** :

- `openclaw-canvas://<session>/<path>`

Exemples :

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

Si aucun `index.html` n'existe a la racine, l'application affiche une **page scaffold intégrée**.

## Comportement du panneau

- Panneau sans bordure, redimensionnable, ancre pres de la barre de menus (ou du curseur de la souris).
- Memorise la taille/position par session.
- Se recharge automatiquement lorsque les fichiers locaux du canvas changent.
- Un seul panneau Canvas est visible a la fois (la session est changee si nécessaire).

Le Canvas peut être désactivé depuis Paramètres → **Autoriser le Canvas**. Lorsqu'il est désactivé, les commandes
du nœud canvas retournent `CANVAS_DISABLED`.

## Surface API de l'agent

Le Canvas est expose via le **WebSocket du Gateway**, pour que l'agent puisse :

- afficher/masquer le panneau
- naviguer vers un chemin ou une URL
- exécuter du JavaScript
- capturer une image instantanee

Exemples CLI :

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notes :

- `canvas.navigate` accepte les **chemins canvas locaux**, les URLs `http(s)` et les URLs `file://`.
- Si vous passez `"/"`, le Canvas affiche le scaffold local ou `index.html`.

## A2UI dans le Canvas

A2UI est hébergé par l'hôte Canvas du Gateway et rendu dans le panneau Canvas.
Lorsque le Gateway annonce un hôte Canvas, l'application macOS navigue automatiquement vers la
page hôte A2UI a la première ouverture.

URL par défaut de l'hôte A2UI :

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### Commandes A2UI (v0.8)

Le Canvas accepte actuellement les messages serveur→client **A2UI v0.8** :

- `beginRendering`
- `surfaceUpdate`
- `dataModelUpdate`
- `deleteSurface`

`createSurface` (v0.9) n'est pas supporte.

Exemple CLI :

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Test rapide :

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

## Déclenchement d'executions d'agent depuis le Canvas

Le Canvas peut déclencher de nouvelles executions d'agent via des liens profonds :

- `openclaw://agent?...`

Exemple (en JS) :

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

L'application demande confirmation sauf si une clé valide est fournie.

## Notes de sécurité

- Le schema Canvas bloque la traversee de répertoires ; les fichiers doivent se trouver sous la racine de la session.
- Le contenu Canvas local utilisé un schema personnalisé (pas de serveur loopback nécessaire).
- Les URLs `http(s)` externes ne sont autorisées que lorsqu'elles sont explicitement naviguees.
