---
summary: "Utiliser un abonnement Claude Max/Pro comme endpoint API compatible OpenAI"
read_when:
  - Vous souhaitez utiliser un abonnement Claude Max avec des outils compatibles OpenAI
  - Vous voulez un serveur API local qui encapsule le CLI Claude Code
  - Vous souhaitez économiser en utilisant l'abonnement plutôt que des clés API
title: "Claude Max API Proxy"
x-i18n:
  generated_at: "2026-02-25T12:00:00Z"
  model: claude-opus-4-6
  provider: claude-code
  source_path: providers/claude-max-api-proxy.md
  workflow: manual
---

# Claude Max API Proxy

**claude-max-api-proxy** est un outil communautaire qui expose votre abonnement Claude Max/Pro comme un endpoint API compatible OpenAI. Cela vous permet d'utiliser votre abonnement avec n'importe quel outil prenant en charge le format API OpenAI.

## Pourquoi l'utiliser ?

| Approche                  | Coût                                                         | Idéal pour                                        |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| API Anthropic             | Paiement par token (~15 $/M entrée, 75 $/M sortie pour Opus) | Applications en production, haut volume           |
| Abonnement Claude Max     | 200 $/mois forfaitaire                                       | Usage personnel, développement, usage illimité    |

Si vous avez un abonnement Claude Max et souhaitez l'utiliser avec des outils compatibles OpenAI, ce proxy peut vous faire économiser considérablement.

## Fonctionnement

```
Votre App → claude-max-api-proxy → CLI Claude Code → Anthropic (via abonnement)
     (format OpenAI)              (convertit le format)      (utilise votre login)
```

Le proxy :

1. Accepte les requêtes au format OpenAI sur `http://localhost:3456/v1/chat/completions`
2. Les convertit en commandes CLI Claude Code
3. Retourne les réponses au format OpenAI (streaming pris en charge)

## Installation

```bash
# Nécessite Node.js 20+ et le CLI Claude Code
npm install -g claude-max-api-proxy

# Vérifiez que le CLI Claude est authentifié
claude --version
```

## Utilisation

### Démarrer le serveur

```bash
claude-max-api
# Le serveur tourne sur http://localhost:3456
```

### Tester

```bash
# Vérification de santé
curl http://localhost:3456/health

# Lister les modèles
curl http://localhost:3456/v1/models

# Complétion de chat
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Avec OpenClaw

Vous pouvez pointer OpenClaw vers le proxy comme endpoint personnalisé compatible OpenAI :

```json5
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```

## Modèles disponibles

| ID du modèle      | Correspond à     |
| ------------------ | ---------------- |
| `claude-opus-4`    | Claude Opus 4    |
| `claude-sonnet-4`  | Claude Sonnet 4  |
| `claude-haiku-4`   | Claude Haiku 4   |

## Démarrage automatique sur macOS

Créez un LaunchAgent pour exécuter le proxy automatiquement :

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## Liens

- **npm :** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
- **GitHub :** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
- **Issues :** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## Notes

- Il s'agit d'un **outil communautaire**, non officiellement supporté par Anthropic ou OpenClaw
- Nécessite un abonnement Claude Max/Pro actif avec le CLI Claude Code authentifié
- Le proxy s'exécute localement et n'envoie aucune donnée à des serveurs tiers
- Les réponses en streaming sont entièrement prises en charge

## Voir aussi

- [Fournisseur Anthropic](/providers/anthropic) - Intégration native OpenClaw avec setup-token Claude ou clés API
- [Fournisseur OpenAI](/providers/openai) - Pour les abonnements OpenAI/Codex
